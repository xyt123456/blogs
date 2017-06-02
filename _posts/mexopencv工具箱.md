---
title: mexopencv工具箱 
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---
1. for opencv2.4：https://github.com/kyamagu/mexopencv/tree/v2.4
for opencv3.0：https://github.com/kyamagu/mexopencv/tree/v3.0
for opencv3.1：https://github.com/kyamagu/mexopencv （master的是最新版本，可能会持续更新）
**注意下载对应版本的mexopencv。**
2. 3.0对应指令：
![enter description here][1]
注意，第一个参数为字符串‘opencv_path’,第二个参数为include文件所在路径。

3. 编译好的.M文件在‘+cv’文件夹，，32位matlab对应生成*.mexw32文件

4. "不是有效的*.mexw32文件"：
   解决办法：https://cn.mathworks.com/matlabcentral/answers/95097-why-am-i-unable-to-execute-a-mex-file-created-in-matlab-7-2-r2006a-with-visual-studio-2005-8-0
   可能是依赖项缺失，因cmake先编译后link，缺失依赖项仍然能编译成功。找到$MATLAB\bin\win32下的 vcredist_x86.exe，运行安装必要依赖项。
   “Microsoft supplies a self-extracting executable that installs the required libraries. However, note that the version of the executable that you need to run is dependent on the compiler that was used to build the MEX file.”也就是说，在64位操作系统上，链接到的是64位库，故而要运行_x86版本的依赖项安装程序。

欢迎使用 **{小书匠}(xiaoshujiang)编辑器**，您可以通过==设置==里的修改模板来改变新建文章的内容。


  [1]: ./images/1496437752564.jpg "1496437752564.jpg"
