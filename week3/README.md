# Week3
---

## 1. 量化ResNet模型

   参考文档：https://github.com/open-mmlab/mmdeploy/blob/master/docs/zh_cn/02-how-to-run/quantize_model.md
   
   ```
   cd /path/to/mmdeploy
   export MODEL_CONFIG=/path/to/mmclassification/configs/resnet/resnet18_8xb16_cifar10.py
   export MODEL_PATH=https://download.openmmlab.com/mmclassification/v0/resnet/resnet18_b16x8_cifar10_20210528-bd6371c8.pth

   python3 tools/deploy.py  configs/mmcls/classification_ncnn-int8_static.py  ${MODEL_CONFIG}  ${MODEL_PATH}   /path/to/self-test.png   --work-dir work_dir --device cpu --quant --quant-image-dir /path/to/images
   ...
   ```
   
   [`deploy.py`](https://github.com/open-mmlab/mmdeploy/blob/master/tools/deploy.py) 工作流程：
   1. load deploy_cfg和model_cfg (line 119)
   2. 把torch模型转化成onnx模型(line 130)
   3. 拆分模型（line 146）在模型过大的情况下需要拆分模型 这里量化ResNet无需考虑
   4. 获得校准数据（line 176）
   5. 通过PPQ使用校准数据生成校准表（line 254）（onnx2ncnn_quant_table line 10）
   6. 将onnx模型转化为ncnn_fp32模型（line 239）
   7. 使用校准表将ncnn_fp32模型量化为int8模型（line 263）
