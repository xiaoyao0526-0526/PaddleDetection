# 1.训练模型

## 准备阶段
- 我们选择用AI Studio，PaddlePaddle2.1.2

```shell
git clone https://github.com/xiaoyao0526-0526/PaddleDetection.git
```
- 切换路径
```shell
cd PaddleDetection
```
- 以下3步每次启动环境都需要做一遍
```shell
pip install paddleslim==2.1.1
```
```shell
pip install -r requirements.txt
```
```shell
python setup.py install
```
- 拷贝myconfig文件夹到PaddleDetection目录

---
## 训练全精度模型
- 训练全精度模型需要用到3个原版py脚本
<ol>
<li>PaddleDetection/ppdet/modeling/layers.py</li> 
<li>PaddleDetection/ppdet/modeling/heads/ssd_head.py</li>
<li>PaddleDetection/ppdet/slim/quant.py</li>
</ol>

```shell
python tools/train.py -c myconfig/ssd_mobilenet_v1.yml --eval
```
## 全精度模型推理

```shell
python tools/infer.py -c myconfig/ssd_mobilenet_v1.yml -o weights=output/ssd_mobilenet_v1/best_model --infer_img=dataset/aluminum_ssd_voc/images/1003.jpg --draw_threshold=0.5
```
## 在线量化训练
- 训练量化模型需要替换3个py脚本（还是自己上传吧，本来想写个脚本替换的，没整成）
```shell
python tools/train.py -c myconfig/ssd_mobilenet_v1.yml --slim_config myconfig/ssd_mobilenet_v1_qat.yml --eval
```
## 量化模型推理
```shell
python tools/infer.py -c myconfig/ssd_mobilenet_v1.yml --slim_config myconfig/ssd_mobilenet_v1_qat.yml -o weights=output/ssd_mobilenet_v1_qat/best_model --infer_img=dataset/aluminum_ssd_voc/images/1003.jpg --draw_threshold=0.5
```
- 推理整个文件夹
```shell
python -u tools/infer.py -c myconfig/ssd_mobilenet_v1.yml --slim_config myconfig/ssd_mobilenet_v1_qat.yml -o weights=output/ssd_mobilenet_v1_qat/best_model --infer_dir=dataset/aluminum_ssd_voc/images --output_dir=infer_output/testout
```

---
## 查看模型精度
- 全精度模型
```shell
python tools/eval.py -c myconfig/ssd_mobilenet_v1.yml -o weights=output/ssd_mobilenet_v1/best_model
```
- 量化后的
```shell
python tools/eval.py -c myconfig/ssd_mobilenet_v1.yml --slim_config myconfig/ssd_mobilenet_v1_qat.yml -o weights=output/ssd_mobilenet_v1_qat/best_model
```

---
## 模型转换
```shell
python tools/export_model.py -c myconfig/ssd_mobilenet_v1.yml --slim_config myconfig/ssd_mobilenet_v1_qat.yml -o weights=output/ssd_mobilenet_v1_qat/best_model
```

# 2.在虚拟中的操作

- 首先将上一步转换完成的inference_model的四个文件放到虚拟机中的这个路径下的/opt/nna/Paddle-Lite/build.opt/lite/api/inference文件夹中文件目录如下
- 在传输文件的时候，我建议用Netsarang家的Xftp软件传输文件，vm tools传输文件会有损失
- inference的文件树
├── inference
│   ├── infer_cfg.yml
│   ├── model.pdiparams
│   ├── model.pdiparams.info
│   └── model.pdmodel

```shell
./opt --model_dir=inference/ --valid_targets=intel_fpga,arm --optimize_out_type=naive_buffer --optimize_out=ssd_mobilenet_v1_opt
```
- 转换完成后，生成名为ssd_mobilenet_v1_opt.nb的文件
## 其次将run.sh与run1.sh要放到/opt/nna/paddle_frame文件夹下
- /opt/nna/paddle_frame下的文件树
[root@awcloud paddle_frame]# tree
.
├── 1006_aug_2.jpg
├── 1350_aug_2.jpg
├── 91_aug_3.jpg
├── 98_aug_3.jpg
├── amm_wr_drv.ko
├── cmadrv.ko
├── config.txt
├── debug.sh
├── dog.jpg
├── labels
│   └── label_list
├── paddlelite_lib
│   ├── libgcc_s.so.1
│   ├── libgomp.so.1
│   ├── libpaddle_full_api_shared.so
│   ├── libstdc++.so
│   ├── libstdc++.so.6
│   ├── libstdc++.so.6.0.28
│   └── libvnna.so
├── run1.sh
├── run.sh
├── ssd_detection
└── ssd_mobilenet_v1_opt.nb


# 3.板子上的操作
## 相关登陆信息
- AIEP账户：
```shell
CICC****
```
-  AIEP密码：
```shell
****
```
-  开发板IP：
```shell
*****
```
-  开发板密码：随机

## 现在远程到板子上

- 上传我们自己写的c.c，c.h和相关的编译所需头文件：文件夹名为“include”、“soc_cv_av”与“hps_0.h”
- 要放到/opt/paddle_frame路径下
- 生成c.o文件
```shell
gcc -c c.c -o c.o
```
- 生成c.a文件
```shell
ar rcs c.a c.o
```

# 4.返回到虚拟机上

- 使用Xftp软件去到/opt/nna/ssd_detection_demo/ssd_detection_src目录下
```shell
cd /opt/nna/ssd_detection_demo/ssd_detection_src
```
-  将c.c、c.h、c.o、c.a放到该目录下

---
## 将NEW文件夹中的ssd_detection.cc放到该文件夹下

---
- 在没有重启的情况下不用管这里
 - 配置编译环境路径
 ```shell
export PATH=/opt/software/gcc-linaro-5.4.1-2017.05-x86_64_arm-linux-gnueabihf/bin:$PATH
 ```
- 配置板子IP，这样才能远程连接到板子
 ```shell
export AIEP_HOST=172.16.208.109
 ```
---
- 请更改用户为su
 ```shell
su
 ```
- 密码为“a”
 ```shell
a
 ```
- 编译ssd_detection.cc
 ```shell
./build.demo.sh
 ```

- 将所需的东西靠deploy2aiep.sh上传的板子上
- 这时候会输入密码，这个密码为AIEP软件上随机的密码
```shell
./deploy2aiep.sh
```
## 上传后ssd_detection的路径为/opt/paddle_frame
### 此时需要用命令将上传到板子上的ssd_detection可执行文件的名字换成ssd_new
### 或使用Xftp直接编辑ssd_detection名称，编辑为ssd_new
---

## 将Old文件夹中的ssd_detection.cc放到该文件夹下

---
- 在没有重启的情况下不用管这里
 - 配置编译环境路径
 ```shell
export PATH=/opt/software/gcc-linaro-5.4.1-2017.05-x86_64_arm-linux-gnueabihf/bin:$PATH
 ```
- 配置板子IP，这样才能远程连接到板子
 ```shell
export AIEP_HOST=172.**.**.**
 ```
---
- 请更改用户为su
 ```shell
su
 ```
- 密码为“a”
 ```shell
a
 ```
- 编译ssd_detection.cc
 ```shell
./build.demo.sh
 ```

- 将所需的东西靠deploy2aiep.sh上传的板子上
- 这时候会输入密码，这个密码为AIEP软件上随机的密码
```shell
./deploy2aiep.sh
```
# 5.再到板子上

- 切换路径为/opt/paddle_frame
 ```shell
cd /opt/paddle_frame
 ```

- 配置环境变量
 ```shell
export LD_LIBRARY_PATH=./opencv_lib:$LD_LIBRARY_PATH
 ```
 ```shell
export LD_LIBRARY_PATH=./paddlelite_lib:$LD_LIBRARY_PATH+
 ```

## 运行程序前要打开摄像头换成本地文件格式，打开HDMI
- 运行程序
 ```shell
./ssd_new config.txt
 ```
