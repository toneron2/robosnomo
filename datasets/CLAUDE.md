# Datasets - Training Data & AI Models
## Project RoboSnomo | Winter Environment ML Training

**Parent Project**: [Project RoboSnomo](/storage/emulated/0/Documents/projects/robosnomo/CLAUDE.md)
**Project Root**: `/storage/emulated/0/Documents/projects/robosnomo/`

---

### Scope

Winter-specific image datasets, trained AI models, and TensorRT optimized engines for autonomous navigation.

**Responsibilities**:
- Winter obstacle dataset collection (15,000+ labeled images)
- YOLOv8 training (trees, rocks, trails, snowmobiles, animals)
- Semantic segmentation training (snow, ice, vegetation, trail classification)
- TensorRT optimization (INT8 quantization for 65 FPS on Jetson)
- Model versioning and performance benchmarking

**Dataset Sources**:
- RSOD (Remote Sensing Object Detection) - starting point
- Cityscapes (augmented with winter conditions)
- Custom collection: Field photography in winter environments

**Directory Structure**:
```
datasets/
├── CLAUDE.md
├── raw-images/
│   ├── training/      # 80% split
│   ├── validation/    # 10% split
│   └── test/          # 10% split
├── annotations/       # YOLO format labels, COCO JSON
├── models/
│   ├── yolov8n/       # PyTorch checkpoints
│   ├── deeplabv3/     # Segmentation models
│   └── tensorrt/      # Optimized .engine files
├── benchmarks/        # FPS, mAP, power consumption logs
└── scripts/
    ├── data_augmentation.py
    ├── train_yolo.py
    └── tensorrt_export.py
```

**Key References**: `/layer1-brain/CLAUDE.md` - Vision system requirements

**Note**: Large files (>100MB) will use Git LFS when repository initialized.

**Status**: Phase 1 - Planning (Dataset collection pending field testing)

---

**Last Updated**: November 18, 2025
