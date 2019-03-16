# opencv-haar-classifier-training 
[本篇文章完全参考Thorsten Ball的博客](http://coding-robin.de/2013/07/22/train-your-own-opencv-haar-classifier.html)

[作者github链接](https://github.com/mrnugget/opencv-haar-classifier-training.git)
## 如何在ubuntu16.04下训练自己的级联分类器：

以红绿灯和停止标志的识别为例子

训练级联分类器需要：

大量的正面样本（你需要识别的物体图片，可以是任意图片但必须包含你所需要识别的物体）

大量的负面样本（一定不能包含你想要识别的物体）

收集好所需的图片后，你需要有一个.vec文件包含你图片的大小信息，被识别物体所在的位置。<br>
有两种方法可以创建文本信息：

		1：手动创建文本，人工标签识别物体的位置
		2：利用opencv的createsamples功能，把正面样本和负面样本混合，生成大量的正面样本同时文本信息也自动给你生成了 <br>
		(这里采用第二种方法，理论上第一种方法的准确率要高，但需要耗费大量的人工)

stop_sign_pictures.tar压缩包里包含了25张停止标志的图片 <br>
traffic_light_pictures包含了30张红绿灯的图片 <br>
trained_classifiers文件夹下放了一些训练好的xml文件，可以直接使用。<br>

完成以上工作完成后即可进行训练

--------------------------------------
1.打开终端输入下面指令下载文件到电脑上

安装git：
		sudo apt-get install git

clone文件：
		git clone https://github.com/dj140/opencv-haar-classifier-training.git



2.解压neg_pictures.tar文件，把里面的图片复制到negative_images文件夹下，负面图片的大小要比正面图像大，然后执行下列命令
 
 
		find ./negative_images -iname "*.jpg" > negatives.txt

3.正面样本需要自己用手机拍摄，剪裁，转化为灰度图，拷贝到positive_images文件夹下（已经拍好放在压缩包里了）

ubuntu下有一个指令可以批量修改图像的大小：

		sudo mogrify -resize 25x25 -format jpg *

4.将你想要检测的物体拍照，剪裁为特定大小（图像越大训练的时间会大大增长），转换为灰度图，放到positive_images文件夹下，执行下列命令。（已经拍好图片了）

		find ./positive_images -iname "*.jpg" > positives.txt


5.执行下列指令，将正面图片和负面图片混合以此生成一大堆带有负面图片背景的正面图片样本。生成的样本数量为1500，大小为25x25.

		perl bin/createsamples.pl positives.txt negatives.txt samples 1500\
   		  "opencv_createsamples -bgcolor 0 -bgthresh 0 -maxxangle 1.1\
   		  -maxyangle 1.1 maxzangle 0.5 -maxidev 40 -w 25 -h 25"




6.上一条指令执行结束后，会在samples文件加上生成一些带.vec后缀的文件，再执行下面的指令，用mergevec.py程序把文件夹下的所有.vec文件和为一体，输出samples.vec文件

		python ./tools/mergevec.py -v samples/ -o samples.vec


完成上面的步骤后，你的文件目录应该有如下文件：

![image](https://github.com/dj140/opencv-haar-classifier-training/raw/master/images/file.png)



7.利用opencv进行训练级联分类器，正面样本1000张，负面样本600张，（一般正面样本不要取全，负面样本取正面样本数量的一半）bufsize根据自己电脑的内存选择

		opencv_traincascade -data classifier -vec samples.vec -bg negatives.txt\
   		  -numStages 20 -minHitRate 0.999 -maxFalseAlarmRate 0.5 -numPos 1000\
   		  -numNeg 600 -w 25 -h 25 -mode ALL -precalcValBufSize 1024\
   		  -precalcIdxBufSize 1024
   
   




8.opencv的另一种训练模型LBP,速度要比前面一个快很多，准确度可能会差一点点，建议使用这个

		opencv_traincascade -data classifier -vec samples.vec -bg negatives.txt\
   		  -numStages 20 -minHitRate 0.999 -maxFalseAlarmRate 0.5 -numPos 1000\
   		  -numNeg 600 -w 25-h 25 -mode ALL -precalcValBufSize 1024\
   		  -precalcIdxBufSize 1024 -featureType LBP



	执行指令后输出如下：
	```
	===== TRAINING 0-stage =====
	<BEGIN
	POS count : consumed   1000 : 1000
	NEG count : acceptanceRatio    600 : 1
	Precalculation time: 11
	+----+---------+---------+
	|  N |    HR   |    FA   |
	+----+---------+---------+
	|   1|        1|        1|
	+----+---------+---------+
	|   2|        1|        1|
	+----+---------+---------+
	|   3|        1|        1|
	+----+---------+---------+
	|   4|        1|        1|
	+----+---------+---------+
	|   5|        1|        1|
	+----+---------+---------+
	|   6|        1|        1|
	+----+---------+---------+
	|   7|        1| 0.711667|
	+----+---------+---------+
	|   8|        1|     0.54|
	+----+---------+---------+
	|   9|        1|    0.305|
	+----+---------+---------+
	END>
	Training until now has taken 0 days 3 hours 19 minutes 16 seconds.
	```
--------------------------------
以上步骤都执行完成后，classifier文件夹下会有一些.xml文件，cascade.xml文件即可在opencv中使用<br>
如果stage没跑到20层，试着把positives样本数量降低。<br>
测试准确度如果很差，可以采用第一种方法重新训练一遍

参考文章链接：
https://pythonprogramming.net/haar-cascade-object-detection-python-opencv-tutorial/ <br>
https://coding-robin.de/2013/07/22/train-your-own-opencv-haar-classifier.html <br>
https://github.com/mrnugget/openchttps://coding-robin.de/2013/07/22/train-your-own-opencv-haar-classifier.htmlv-haar-classifier-training

