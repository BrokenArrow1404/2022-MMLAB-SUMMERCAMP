# Week1
---

## 准备

1. 编译安装NCNN
    
    运行环境：i7-10700 WSL2下的Ubuntu 20.04 
    
    由于WSL2在2022.7时对GPU支持不是很好因此禁用了Vulkan(GPU加速)
    
    编译命令：`cmake -DCMAKE_BUILD_TYPE=Release -DNCNN_VULKAN=OFF -DNCNN_BUILD_EXAMPLES=ON ..`
    
2. 用example中的squeezenet模型识别[猫猫图片](https://github.com/nihui/ncnn-android-squeezenet/blob/master/screenshot.png)
   
   `./squeezenet target1.png`
   
   得到结果： 
   ```
   281 = 0.605267
   285 = 0.107097
   282 = 0.040109
   ```
   
   在同目录下的[synset_words.txt](https://github.com/Tencent/ncnn/blob/master/examples/synset_words.txt)中查询可得281为 `tabby, tabby cat` 285为 `Egyptian cat` 282为 `tiger cat` 可知推理大致正确
   
## 开始量化

1. 用[校准集](https://github.com/nihui/imagenet-sample-images)的图片生成 calibration table

   ```
   ./ncnn2table squeezenet_v1.1.param squeezenet_v1.1.bin imagelist.txt x.table mean=[104,117,123] norm=[1,1,1] shape=[227,227,3] pixel=BGR thread=1 method=kl
   mean = [104.000000,117.000000,123.000000]
   norm = [1.000000,1.000000,1.000000]
   shape = [227,227,3]
   pixel = BGR
   thread = 1
   method = kl
   ```
   
2. 利用NCNN提供的ncnn2table工具量化squeezenet

   ```
   ./ncnn2int8 squeezenet_v1.1.param squeezenet_v1.1.bin int8.param int8.bin x.table
   ```
3. 修改[squeezenet.cpp](https://github.com/Tencent/ncnn/blob/master/examples/squeezenet.cpp)中的模型路径并重新make安装

4. 用量化后的模型推理

   得到结果：
   ```
   281 = 0.607189
   285 = 0.090009
   282 = 0.027600
   ```
   
## naive convolution的C++实现

由于不怎么会调库就直接写了一个最简单的

```
#include<cstdio>
#include<cstring>
#include<iostream>
using namespace std;

//no padding naive convolution

int graphX, graphY, kernelX, kernelY, resultX, resultY;
int graph[5][5] = {{1,1,1,0,0}, {0,1,1,1,0}, {0,0,1,1,1}, {0,0,1,1,0}, {0,1,1,0,0}}, kernel[3][3] ={{1,0,1}, {0,1,0}, {1,0,1}}, result[1000][1000];

void convolution(){
    resultX = graphX - kernelX + 1;
    resultY = graphY - kernelY + 1;

    for(int i = kernelX / 2; i < kernelX / 2 + resultX; i++){
        for(int j = kernelY / 2; j < kernelY / 2 + resultY; j++){
            int sum = 0;
 
            for(int k = 0; k < kernelX; k++){
                for(int l = 0; l < kernelY; l++){
                    sum += kernel[k][l] * graph[i - kernelX / 2 + k][j - kernelY / 2 + l];
                }
            }

            result[i][j] = sum;
        }
    }
}

int main(){

    memset(result, 0, sizeof(result));

    graphX = graphY = 5;
    kernelX = kernelY = 3;

    convolution();

    for(int i = kernelX / 2; i < kernelX / 2 + resultX; i++){
        for(int j = kernelY / 2; j < kernelY / 2 + resultY; j++){
            printf("%d ",result[i][j]);
        }
        printf("\n");
    }
}
```
