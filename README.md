# flexible-yolov5


*I update the code for  [ultralytics/yolov5](https://github.com/ultralytics/yolov5) version 6.1+.*
---

The original Yolo V5 was an amazing project. When I want to make some changes to the network, it's not so easy, such as adding branches and trying other backbones. Maybe there are people like me, so I split the yolov5 model to {backbone, neck, head} to facilitate the operation of various modules and support more backbones.Basically, I only changed the model, and I didn't change the architecture, training and testing of yolov5. Therefore, if the original code is updated, it is also very convenient to update this code. if this repo can help you, please give me a star.

## Table of contents
* [Features](#features)
* [Notices](#Notices)
* [Prerequisites](#prerequisites)
* [Getting Started](#getting-started)
    * [Dataset Preparation](#dataset-preparation)
    * [Training and Testing](#Training-and-Testing)
        *[Training](#training)
        *[Testing and Visualize](#testing-and-visualize)
    * [Model performance comparison](#model-performance-comparison)
    * [Detection](#Detection)
    * [Deploy](#Deploy)
* [Reference](#Reference)


## Features
- Reorganize model structure, such as backbone, neck, head, can modify the network flexibly and conveniently
- mobilenetV3-small, mobilenetV3-large 
- shufflenet_v2_x0_5, shufflenet_v2_x1_0, shufflenet_v2_x1_5, shufflenet_v2_x2_0
- yolov5s, yolov5m, yolov5l, yolov5x, yolov5transformer
- resnet18, resnet50, resnet34, resnet101, resnet152 
- efficientnet_b0 - efficientnet_b8, efficientnet_l2
- hrnet 18,32,48
- CBAM, SE
- swin transformer - base, tiny, small, large (please set half=False in scripts/eval.py and don't use model.half in train.py)
- DCN (mixed precision training not support, if you want use dcn, please close amp in line 292 of scripts/train.py)
- coord conv
- drop_block
- vgg, repvgg

## Notices

* The CBAM, SE, DCN, coord conv. At present, the above plug-ins are not added to all networks, so you may need to modify the code yourself.
* The default gw and gd for PAN and FPN of other backbone are same as yolov5_s, so if you want a strong model, please modify self.gw and self.gd in FPN and PAN.

## Prerequisites

please refer requirements.txt

## Getting Started

### Dataset Preparation

Make data for yolov5 format. you can use od/data/transform_voc.py convert VOC data to yolov5 data format.

### Training and Testing

For training and Testing, it's same like yolov5.

#### Training

1. check out configs/data.yaml, and replace with your data， and number of object nc
2. check out configs/model_*.yaml, choose backbone. and change nc to your dataset. please refer support_backbone in models.backbone.__init__.py
3. 
```shell script
$ python scripts/train.py  --batch 16 --epochs 5 --data configs/data.yaml --cfg configs/model_XXX.yaml
```

A google colab demo in train_demo.ipynb

#### Testing and Visualize

```shell script
$ python scripts/eval.py   --data configs/data.yaml  --weights runs/train/yolo/weights/best.py
```

### Model performance comparison 

compare results of COCO2017 on the way.


--------------------------
### Detection

```shell
python scripts/detector.py   --weights yolov5.pth --imgs_root  test_imgs   --save_dir  ./results --img_size  640  --conf_thresh 0.4  --iou_thresh 0.4
```

### Deploy

For tf_serving or triton_server, you can set model.detection.export = False in scripts/export.py in line 50 to export an onnx model, A new output node will be added to combine the three detection output nodes into one. 
For Official tensorrt converter, you should set model.detection.export = True, because  ScatterND op not support by trt. For this repo, best use official tensorrt converter, not [tensorrtx](https://github.com/wang-xinyu/tensorrtx)

#### Quantization

You can directly quantify the onnx model

```shell
python scripts/trt_quant/convert_trt_quant.py  --img_dir  /XXXX/train/  --img_size 640 --batch_size 6 --batch 200 --onnx_model runs/train/exp1/weights/bast.onnx  --mode int8
```
[See](scripts/trt_quant/README)

trt python infer demo scripts/trt_quant/trt_infer.py


## bugs fixed

- ~~resnet with dcn, training on gpu *RuntimeError: expected scalar type Half but found Float~~
- ~~swin-transformer, training is ok, but testing report *RuntimeError: expected object of scalar type Float but got scalar type Half for argument #2 'mat2' in call to_th_bmm_out in swin_trsansformer.py 143~~
- ~~mobilenet export onnx failed, please replace HardSigmoid() by others, because onnx don't support pytorch nn.threshold~~
## Reference

* [ultralytics/yolov5](https://github.com/ultralytics/yolov5)
* [EfficientNet-PyTorch](https://github.com/lukemelas/EfficientNet-PyTorch)
* [Mobilenet v3](https://arxiv.org/abs/1905.02244)
* [resnet](https://arxiv.org/abs/1512.03385)
* [hrnet](https://arxiv.org/abs/1908.07919)
* [shufflenet](https://arxiv.org/abs/1707.01083)
* [swin transformer](https://github.com/SwinTransformer/Swin-Transformer-Object-Detection)
* [dcn-v2](https://github.com/jinfagang/DCNv2_latest)
* [coord_conv](https://github.com/mkocabas/CoordConv-pytorch)
* [triton server](https://github.com/triton-inference-server/server)
* [drop_block](https://github.com/miguelvr/dropblock)
* [trt quan](https://github.com/Wulingtian/nanodet_tensorrt_int8_tools)
* [repvgg](https://github.com/DingXiaoH/RepVGG)
