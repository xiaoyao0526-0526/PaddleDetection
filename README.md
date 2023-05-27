# 准备环境

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
# 训练全精度模型
- 训练全精度模型需要用到3个原版py脚本
<ol>
<li>PaddleDetection/ppdet/modeling/layers.py</li> 
<li>PaddleDetection/ppdet/modeling/heads/ssd_head.py</li>
<li>PaddleDetection/ppdet/slim/quant.py</li>
</ol>

```shell
python tools/train.py -c myconfig/ssd_mobilenet_v1.yml --eval
```
# 全精度模型推理

```shell
python tools/infer.py -c myconfig/ssd_mobilenet_v1.yml -o weights=output/ssd_mobilenet_v1/best_model --infer_img=dataset/aluminum_ssd_voc/images/1003.jpg --draw_threshold=0.5
```
# 在线量化训练
- 训练量化模型需要替换3个py脚本（还是自己上传吧，本来想写个脚本替换的，没整成）
```shell
python tools/train.py -c myconfig/ssd_mobilenet_v1.yml --slim_config myconfig/ssd_mobilenet_v1_qat.yml --eval
```
# 量化模型推理
```shell
python tools/infer.py -c myconfig/ssd_mobilenet_v1.yml --slim_config myconfig/ssd_mobilenet_v1_qat.yml -o weights=output/ssd_mobilenet_v1_qat/best_model --infer_img=dataset/aluminum_ssd_voc/images/1003.jpg --draw_threshold=0.5
```
- 推理到一个文件夹
```shell
python -u tools/infer.py -c myconfig/ssd_mobilenet_v1.yml --slim_config myconfig/ssd_mobilenet_v1_qat.yml -o weights=output/ssd_mobilenet_v1_qat/best_model --infer_dir=dataset/aluminum_ssd_voc/images --output_dir=infer_output/testout
```

---
# 查看模型精度
```shell
python tools/eval.py -c myconfig/ssd_mobilenet_v1.yml -o weights=output/ssd_mobilenet_v1/best_model
```
```shell
python tools/eval.py -c myconfig/ssd_mobilenet_v1.yml --slim_config myconfig/ssd_mobilenet_v1_qat.yml -o weights=output/ssd_mobilenet_v1_qat/best_model
```

---
# 模型转换
```shell
python tools/export_model.py -c myconfig/ssd_mobilenet_v1.yml --slim_config myconfig/ssd_mobilenet_v1_qat.yml -o weights=output/ssd_mobilenet_v1_qat/best_model
```
- 在虚拟机里运行
```shell
./opt --model_dir=inference/ --valid_targets=intel_fpga,arm --optimize_out_type=naive_buffer --optimize_out=ssd_mobilenet_v1_opt
```
