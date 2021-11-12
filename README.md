#ByteTrack训练自己数据集详细教程！！

## 一、配置环境
### 1. Installing on the host machine
Step1. Install ByteTrack.
```shell
git clone https://github.com/Double-zh/ByteTrack.git
cd ByteTrack
pip3 install -r requirements.txt
python3 setup.py develop
```

Step2. Install [pycocotools](https://github.com/cocodataset/cocoapi).

```shell
pip3 install cython; pip3 install 'git+https://github.com/cocodataset/cocoapi.git#subdirectory=PythonAPI'
```

Step3. Others
```shell
pip3 install cython_bbox
```
### 2. Docker build
```shell
docker build -t bytetrack:latest .

# Startup sample
mkdir -p pretrained && \
mkdir -p YOLOX_outputs && \
xhost +local: && \
docker run --gpus all -it --rm \
-v $PWD/pretrained:/workspace/ByteTrack/pretrained \
-v $PWD/datasets:/workspace/ByteTrack/datasets \
-v $PWD/YOLOX_outputs:/workspace/ByteTrack/YOLOX_outputs \
-v /tmp/.X11-unix/:/tmp/.X11-unix:rw \
--device /dev/video0:/dev/video0:mwr \
--net=host \
-e XDG_RUNTIME_DIR=$XDG_RUNTIME_DIR \
-e DISPLAY=$DISPLAY \
--privileged \
bytetrack:latest
```

## 二、准备VOC数据集和下载预训练模型

```
### 1. datasets
           └——————VOCdevkit
           |         └——————VOC2012
           |                   └——————Annotations
           |                   └——————ImageSets
           |                                 └——————Main
           |                   └——————JPEGImages
                               └—————— divide_dataset.py
```
### 2. Download pretrained model
```
The COCO pretrained YOLOX model can be downloaded from their [model zoo](https://github.com/Megvii-BaseDetection/YOLOX/tree/0.1.0). After downloading the pretrained models, you can put them under <ByteTrack_HOME>/pretrained.
```


## 三、准备模型配置文件{create a Exp file for your dataset && modify get_data_loader and get_eval_loader in your Exp file}

```
根据需求修改文件yolox_voc_s_ZZH.py的种类数，在路径"exps/example/custom/"文件夹下

class Exp(MyExp):
    def __init__(self):
        super(Exp, self).__init__()
        self.num_classes = 2 #在这进行修改
        self.depth = 0.33
        self.width = 0.50
        self.warmup_epochs = 1


## 四、Training

**Train with custom dataset**

```shell
cd <ByteTrack_HOME>
python3 train.py -f exps/example/custom/yolox_voc_s_ZZH.py -d 1 -b 1 --fp16 -o -c pretrained/yolox_s.pth
```

## 五、Demo

### 1. 对视频进行检测跟踪，并保存结果
```shell
cd <ByteTrack_HOME>
python3 ZZH_track.py video -f exps/example/custom/yolox_voc_s_ZZH.py -c YOLOX_outputs/yolox_voc_s_ZZH/latest_ckpt.pth.tar --fp16 --fuse --save_result

### 2. 调用摄像头进行实时检测跟踪，并保存结果
```
python3 ZZH_track.py webcam -f exps/example/custom/yolox_voc_s_ZZH.py -c YOLOX_outputs/yolox_voc_s_ZZH/latest_ckpt.pth.tar --fp16 --fuse --save_result


## 六、Deploy

1.  [ONNX export and ONNXRuntime](./deploy/ONNXRuntime)
2.  [TensorRT in Python](./deploy/TensorRT/python)
3.  [TensorRT in C++](./deploy/TensorRT/cpp)
4.  [ncnn in C++](./deploy/ncnn/cpp)


## 七、Citation

```
@article{zhang2021bytetrack,
  title={ByteTrack: Multi-Object Tracking by Associating Every Detection Box},
  author={Zhang, Yifu and Sun, Peize and Jiang, Yi and Yu, Dongdong and Yuan, Zehuan and Luo, Ping and Liu, Wenyu and Wang, Xinggang},
  journal={arXiv preprint arXiv:2110.06864},
  year={2021}
}
```

## 八、Acknowledgement

A large part of the code is borrowed from [YOLOX](https://github.com/Megvii-BaseDetection/YOLOX), [FairMOT](https://github.com/ifzhang/FairMOT), [TransTrack](https://github.com/PeizeSun/TransTrack) and [JDE-Cpp](https://github.com/samylee/Towards-Realtime-MOT-Cpp). Many thanks for their wonderful works.
