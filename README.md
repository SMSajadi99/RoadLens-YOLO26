<div align="center">

# 🚦 RoadLens · YOLO26

**Object detection from the driver's perspective**

![Model: YOLO26n](https://img.shields.io/badge/model-YOLO26n-164E63?style=flat-square)
![Road classes: 8](https://img.shields.io/badge/road_classes-8-047857?style=flat-square)
![Input: dashcam footage](https://img.shields.io/badge/input-dashcam_footage-B45309?style=flat-square)

[Try a road clip](#try-a-road-clip) · [Explore the classes](#what-appears-in-the-frame) · [Train on your footage](#train-on-your-footage)

</div>

At a busy intersection, a cyclist can pass beside a truck while a pedestrian steps into the same frame. RoadLens uses YOLO26 to locate each visible road user and traffic control separately. Each detection has a bounding box, a class label, and a confidence score.

## Try a road clip

Install Ultralytics, place a dashcam clip named `drive.mp4` in your working directory, and run:

```bash
python -m pip install --upgrade ultralytics
yolo detect predict model=yolo26n.pt source="drive.mp4" classes=0,1,2,3,5,7,9,11 conf=0.25 save=True project=outputs name=road-demo
```

The first run downloads the pretrained weights. Open the annotated video in the output directory printed by the command (initially `outputs/road-demo/`). To inspect a single intersection, replace `drive.mp4` with the path to a street photograph.

## What appears in the frame?

| In the scene | Detection label | Pretrained COCO ID |
| --- | --- | ---: |
| A pedestrian or the rider on a bike | `person` | 0 |
| A bicycle | `bicycle` | 1 |
| A passenger car | `car` | 2 |
| A motorcycle | `motorcycle` | 3 |
| A bus | `bus` | 5 |
| A truck | `truck` | 7 |
| A traffic signal | `traffic light` | 9 |
| A stop sign | `stop sign` | 11 |

A cyclist can produce two boxes, one for the rider and another for the bicycle. The traffic-light label locates the signal housing. Recognizing red, amber, or green requires state labels or a separate classifier.

## Train on your footage

Split the data by drive so adjacent frames from one recording stay together. Under a dataset directory, create `images/train`, `images/val`, `labels/train`, and `labels/val`. Pair each image with a same-stem label file, such as `images/train/frame012.jpg` and `labels/train/frame012.txt`.

Save the following as `road.yaml`, replacing the example path with the absolute path to your dataset:

```yaml
path: /absolute/path/to/road-scenes
train: images/train
val: images/val
names: [person, bicycle, car, motorcycle, bus, truck, traffic light, stop sign]
```

Each annotation line uses `class_id x_center y_center width height`, with box coordinates normalized to the image dimensions. This custom dataset assigns IDs **0–7** in the order above. For example, a truck is class **5** in these label files and class **7** in the pretrained COCO model.

```bash
yolo detect train model=yolo26n.pt data=road.yaml epochs=50 imgsz=640 project=outputs name=road-train
yolo detect val model=outputs/road-train/weights/best.pt data=road.yaml
yolo detect predict model=outputs/road-train/weights/best.pt source="drive.mp4" save=True
```

These paths assume the first training run. For subsequent runs, use the checkpoint path printed at the end of training. The fine-tuned model already uses the eight custom classes, so the final prediction command needs no COCO class filter.

## Evaluate a drive

Review per-class precision, recall, and mAP50–95 alongside the annotated frames. A missed distant signal and a rider missed behind a bus call for different training examples, even when both reduce overall recall. For a speed measurement, record the model, image size, hardware, and whether video decoding and output encoding are included.

## References

[Ultralytics YOLO26](https://docs.ultralytics.com/models/yolo26/) · [COCO class map](https://github.com/ultralytics/ultralytics/blob/main/ultralytics/cfg/datasets/coco.yaml) · [Custom detection datasets](https://docs.ultralytics.com/datasets/detect/) · [Related person-detection project](https://github.com/SMSajadi99/Custom-Data-YOLOv8-Person-Detection)
