# 文书字段检测（YOLO）

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![Ultralytics YOLO11](https://img.shields.io/badge/detector-YOLO11s-green.svg)](https://github.com/ultralytics/ultralytics)

面向**知情同意书**图像的**表单项定位**项目：按文书类型训练 YOLO 模型，在拍照/扫描图上检测「姓名、签字、勾选、牙位」等字段的位置与类别，为后续 OCR、规则审表等算法提供区域输入。

> **Informed consent form field detection for dental clinics.** Per-form-type YOLO models localize labeled form fields on document images; intended as stage-1 layout detection before OCR / business-rule checking.

---

## 项目背景

知情同意书为纸质或拍照表格，版式因模板与页次而异。本项目将「找到每个该填/该签的位置并识别字段类型」建模为**目标检测**任务，采用 **YOLO11s**（[Ultralytics](https://github.com/ultralytics/ultralytics)）按同意书种类分别训练。

典型流水线（当前以检测为主，后处理规划中）：

```
同意书照片 → [文书类型] → YOLO 字段检测 → 裁剪 ROI → OCR / 勾选识别 / 规则校验 → 结构化结果
```

---

<img width="1740" height="306" alt="06e05c86081096586dd4fba0f210e878" src="https://github.com/user-attachments/assets/ea8061f6-1664-4cc5-8196-3ec1957ee060" />
<img width="1671" height="279" alt="822bdbce62f728421e22ee8a91a4b44d" src="https://github.com/user-attachments/assets/dccc7247-9929-498d-91df-2c009dfb2468" />
<img width="1512" height="270" alt="c3e26f7297aa7359dae4bc3ef796d110" src="https://github.com/user-attachments/assets/a478bdcd-0104-4147-a114-da6c6fea2c1f" />


## 主要能力

- 多种同意书 **独立检测模型**（数据、类别、`data.yaml` 分项目维护）
- 标注格式：**AnyLabeling / JSON** → YOLO `labels`；支持数据增强与 train/val 划分
- **训练 / 验证 / 推理 / 评估** 脚本与 Word、CSV 报告
- 四主项目 **多模型路由推理**（按文件名前缀或子文件夹选择 `best.pt`）
- 权重可导出 **ONNX**（各项目 `detect/export_best_onnx.py`）

---

## 重点支持的同意书（四主项目）

| 同意书类型 | 训练目录 | Val 整体表现（参考） |
|------------|----------|----------------------|
| 冠桥修复 | `训练YOLO/冠桥修复知情同意书/detect` | mAP@0.5 ≈ 99.2% |
| 根管治疗 | `训练YOLO/根管治疗知情同意书/detect` | mAP@0.5 ≈ 98.3% |
| 拔牙手术 | `训练YOLO/拔牙手术知情同意书/detect` | mAP@0.5 ≈ 96.2% |
| 牙齿充填 | `训练YOLO/牙齿充填知情同意书/detect` | mAP@0.5 ≈ 89.9% |

另有嵌体、洁牙、牙周、活动义齿等项目的训练目录；小样本类型 Val 仅十余张，指标仅供参考。详细数值见 `训练YOLO/评估报告/val_mAP对比表.csv` 与 Val 逐类报告。

---

## 类别命名约定

检测类别采用 **`字段名-页次-类型`**，例如：

- `姓名-1-n`：第 1 页、非勾选类填空项  
- `性别-1-y-m` / `性别-1-y-f`：第 1 页勾选、男/女  
- `选择-1-n`：勾选框类  

完整类别列表见各项目 `detect/data.yaml` 的 `names`，或根目录脚本生成的 `全部同意书_类别填写表.xlsx`。

---

## 仓库结构

```
同意书/
├── README.md
├── generate_category_xlsx.py      # 从 模型/*/data.yaml 汇总类别到 Excel
├── 新标注/                         # 标注与推理可视化（JSON / 图片）
├── 验证/                           # 与 train/val 独立的试跑图片（按子目录分类型）
├── 模型/                           # 汇总后的 data.yaml / 权重副本（可选）
├── OCR/                            # 早期 OCR / 标注相关实验（与主流程并行）
└── 训练YOLO/
    ├── ultralytics-8.3.163/        # 本地 Ultralytics 源码与 yolo11s.pt 预训练权重
    ├── 评估报告/                   # mAP、逐类准确率 CSV / Word
    ├── evaluate_models.py          # Val mAP + 新标注 JSON 对比（拔牙等）
    ├── class_field_accuracy.py     # Val 逐类召回（字段级）
    ├── predict_sample20_routed.py  # 四模型路由批量画框
    ├── pipeline_three_projects.py  # 增强 → 准备数据 → 训练 → 导出（部分项目）
    └── <同意书名>/detect/
        ├── data.yaml
        ├── images/train|val
        ├── labels/train|val
        ├── train_yolo.py
        ├── prepare_yolo_dataset.py
        ├── export_best_onnx.py
        └── runs/train/weights/best.pt   # 训练后生成（默认不提交 Git）
```

---

## 环境要求

- **OS**：Windows / Linux（脚本在 Windows 上开发，`train_yolo.py` 默认 `workers=0` 以避免 OMP 问题）
- **Python**：3.10+ 推荐  
- **GPU**：CUDA 可选；CPU 可跑推理但训练较慢  
- **依赖**（示例）：`ultralytics`、`opencv-python`、`numpy`、`Pillow`、`pyyaml`；生成报告需 `python-docx`、`openpyxl`

安装示例：

```bash
pip install ultralytics opencv-python numpy pillow pyyaml python-docx openpyxl
```

将仓库中的 `训练YOLO/ultralytics-8.3.163` 作为本地 Ultralytics 使用（各 `train_yolo.py` 已指向该路径）。若克隆后路径变化，请修改各脚本中的 `ULTRALYTICS_ROOT` / `YOLO_ROOT` 或通过 `--data`、`--weights` 传参。

---

## 快速开始

### 1. 准备数据集

在对应项目的 `detect/` 下运行（因项目而异）：

```bash
cd 训练YOLO/根管治疗知情同意书/detect
python prepare_yolo_dataset.py
```

确保 `data.yaml` 中 `train` / `val` 指向 `images/train` 与 `images/val`，且 **train 与 val 无重叠**。

### 2. 训练

```bash
python train_yolo.py --epochs 150 --batch 4 --imgsz 640 --device 0
```

权重默认输出：`runs/train/weights/best.pt`（由 Val mAP 选优，Val **不参与梯度更新**）。

### 3. 验证（mAP）

```bash
cd 训练YOLO
python evaluate_models.py
```

结果写入 `评估报告/val_mAP对比表.csv`。

### 4. 逐类字段召回（Val）

```bash
python class_field_accuracy.py --split val --device 0
```

生成 `评估报告/类别准确率_<项目>_val.csv` 与 `全部同意书_各类别准确率_val.csv`。

### 5. 生成 Val 报告（Word，四主项目）

```bash
python 评估报告/generate_val_class_report_docx.py
```

输出：`评估报告/Val验证集_各类别准确率报告.docx`。

### 6. 批量推理（四模型路由）

```bash
python predict_sample20_routed.py --source 新标注/每类抽样20张 --out 新标注/每类抽样20张/推理覆盖
```

按文件名 `同意书名__原图.jpg` 或 `--source` 下子文件夹名（如 `验证/根管治疗`）自动选择模型。

---

## 评估指标说明

| 指标 | 含义 |
|------|------|
| **mAP@0.5** | YOLO 官方 Val 检测均值，便于与通用检测论文对比 |
| **Precision / Recall** | 全框层面的精确率 / 召回率 |
| **字段级「准确率」**（本仓库脚本） | 实为 **Recall = 预测正确数 / 该字段标注总数**（IoU≥0.5，同类匹配） |

---

## 数据与隐私

- 知情同意书含**患者隐私**，请勿将含真实信息的原图、JSON 标注提交到公开仓库。  
- 建议仅提交代码、脱敏样例、`data.yaml` 类别定义；大图与 `best.pt` 使用 Git LFS 或私有存储。  
- 根目录 `.gitignore` 已忽略常见权重、运行目录与本地可执行文件（可按需调整）。

---

## 路线图

- [x] 多类型同意书 YOLO 训练与 Val 评估  
- [x] 四主项目路由推理与报告导出  
- [ ] 独立 **测试集**（`验证/`）上的统一评估脚本  
- [ ] 检测框裁剪 + **OCR / 手写识别**  
- [ ] 勾选、空白检测与 **必填项 / 业务规则** 审表  
- [ ] 文书类型自动分类（单入口多模板）

---

## 引用与致谢

检测框架基于 [Ultralytics YOLO](https://github.com/ultralytics/ultralytics)。若在研究中使用了本项目思路或脚本，请自行引用 YOLO 原文及本仓库（待补充 DOI / 论文信息）。

---

## License

未指定开源协议前，代码仅供学习与研究参考；数据集与模型权重版权归原作者所有。如需商用或二次分发，请先联系维护者并确保符合医疗数据合规要求。
