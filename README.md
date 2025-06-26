# zairyuocrkuren
大規模モデルで小型ocrモデルを訓練
🧾 PaddleOCR 检测+识别（det+rec）模型训练方案
🔍 1. 项目目标
使用在留卡图像进行字段级文本检测与识别训练

目标部署环境：轻量设备 / 中低端 GPU / 本地服务器

教师模型来源：DeepSeek-VL2 生成伪标注

⚙️ 2. 技术选型对比
项目	PaddleOCR（det+rec）	DeepSeek-VL（大模型）
功能	文本框检测 + OCR识别	多模态推理（字段提取）
精度	中-高（依赖数据质量）	高（zero-shot能力强）
推理速度	快（可部署边缘设备）	慢（需大GPU，响应慢）
训练资源需求	中（消费级GPU可用）	非常高（A100/H100推荐）
模型大小	5MB～100MB 可选	数十GB
开源许可	Apache 2.0	DeepSeek社群/非商用可用

结论：PaddleOCR适合作为部署模型，通过从 DeepSeek 学习获得准确识别能力。

📁 3. 数据格式（det+rec）
每张图像配备一个对应标注格式如下：

plaintext
コピーする
編集する
images/img_001.jpg	[
  {"transcription": "氏名", "points": [[30, 40], [130, 40], [130, 65], [30, 65]]},
  {"transcription": "ZHA JUNJIE", "points": [[140, 40], [300, 40], [300, 65], [140, 65]]}
]
points：为四点坐标（顺时针）表示文本框位置

transcription：为该区域的识别目标文本

🧪 4. 性能表现参考（PaddleOCR官方基准）
模型名	任务	训练集	精度	单图推理时间（CPU）	备注
PP-OCRv3-Mobile	det+rec	OCR联合任务	F-score > 85%	~70ms	非常适合轻量部署
PP-OCRv3-Server	det+rec	OCR联合任务	F-score > 90%	~30ms（GPU）	精度高，适合服务器端

🏗 5. 训练计划（Training Plan）
📦 数据准备阶段
使用 DeepSeek 推理生成的字段识别结果 + 图像

自动转换为 label.txt 格式，包括位置与文本

🧠 模型训练阶段
🔹 推荐配置文件：
bash
コピーする
編集する
configs/ppocrv3/...
🔹 示例训练命令：
bash
コピーする
編集する
python tools/train.py \
  -c configs/ppocrv3/ch_PP-OCRv3_det_rec.yml \
  -o Global.pretrained_model=./pretrain/ \
     Global.save_model_dir=./output/zairyu_ocr/ \
     Train.dataset.data_dir=./dataset/ \
     Train.dataset.label_file_list=["./dataset/label.txt"]
🔹 超参数（可调整）：
项目	默认值	建议备注
Epoch	100	50~100
Batch Size	32~128	视显存情况而定
学习率	0.001	可使用 CosineDecay
训练时长	1~4 小时	RTX 3060 约需 2h

🚀 6. 模型导出与部署
bash
コピーする
編集する
python tools/export_model.py \
  -c configs/ppocrv3/ch_PP-OCRv3_det_rec.yml \
  -o Global.pretrained_model=output/zairyu_ocr/best_accuracy \
     Global.save_inference_dir=./inference/zairyu_ocr/
导出后可通过：

Python 推理

C++ SDK 部署

Paddle Lite → 安卓设备

ONNX → TensorRT部署

✅ 7. 优势总结
关键指标	内容
成本	可在消费级 GPU 训练
效果	接近大型模型水准（通过蒸馏）
速度	单图<80ms，适合实时系统
部署	支持边缘端（Raspberry Pi、Jetson）
兼容	可与DeepSeek共同使用做迭代训练
