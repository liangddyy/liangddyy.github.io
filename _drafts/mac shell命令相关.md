OS X使用fswatch+rsync

```
# 安装
# MacPorts
$ port install fswatch

# Homebrew
$ brew install fswatch
```

自动输入密码相关！！！

```
echo 123 | sudo -S apt-get update
# 使用管道（上一个命令的 stdout 接到下一个命令的 stdin）
# 亲测
```

```
#!/bin/bash
sudo -S apt-get update << EOF 
你的密码
EOF
```

> 说明：在shell脚本中，通常将EOF与 << 结合使用，表示后续的输入作为子命令或子Shell的输入，直到遇到EOF为止，再返回到主Shell,即将‘你的密码’当做命令的输入

Mac shell脚本退出时关闭窗口设置

​	终端 - 偏好设置  - 设置 - shell 。。。。

Shell脚本获取进程pid

```
$$  获取当前shell的进程号（PID）
$!  执行上一个指令的PID
```

关闭窗口相关

```
osascript -e 'tell application "Terminal" to quit'
# 上面这个脚本将弹出确认关闭应用程序的选项。您可以在终端首选项中禁用此选项。
```

```
killall Terminal
# 关闭所有打开的终端窗口
```



判断一串字符是否为数字：

```
TEST="123"
if echo "$TEST" | grep -q '^[a-zA-Z0-9]\+$'; then  
	echo "$TEST is number."  
else  
	echo "$TEST is not number."  
fi
```



查看端口被哪个程序占用

```\
sudo lsof -i tcp:port
```

如：`sudo lsof -i tcp:8080`

显示所有进程

```
top
```

结束进程：

```
sudo killall [进程名]
sudo kill [pid]
sudo kill -9 PID
```

打开程序：

```
open /path/to/some.app
open "/Volumes/Macintosh HD/foo.txt"
open /applications/Unity5.4.3f1/Unity5.4.3f1.app
```
打开程序（补充）

`open /path/to/some.app` 打开目录中的程序

`open "/Volumes/Macintosh HD/foo.txt"` 

　　opens the document in the default application for its type (as determined by LaunchServices).

`open /Applications/` 在默认应用程序中为其类型打开文档（由LaunchServices确定）

`open -a /Applications/TextEdit.app "/Volumes/Macintosh HD/foo.txt"` 在指定的应用程序（在这种情况下为TextEdit）中打开文档。

`open -e "/Volumes/Macintosh HD/foo.txt"` 在TextEdit中打开文档（-e选项指定TextEdit）。

`open http://www.apple.com/` opens the URL in the default browser (lynx, naturally *wink*)

`open "file://localhost/Volumes/Macintosh HD/foo.txt"` 默认程序打开文件

`open "file://localhost/Volumes/Macintosh HD/Applications/"`