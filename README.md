# opencv-haar-classifier-training

##如何在ubuntu16.04下训练自己的级联分类器：

###1.打开终端输入下面指令下载文件到电脑上
git clone https://github.com/dj140/opencv-haar-classifier-training.git

压缩图片大小

sudo mogrify -resize 800x600 -format jpg *

2.解压picture.tar文件，把里面的600张负面图片复制到negative_images文件夹下，负面图片的大小要比正面图像大，然后执行下列命令
 
 
		find ./negative_images -iname "*.jpg" > negatives.txt



3.将你想要检测的物体拍照，剪裁为特定大小（图像越大训练的时间会大大增长），转换为灰度图，放到positive_images文件夹下，执行下列命令。（已经拍好图片了）

 find ./positive_images -iname "*.jpg" > positives.txt


4.执行下列指令，将正面图片和负面图片混合以此生成一大堆带有负面图片背景的正面图片样本。生成的样本数量为1500，大小为25x25.
perl bin/createsamples.pl positives.txt negatives.txt samples 1500\
   "opencv_createsamples -bgcolor 0 -bgthresh 0 -maxxangle 1.1\
   -maxyangle 1.1 maxzangle 0.5 -maxidev 40 -w 25 -h 25"




5.上一条指令执行结束后，会在samples文件加上生成一些带.vec后缀的文件，再执行下面的指令，用mergevec.py程序把文件夹下的所有.vec文件和为一体，输出samples.vec文件

python ./tools/mergevec.py -v samples/ -o samples.vec


完成上面的步骤后，你的文件目录应该有如下文件：

![image](https://github.com/dj140/opencv-haar-classifier-training/raw/master/images/file.png)



6.利用opencv进行训练级联分类器，正面样本1000张，负面样本600张，（一般正面样本不要取全，负面样本取正面样本数量的一半）bufsize根据自己电脑的内存选择

opencv_traincascade -data classifier -vec samples.vec -bg negatives.txt\
   -numStages 20 -minHitRate 0.999 -maxFalseAlarmRate 0.5 -numPos 1000\
   -numNeg 600 -w 25 -h 25 -mode ALL -precalcValBufSize 1024\
   -precalcIdxBufSize 1024
   
   




7.opencv自带的另一种训练模型,速度要比前面一个快很多，准确度可能会差一点点，建议使用这个
opencv_traincascade -data classifier -vec samples.vec -bg negatives.txt\
   -numStages 20 -minHitRate 0.999 -maxFalseAlarmRate 0.5 -numPos 1000\
   -numNeg 600 -w 25-h 25 -mode ALL -precalcValBufSize 1024\
   -precalcIdxBufSize 1024 -featureType LBP

