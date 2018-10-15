---
layout: post 
title: Unity 中lock使用不当导致Unity卡死
categories: [Unity, C#]
description: Unity 中lock使用不当导致Unity卡死
keywords: Unity, lock锁死
---

早上遇到个问题，多个文件同时下载失败时，导致 Unity 卡死。后面才发现是 线程锁使用不当的原因。

代码里下载的逻辑是 当前文件下载失败后（回调 error_callback_）（如下面的简要代码），在 error_callback_ 里 查找是否还有待下载的文件；如果有，查找是否有空闲的下载器；如果有，初始化下载，执行start()

问题就来了，error_callback 是在 OnFailed 方法的 lock (lock_obj_) 里，实际上，OnFailed 的lock 还未执行完，而此时这个下载器 已经 IsDone 了，所以我从下载器池里是可能取出当前下载器的。当我取出了当前下载器，执行Start( )  就会一直等待在 Start里的lock。因为该脚本还未从OnFailed 的 lock 里出来。

解决也很简单。把回调放到  lock (lock_obj_) 方面就可以了。

实际上 使用lock 也是为了避免同时访问一些东西，类似 error_callback 这种回调 已经和该类要避免同时访问的东西完全不搭边了 就不应该放里面的。

习惯不好，导致这个小问题上午排查了很久。

```
public class HttpAsyDownload
{
    /// <summary>
    ///     开始下载
    /// </summary>
    public void Start(string root, string local_file_name
        , Action<HttpAsyDownload, long> notify = null
        , Action<HttpAsyDownload> error_cb = null,Action<long> begin_cb=null)
    {
        lock (lock_obj_)
        {
            Abort();
            IsDone = false;
            ......
            _Download();
        }
    }

    /// <summary>
    ///     下载失败
    /// </summary>
    private void OnFailed(emErrorCode code)
    {
        lock (lock_obj_)
        {
            ......
            IsDone = true;
            ErrorCode = code;
            if (error_callback_ != null)
            	error_callback_(this);
        }
    }
}
```

