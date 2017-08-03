---
layout: post
title: "Android 解析RSS中所遇到的错误 NetworkOnMainThreadException"
keywords: 
description: ""
category: Android
tags: 
---

<!--markdown-->在做个RSS解析模块时，遇上了很奇葩的问题。目标 SDK 版本改为低版本时不会报错，但是改为高版本时就会报错。  
  
错误的代码：

```  
public static RSSFeed getFeed(String urlString) {  
    try {  
        URL url = new URL(urlString);  
        SAXParserFactory factory = SAXParserFactory.newInstance();  
        SAXParser parser = factory.newSAXParser();  
        XMLReader xmlreader = parser.getXMLReader();  
        RSSHandler rssHandler = new RSSHandler();  
        xmlreader.setContentHandler(rssHandler);  
        InputSource is = new InputSource(url.openStream());  
        xmlreader.parse(is);  
        return rssHandler.getFeed();  
    } catch (Exception ex) {  
        return null;  
    }  
}  
```  
  
错误行：  
  
```  
InputSource is = new InputSource(url.openStream());  
```  
  
查了很多资料后，终于找到了解决办法。  
  
在该方法中加入下面两行代码：  
  
```  
StrictMode.ThreadPolicy policy = new StrictMode.ThreadPolicy.Builder().permitAll().build();  
StrictMode.setThreadPolicy(policy);  
```  
  
> 错误原因  
  
因为我们在 UI线程 中使用了网络。阻塞了 UI线程。  
  
> 为什么加入这两行代码可以解决问题？  
  
StrictMode 是在Android2.3 版本后加入的，又称为严苛模式。严苛模式是一个开发工具，能够检测程序中的违例，从而修复。最常用的地方就是主线程中 disk的读写和 network。而恰好也能解决上面的问题，在 UI线程中访问网络。  
  
  
  
其实明白了错误原因，就知道怎么处理了，单独开个线程也可以解决上面的问题，只是用 StrictMode 似乎简单了好多。  
