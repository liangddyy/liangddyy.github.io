---
layout: post
title: "Typecho迁移至Jekyll"
keywords: 
description: ""
category: 
tags: Typecho Jekyll
---

## 数据导出

typecho中导出markdown的工具[typecho-to-md](https://www.npmjs.com/package/typecho-to-md) 

导出模板配置 tempt.md

```
---
layout: <%- type %>
title: "<%- title %>"
keywords: 
description: ""
category: 
tags: 
---

<%- text %>
```

命令：

```
typecho2md  --host localhost -u root -k root -d typecho -p typecho_ -t tempt.md output
```

导出的文件名字Slug命名，但是文件修改日期就是写文章的日期。所有用 [Total Commander](http://down.tech.sina.com.cn/page/8135.html)将其根据文件修改日期批量命名。

## Github部分

新建仓库 `用户名.github.io` 这样Github默认使用jekyll构建，只需要下载一个模板，将导出的文章copy至post。