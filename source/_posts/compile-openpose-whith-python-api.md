---
title: 编译openpose，支持python api
date: 2019-11-29 09:50:14
categories: openpose
tags:
    - openpose
    - python
---
本篇主要讲述，如何在windows 10 x64上，从源代码编译openpose，并支持python api。
<!-- more -->
# 安装必要的软件
## windows 10
## git
## 安装python，并把python放到系统的path里。  
现阶段（openpose 1.5）还未支持python3.8，所以下载3.7的最新版本  
https://www.python.org/ftp/python/3.7.5/python-3.7.5-amd64.exe
- 安装所需的模块
```
pip install opencv-python
```
## 安装cuda
### 确认本机显卡驱动版本
- 从控制面板打开NVIDIA面板
- 点击【帮助】菜单的【系统信息】便可以查看显卡驱动版本，我的版本是`422.02`

### 打开以下网站，确认适合的CUDA版本
https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html

### 本次以9.2版本为例安装
https://developer.nvidia.com/cuda-92-download-archive?target_os=Windows&target_arch=x86_64&target_version=10&target_type=exelocal

## 安装cudnn
- 下载cuda所对应版本的cudnn，这个需要注册才能下载
- 安装非常简单，只要把文件放置到cuda的相应目下即可
- 为了让cudann支持最新的visual studio 2017版本，修改以下cudann的头文件
```
C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v9.2\include\crt\host_config.h
```
131行把最后一个数字增大，1920是visual studio 2019版的初始版本，改为1920应该没问题
```c
#if _MSC_VER < 1600 || _MSC_VER > 1920
```
## 安装visual studio community 2017
- 安装c++编译器即可
## 安装cmake,这里以版本3.6.0为例。
https://github.com/Kitware/CMake/releases/download/v3.16.0/cmake-3.16.0-win64-x64.msi
# 编译openpose
## 克隆最新的openpose源代码，由于openpose内部参照了其他的仓库资源，所以这边用--recursive来克隆
```bat
git clone --recursive https://github.com/CMU-Perceptual-Computing-Lab/openpose.git
```
## 打开cmake配置编译项
- `[where is the source code]`指定openpose的目录
- `[where to build the binaries]`指定编译目录，一般指定为openpose目录下的`build`目录
- 点击`Configure`按钮，便会下载所需文件，以及配置编译参数
- 选项中选择`BUILD_PYTHON`，然后点击`Generate`按钮，生成visual studio 2017的工程
- 点击`Open Project`按钮，打开visual studio 2017，并打开openpose工程
## build
- 在visual studio 2017中，选择编译为release，右键点击solution OpenPose，选择build solution。

# 用python开发程序时，用到的文件
## openpose所需dll文件
- openpose\build\bin下的所有文件
- openpose\build\x64\Release下的openpose.dll文件
## python的包文件
- openpose\build\python\openpose\Release下的pyopenpose.cp37-win_amd64.pyd文件
## openpose的模块文件
- openpose\models

# python示例
```python
# From Python
# It requires OpenCV installed for Python
import sys
from cv2 import cv2
import os
import argparse

try:
    dir_path = os.path.dirname(os.path.realpath(__file__))
    try:
        sys.path.append(dir_path + '/pyd') # 这个地方放的是openpose的模块文件
        os.environ['PATH'] = os.environ['PATH'] + ';' + dir_path + '/lib;' #这个地方放的是openpose所需dll文件
        import pyopenpose as op
    except ImportError as e:
        print('Error: OpenPose library could not be found. Did you enable `BUILD_PYTHON` in CMake and have this Python script in the right folder?')
        raise e

    # Flags
    parser = argparse.ArgumentParser()
    parser.add_argument("--image_path", default="./COCO_val2014_000000000241.jpg",
                        help="Process an image. Read all standard formats (jpg, png, bmp, etc.).")
    args = parser.parse_known_args()

    # Custom Params (refer to include/openpose/flags.hpp for more parameters)
    params = dict()
    params["model_folder"] = dir_path + "/models" # 这个地方放的是openpose的模块文件
    #params["face"] = True
    #params["hand"] = True

    # Add others in path?
    for i in range(0, len(args[1])):
        curr_item = args[1][i]
        if i != len(args[1])-1:
            next_item = args[1][i+1]
        else:
            next_item = "1"
        if "--" in curr_item and "--" in next_item:
            key = curr_item.replace('-', '')
            if key not in params:
                params[key] = "1"
        elif "--" in curr_item and "--" not in next_item:
            key = curr_item.replace('-', '')
            if key not in params:
                params[key] = next_item

    # Construct it from system arguments
    # op.init_argv(args[1])
    # oppython = op.OpenposePython()

    # Starting OpenPose
    opWrapper = op.WrapperPython()
    opWrapper.configure(params)
    opWrapper.start()

    # Process Image
    datum = op.Datum()
    imageToProcess = cv2.imread(args[0].image_path)
    datum.cvInputData = imageToProcess
    opWrapper.emplaceAndPop([datum])

    # Display Image
    print("Body keypoints: \n" + str(datum.poseKeypoints))
    #print("Face keypoints: \n" + str(datum.faceKeypoints))
    #print("Left hand keypoints: \n" + str(datum.handKeypoints[0]))
    #print("Right hand keypoints: \n" + str(datum.handKeypoints[1]))
    cv2.imshow("OpenPose 1.5.1 - Tutorial Python API", datum.cvOutputData)
    cv2.waitKey(0)
except Exception as e:
    print(e)
    sys.exit(-1)

```