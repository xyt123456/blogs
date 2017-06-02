---
title: VS 头文件目录有关问题 
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---
找到*.vcxproj文件
![enter description here][1]
在</project>标签前添加
![enter description here][2]
$(IncludePath)为原头文件路径，一定要包含进去。
重启后，则项目默认包含指定头文件路径。

欢迎使用 **{小书匠}(xiaoshujiang)编辑器**，您可以通过==设置==里的修改模板来改变新建文章的内容。


  [1]: ./images/1496427867486.jpg "1496427867486.jpg"
  [2]: ./images/1496427911331.jpg "1496427911331.jpg"
