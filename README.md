# 2022-MMLAB-SUMMERCAMP
A project about quantization of neural networks

记录一下使用MMDeploy部署及量化NCNN后端的OCR算法TextSnake

[MMDeploy](https://github.com/open-mmlab/mmdeploy)

[NCNN](https://github.com/Tencent/ncnn)

---

- week1
   - 使用NCNN的quantize工具量化sqznet并测试精度
   - 实现naive image conv

- week2
   - 阅读量化相关论文
   - 对比理解对称和非对称量化的区别
   - 理解校准集(calibration dataset)相关知识

- week3
   - 使用MMDeploy部署并量化MMCls中的ResNet模型
   - 理解deploy.py的工作流程
   - 简单了解ppq的工作原理
   
- week4
   - 阅读TextSnake相关论文
   - 实现MMOCR的TextSnake模型量化
   - 评估量化后的模型
