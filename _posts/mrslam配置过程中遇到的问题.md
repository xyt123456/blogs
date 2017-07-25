---
title: mrslam配置过程中遇到的问题
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---
1. g2o库编译错误，提示找不到g2o/config.h文件
  -解决办法：build/g2o下找到config.h文件，复制到src/g2o目录下

2. 

> -- ==> add_subdirectory(laser_scan_matcher)
-- Using these message generators: gencpp;genlisp;genpy
-- checking for module 'openni-dev'
--   package 'openni-dev' not found
-- checking for module 'openni-dev'
--   package 'openni-dev' not found
-- checking for module 'openni-dev'
--   package 'openni-dev' not found
-- checking for module 'csm'
--   package 'csm' not found

dpkg -l 显示软件包列表 状态标志： 
iU 表示软件包未安装成功
ii 表示安装成功
rc 表示软件包已经被卸载,但配置文件仍在

> xyt@ubuntu:~/mrslam_ws/ROSProjet$ dpkg -l | grep openni
ii  libopenni-dev                                         1.5.4.0-7                                           i386         headers for OpenNI 'Natural Interaction' frameworks
ii  libopenni-sensor-pointclouds0                         5.1.0.41.1-1                                        i386         Microsoft Kinect sensor modules for the OpenNI framework
ii  libopenni0                                            1.5.4.0-7                                           i386         framework for sensor-based 'Natural Interaction'
ii  openni-utils                                          1.5.4.0-7                                           i386         debug and test utilities OpenNI framework


![apt-get方式下载安装][1]
欢迎使用 **{小书匠}(xiaoshujiang)编辑器**，您可以通过==设置==里的修改模板来改变新建文章的内容。


  [1]: ./images/1500942982912.jpg "1500942982912.jpg"
