---
title: Pitor Toolbox使用学习（未完成）
tags: matlab工具包，soft-cascade，adaboost，acf特征
grammar_cjkRuby: true
---

工具包的组成结构：
![enter description here][1]
项目中主要直接调用detector中的函数：acfTrain训练模型->acfDetect使用训练出的detector检测—>acfModify可修改detector参数->acfTest测试分类效果

### 1. 安装、跑示例
这部分参考博客http://blog.csdn.net/ddreaming/article/details/52334913
工具包下载地址：https://pdollar.github.io/toolbox/
示例所用INRIA数据下载地址：http://www.vision.caltech.edu/Image_Datasets/CaltechPedestrians/datasets/INRIA/
.seq格式文件转为.png文件函数库下载地址：http://www.vision.caltech.edu/Image_Datasets/CaltechPedestrians/下载页面中的Matlab evaluation/labeling code (3.2.1)

首先将文件放到matlab可查询文件路径下：

``` stylus
 addpath(genpath('path/to/toolbox/')); savepath;
```
由于用到部分c++程序，需要编译(toolboxCompile为工具包自带编译程序)

``` stylus
toolboxCompile
```

打开acfDemoInria.m文件，修改原代码中的路径：

``` stylus
dataDir='E:\无人驾驶小车\code3.2.1\code3.2.1';
for s=1:2, pth='E:\无人驾驶小车\code3.2.1\code3.2.1\data\INRIA';
if(s==1), set='00'; type='train'; else set='01'; type='test'; end
if(exist([dataDir type '/posGt'],'dir')), continue; end
seqIo([pth '/set' set '/V000'],'toImgs',[dataDir type '/pos']);%%“seqIo”函数用于格式转换（第二个参数设为'toImgs'）：.seq转为.png
seqIo([pth '/set' set '/V001'],'toImgs',[dataDir type '/neg']);%%第一个参数为原.seq文件路径，第三个参数为生成文件路径
V=vbb('vbbLoad',[pth '/annotations/set' set '/V000']);
vbb('vbbToFiles',V,[dataDir type '/posGt']);
end
```
以上代码用于将.seq文件转为.png格式文件，存放在指定文件夹下。对应文件夹路径作为训练器的输入数据。
![enter description here][2]

运行结果：
![enter description here][3]

### 2. 输入格式
- posGt:正样本图片内的ground truth
  参考bbGet.m里的bbLoad函数：
  ![enter description here][4]  

``` stylus
if( format==0 )
  % load objs stored in default format
  fId=fopen(fName);
  if(fId==-1), error(['unable to open file: ' fName]); end; v=0;
  try v=textscan(fId,'%% bbGt version=%d'); v=v{1}; catch, end %#ok<CTCH>
  if(isempty(v)), v=0; end
  % read in annotation (m is number of fields for given version v)
  if(all(v~=[0 1 2 3])), error('Unknown version %i.',v); end
  frmt='%s %d %d %d %d %d %d %d %d %d %d %d';
  ms=[10 10 11 12]; m=ms(v+1); frmt=frmt(1:2+(m-1)*3);
  in=textscan(fId,frmt); for i=2:m, in{i}=double(in{i}); end; fclose(fId);
  % create objs struct from read in fields
  n=length(in{1}); objs=create(n);
  for i=1:n, objs(i).lbl=in{1}{i}; objs(i).occ=in{6}(i); end
  bb=[in{2} in{3} in{4} in{5}]; bbv=[in{7} in{8} in{9} in{10}];
  for i=1:n, objs(i).bb=bb(i,:); objs(i).bbv=bbv(i,:); end
  if(m>=11), for i=1:n, objs(i).ign=in{11}(i); end; end
  if(m>=12), for i=1:n, objs(i).ang=in{12}(i); end; end
```

 format=0即默认格式下，posGt文件第一行为“% bbGt version=版本号”，版本号0~3分别对应每行参数个数为[10 10 11 12]。接下来每行对应一个物体（obj），参数分别对应以下内容：
 ![enter description here][5]

- pos：含正样本图片
  注意要与posGt一一对应（命名）。

- neg：不含正样本的图片（也可以没有，程序会从pos中非Gt位置取负样本）

- posWin：正样本窗口

- negWin：负样本窗口

``` stylus
function [Is,IsOrig] = sampleWins( detector, stage, positive )
% Load or sample windows for training detector.
opts=detector.opts; start=clock;
if( positive ), n=opts.nPos; else n=opts.nNeg; end
if( positive ), crDir=opts.posWinDir; else crDir=opts.negWinDir; end
if( exist(crDir,'dir') && stage==0 )
  %若存在存放窗口的目录直接载入窗口图片
  fs=bbGt('getFiles',{crDir}); nImg=length(fs); assert(nImg>0);
  if(nImg>n), fs=fs(:,randSample(nImg,n)); else n=nImg; end
  for i=1:n, fs{i}=[{opts.imreadf},fs(i),opts.imreadp]; end
  Is=cell(1,n); parfor i=1:n, Is{i}=feval(fs{i}{:}); end
else
  % 用sampleWins1()从全图采样窗口
  hasGt=positive||isempty(opts.negImgDir); 
  fs={opts.negImgDir};%用不含正样本的图片生成
  if(hasGt), fs={opts.posImgDir,opts.posGtDir}; end%若无负样本图片，从正样本图片采样
  fs=bbGt('getFiles',fs); nImg=size(fs,2); assert(nImg>0);
  if(~isinf(n)), fs=fs(:,randperm(nImg)); end; Is=cell(nImg*1000,1);
  diary('off'); tid=ticStatus('Sampling windows',1,30); k=0; i=0; batch=64;
  while( i<nImg && k<n )
    batch=min(batch,nImg-i); Is1=cell(1,batch);
    parfor j=1:batch, ij=i+j;
      I = feval(opts.imreadf,fs{1,ij},opts.imreadp{:}); %#ok<PFBNS>
      gt=[]; if(hasGt), [~,gt]=bbGt('bbLoad',fs{2,ij},opts.pLoad); end
      Is1{j} = sampleWins1( I, gt, detector, stage, positive );
    end
    Is1=[Is1{:}]; k1=length(Is1); Is(k+1:k+k1)=Is1; k=k+k1;
    if(k>n), Is=Is(randSample(k,n)); k=n; end
    i=i+batch; tocStatus(tid,max(i/nImg,k/n));
  end
  Is=Is(1:k); diary('on');
  fprintf('Sampled %i windows from %i images.\n',k,i);
end
```
  

### 3. 参数设置

``` stylus
opts=acfTrain(); 
opts.modelDs=[20 20]; opts.modelDsPad=[30 30];
opts.posGtDir=[dataDir 'train/posGt']; opts.posImgDir=[dataDir 'train/pos']; 
opts.nNeg=1000; opts.nAccNeg=1500;opts.nPerNeg=8;
opts.pBoost.pTree.fracFtrs=1/16;opts.nWeak=[64 128];
opts.pLoad={'squarify',{1,1}};
opts.pPyramid.pChns.pColor.colorSpace='hsv';
opts.name='E:\无人驾驶小车\models\parking_slot_detection4';
```
- opts.modelDs：金字塔最小尺度，也即能检测到最小的包含物体区域
- opts.modelDsPad：用于多尺度
- opts.posGtDir、opts.posImgDir：训练数据目录
- opts.nNeg：负样本数量（默认5000）
- opts.nAccNeg：累计负样本数量（从之前的stage中抽取一定数量k负样本，k+nNeg《nAccNeg)
- opts.nPerNeg: 每张图片最多产生负样本数
- opts.pBoost.pTree.fracFtrs：adaboost过程中每个特征选取的特征池抽样比例。
- opts.pLoad：squarify，限定输出的矩形框长宽比？
- opts.name：日志文件和detector.mat存放位置。若该检测器以存在（同名），acfTrain函数会直接load detector
![enter description here][6]
- opts.pPyramid.pChns.pColor.colorSpace：设置3个颜色通道
- opts.pJitter:旋转正样本，产生更多正例。
  ![enter description here][7]



  [1]: ./images/1494344937183.jpg "1494344937183.jpg"
  [2]: ./images/1494347257194.jpg "1494347257194.jpg"
  [3]: ./images/1494348714718.jpg "1494348714718.jpg"
  [4]: ./images/1494348060189.jpg "1494348060189.jpg"
  [5]: ./images/1494348440541.jpg "1494348440541.jpg"
  [6]: ./images/1494349900629.jpg "1494349900629.jpg"
  [7]: ./images/1494350089112.jpg "1494350089112.jpg"
