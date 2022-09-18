# Week4
---

## 论文相关
(待补充)

## 模型量化流程

1. 准备模型和数据集

   在MMOCR的Model Zoo里下载已经训练好的[模型](https://github.com/open-mmlab/mmocr/blob/main/configs/textdet/textsnake/README.md)
   
   下载ctw1500数据集并用tools目录下的ctw1500_converter工具生成json目录 （[文档](https://mmocr.readthedocs.io/zh_CN/latest/datasets/det.html)内有详细说明）
   
   调用test.py工具在数据集上测试模型
   ```
   CUDA_VISIBLE_DEVICES= python tools/test.py configs/textdet/textsnake/textsnake_r50_fpn_unet_1200e_ctw1500.py ../TEXTSNAKE_model/textsnake_r50_fpn_unet_1200e_ctw1500-27f65b64.pth --show
   ```
   
   需要注意没有GUI的docker使用`-show`参数会报错 应使用`-eval`评估模型
   
   运行结果
   ![result1](https://github.com/BrokenArrow1404/2022-MMLAB-SUMMERCAMP/blob/main/week4/images/1.png)
   
   可知hmean约为0.817
   
2. 开始量化

   在config下添加[text-detection_ncnn-int8_static.py](https://github.com/BrokenArrow1404/2022-MMLAB-SUMMERCAMP/blob/main/week4/text-detection_ncnn-int8_static.py)
   
   这里根据[模型文件](https://github.com/open-mmlab/mmocr/blob/main/configs/_base_/det_models/textsnake_r50_fpn_unet.py)可以看出该模型的backbone为ResNet head为TextSnakeHead
   
   [ResNet](https://github.com/open-mmlab/mmocr/blob/91046671125a21471787afc5e20fe064105afec2/mmocr/models/textrecog/backbones/resnet_abi.py)和[TextSnakeHead](https://github.com/open-mmlab/mmocr/blob/main/mmocr/models/textdet/dense_heads/textsnake_head.py)中涉及到的算子均在ONNX中有对应算子 因此无需rewrite
   
   使用deploy.py进行量化
   ```
   cd mmdeploy
   MODEL_CONFIG=~/workspace/mmocr/configs/textdet/textsnake/textsnake_r50_fpn_unet_1200e_ctw1500.py
   MODEL_PATH=~/workspace/TEXTSNAKE_model/textsnake_r50_fpn_unet_1200e_ctw1500-27f65b64.pth
   WORK_DIR=~/workspace/int8_model
   MMDEPLOY_DIR=~/workspace/mmdeploy
   MMOCR_DIR=~/workspace/mmocr

   python ${MMDEPLOY_DIR}/tools/deploy.py \
   ${MMDEPLOY_DIR}/configs/mmocr/text-detection/text-detection_ncnn-int8_static.py \
   ${MMOCR_DIR}/configs/textdet/textsnake/textsnake_r50_fpn_unet_1200e_ctw1500.py \
   ${MODEL_PATH} \
   ${MMOCR_DIR}/demo/demo_text_det.jpg \
   --work-dir ${WORK_DIR} \
   --device cpu \
   --quant \
   --show
   ```
   
3. 测试量化后的模型
   
   可以很明显的看出量化后的模型缩小了很多
   ![result2](https://github.com/BrokenArrow1404/2022-MMLAB-SUMMERCAMP/blob/main/week4/images/4.png)
   
   ```
   python tools/test.py configs/mmocr/text-detection/text-detection_ncnn-int8_static.py ../mmocr/configs/textdet/textsnake/textsnake_r50_fpn_unet_1200e_ctw1500.py --model ../int8_model/end2end_int8.param ../int8_model/end2end_int8.bin --log2file log.txt --device cpu --metrics hmean-iou
   ```
   测试结果：
   ![result3](https://github.com/BrokenArrow1404/2022-MMLAB-SUMMERCAMP/blob/main/week4/images/5.png)
   
   hmean为0.818 量化基本上没损失精度
