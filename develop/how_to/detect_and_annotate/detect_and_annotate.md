---
comments: true
---

# Detect and Annotate

Supervision provides a seamless process for annotating predictions generated by various
object detection and segmentation models. This guide shows how to perform inference
with the [Inference](https://github.com/roboflow/inference),
[Ultralytics](https://github.com/ultralytics/ultralytics) or
[Transformers](https://github.com/huggingface/transformers) packages. Following this,
you'll learn how to import these predictions into Supervision and use them to annotate
source image.

![basic-annotation](https://media.roboflow.com/supervision_detect_and_annotate_example_1.png)

## Run Detection

First, you'll need to obtain predictions from your object detection or segmentation
model.

=== "Inference"

    ```python
    import cv2
    from inference import get_model

    model = get_model(model_id="yolov8n-640")
    image = cv2.imread(<SOURCE_IMAGE_PATH>)
    results = model.infer(image)[0]
    ```

=== "Ultralytics"

    ```python
    import cv2
    from ultralytics import YOLO

    model = YOLO("yolov8n.pt")
    image = cv2.imread(<SOURCE_IMAGE_PATH>)
    results = model(image)[0]
    ```

=== "Transformers"

    ```python
    import torch
    from PIL import Image
    from transformers import DetrImageProcessor, DetrForObjectDetection

    processor = DetrImageProcessor.from_pretrained("facebook/detr-resnet-50")
    model = DetrForObjectDetection.from_pretrained("facebook/detr-resnet-50")

    image = Image.open(<SOURCE_IMAGE_PATH>)
    inputs = processor(images=image, return_tensors="pt")

    with torch.no_grad():
        outputs = model(**inputs)

    width, height = image.size
    target_size = torch.tensor([[height, width]])
    results = processor.post_process_object_detection(
        outputs=outputs, target_sizes=target_size)[0]
    ```

## Load Predictions into Supervision

Now that we have predictions from a model, we can load them into Supervision.

=== "Inference"

    We can do so using the [`sv.Detections.from_inference`](/latest/detection/core/#supervision.detection.core.Detections.from_inference) method, which accepts model results from both detection and segmentation models.

    ```{ .py hl_lines="2 8" }
    import cv2
    import supervision as sv
    from inference import get_model

    model = get_model(model_id="yolov8n-640")
    image = cv2.imread(<SOURCE_IMAGE_PATH>)
    results = model.infer(image)[0]
    detections = sv.Detections.from_inference(results)
    ```

=== "Ultralytics"

    We can do so using the [`sv.Detections.from_ultralytics`](/latest/detection/core/#supervision.detection.core.Detections.from_ultralytics) method, which accepts model results from both detection and segmentation models.

    ```{ .py hl_lines="2 8" }
    import cv2
    import supervision as sv
    from ultralytics import YOLO

    model = YOLO("yolov8n.pt")
    image = cv2.imread(<SOURCE_IMAGE_PATH>)
    results = model(image)[0]
    detections = sv.Detections.from_ultralytics(results)
    ```

=== "Transformers"

    We can do so using the [`sv.Detections.from_transformers`](/latest/detection/core/#supervision.detection.core.Detections.from_transformers) method, which accepts model results from both detection and segmentation models.

    ```{ .py hl_lines="2 19-21" }
    import torch
    import supervision as sv
    from PIL import Image
    from transformers import DetrImageProcessor, DetrForObjectDetection

    processor = DetrImageProcessor.from_pretrained("facebook/detr-resnet-50")
    model = DetrForObjectDetection.from_pretrained("facebook/detr-resnet-50")

    image = Image.open(<SOURCE_IMAGE_PATH>)
    inputs = processor(images=image, return_tensors="pt")

    with torch.no_grad():
        outputs = model(**inputs)

    width, height = image.size
    target_size = torch.tensor([[height, width]])
    results = processor.post_process_object_detection(
        outputs=outputs, target_sizes=target_size)[0]
    detections = sv.Detections.from_transformers(
        transformers_results=results,
        id2label=model.config.id2label)
    ```

You can load predictions from other computer vision frameworks and libraries using:

- [`from_deepsparse`](/latest/detection/core/#supervision.detection.core.Detections.from_deepsparse) ([Deepsparse](https://github.com/neuralmagic/deepsparse))
- [`from_detectron2`](/latest/detection/core/#supervision.detection.core.Detections.from_detectron2) ([Detectron2](https://github.com/facebookresearch/detectron2))
- [`from_mmdetection`](/latest/detection/core/#supervision.detection.core.Detections.from_mmdetection) ([MMDetection](https://github.com/open-mmlab/mmdetection))
- [`from_sam`](/latest/detection/core/#supervision.detection.core.Detections.from_sam) ([Segment Anything Model](https://github.com/facebookresearch/segment-anything))
- [`from_yolo_nas`](/latest/detection/core/#supervision.detection.core.Detections.from_yolo_nas) ([YOLO-NAS](https://github.com/Deci-AI/super-gradients/blob/master/YOLONAS.md))

## Annotate Image with Detections

Finally, we can annotate the image with the predictions. Since we are working with an object detection model, we will use the [`sv.BoxAnnotator`](/latest/detection/annotators/#supervision.annotators.core.BoxAnnotator) and [`sv.LabelAnnotator`](/latest/detection/annotators/#supervision.annotators.core.LabelAnnotator) classes.

=== "Inference"

    ```{ .py hl_lines="10-16" }
    import cv2
    import supervision as sv
    from inference import get_model

    model = get_model(model_id="yolov8n-640")
    image = cv2.imread(<SOURCE_IMAGE_PATH>)
    results = model.infer(image)[0]
    detections = sv.Detections.from_inference(results)

    box_annotator = sv.BoxAnnotator()
    label_annotator = sv.LabelAnnotator()

    annotated_image = box_annotator.annotate(
        scene=image, detections=detections)
    annotated_image = label_annotator.annotate(
        scene=annotated_image, detections=detections)
    ```

=== "Ultralytics"

    ```{ .py hl_lines="10-16" }
    import cv2
    import supervision as sv
    from ultralytics import YOLO

    model = YOLO("yolov8n.pt")
    image = cv2.imread(<SOURCE_IMAGE_PATH>)
    results = model(image)[0]
    detections = sv.Detections.from_ultralytics(results)

    box_annotator = sv.BoxAnnotator()
    label_annotator = sv.LabelAnnotator()

    annotated_image = box_annotator.annotate(
        scene=image, detections=detections)
    annotated_image = label_annotator.annotate(
        scene=annotated_image, detections=detections)
    ```

=== "Transformers"

    ```{ .py hl_lines="23-30" }
    import torch
    import supervision as sv
    from PIL import Image
    from transformers import DetrImageProcessor, DetrForObjectDetection

    processor = DetrImageProcessor.from_pretrained("facebook/detr-resnet-50")
    model = DetrForObjectDetection.from_pretrained("facebook/detr-resnet-50")

    image = Image.open(<SOURCE_IMAGE_PATH>)
    inputs = processor(images=image, return_tensors="pt")

    with torch.no_grad():
        outputs = model(**inputs)

    width, height = image.size
    target_size = torch.tensor([[height, width]])
    results = processor.post_process_object_detection(
        outputs=outputs, target_sizes=target_size)[0]
    detections = sv.Detections.from_transformers(
        transformers_results=results,
        id2label=model.config.id2label)

    box_annotator = sv.BoxAnnotator()
    label_annotator = sv.LabelAnnotator()

    annotated_image = box_annotator.annotate(
        scene=image, detections=detections)
    annotated_image = label_annotator.annotate(
        scene=annotated_image, detections=detections)
    ```

![basic-annotation](https://media.roboflow.com/supervision_detect_and_annotate_example_1.png)

## Display Custom Labels

By default, [`sv.LabelAnnotator`](/latest/detection/annotators/#supervision.annotators.core.LabelAnnotator)
will label each detection with its `class_name` (if possible) or `class_id`. You can
override this behavior by passing a list of custom `labels` to the `annotate` method.

=== "Inference"

    ```{ .py hl_lines="13-17 22" }
    import cv2
    import supervision as sv
    from inference import get_model

    model = get_model(model_id="yolov8n-640")
    image = cv2.imread(<SOURCE_IMAGE_PATH>)
    results = model.infer(image)[0]
    detections = sv.Detections.from_inference(results)

    box_annotator = sv.BoxAnnotator()
    label_annotator = sv.LabelAnnotator()

    labels = [
        f"{class_name} {confidence:.2f}"
        for class_name, confidence
        in zip(detections['class_name'], detections.confidence)
    ]

    annotated_image = box_annotator.annotate(
        scene=image, detections=detections)
    annotated_image = label_annotator.annotate(
        scene=annotated_image, detections=detections, labels=labels)
    ```

=== "Ultralytics"

    ```{ .py hl_lines="13-17 22" }
    import cv2
    import supervision as sv
    from ultralytics import YOLO

    model = YOLO("yolov8n.pt")
    image = cv2.imread(<SOURCE_IMAGE_PATH>)
    results = model(image)[0]
    detections = sv.Detections.from_ultralytics(results)

    box_annotator = sv.BoxAnnotator()
    label_annotator = sv.LabelAnnotator()

    labels = [
        f"{class_name} {confidence:.2f}"
        for class_name, confidence
        in zip(detections['class_name'], detections.confidence)
    ]

    annotated_image = box_annotator.annotate(
        scene=image, detections=detections)
    annotated_image = label_annotator.annotate(
        scene=annotated_image, detections=detections, labels=labels)
    ```

=== "Transformers"

    ```{ .py hl_lines="26-30 35" }
    import torch
    import supervision as sv
    from PIL import Image
    from transformers import DetrImageProcessor, DetrForObjectDetection

    processor = DetrImageProcessor.from_pretrained("facebook/detr-resnet-50")
    model = DetrForObjectDetection.from_pretrained("facebook/detr-resnet-50")

    image = Image.open(<SOURCE_IMAGE_PATH>)
    inputs = processor(images=image, return_tensors="pt")

    with torch.no_grad():
        outputs = model(**inputs)

    width, height = image.size
    target_size = torch.tensor([[height, width]])
    results = processor.post_process_object_detection(
        outputs=outputs, target_sizes=target_size)[0]
    detections = sv.Detections.from_transformers(
        transformers_results=results,
        id2label=model.config.id2label)

    box_annotator = sv.BoxAnnotator()
    label_annotator = sv.LabelAnnotator()

    labels = [
        f"{class_name} {confidence:.2f}"
        for class_name, confidence
        in zip(detections['class_name'], detections.confidence)
    ]

    annotated_image = box_annotator.annotate(
        scene=image, detections=detections)
    annotated_image = label_annotator.annotate(
        scene=annotated_image, detections=detections, labels=labels)
    ```

![custom-label-annotation](https://media.roboflow.com/supervision_detect_and_annotate_example_2.png)

## Annotate Image with Segmentations

If you are running the segmentation model
[`sv.MaskAnnotator`](/latest/detection/annotators/#supervision.annotators.core.MaskAnnotator)
is a drop-in replacement for
[`sv.BoxAnnotator`](/latest/detection/annotators/#supervision.annotators.core.BoxAnnotator)
that will allow you to draw masks instead of boxes.

=== "Inference"

    ```python
    import cv2
    import supervision as sv
    from inference import get_model

    model = get_model(model_id="yolov8n-seg-640")
    image = cv2.imread(<SOURCE_IMAGE_PATH>)
    results = model.infer(image)[0]
    detections = sv.Detections.from_inference(results)

    mask_annotator = sv.MaskAnnotator()
    label_annotator = sv.LabelAnnotator(text_position=sv.Position.CENTER_OF_MASS)

    annotated_image = mask_annotator.annotate(
        scene=image, detections=detections)
    annotated_image = label_annotator.annotate(
        scene=annotated_image, detections=detections)
    ```

=== "Ultralytics"

    ```python
    import cv2
    import supervision as sv
    from ultralytics import YOLO

    model = YOLO("yolov8n-seg.pt")
    image = cv2.imread(<SOURCE_IMAGE_PATH>)
    results = model(image)[0]
    detections = sv.Detections.from_ultralytics(results)

    mask_annotator = sv.MaskAnnotator()
    label_annotator = sv.LabelAnnotator(text_position=sv.Position.CENTER_OF_MASS)

    annotated_image = mask_annotator.annotate(
        scene=image, detections=detections)
    annotated_image = label_annotator.annotate(
        scene=annotated_image, detections=detections)
    ```

=== "Transformers"

    ```python
    import torch
    import supervision as sv
    from PIL import Image
    from transformers import DetrImageProcessor, DetrForSegmentation

    processor = DetrImageProcessor.from_pretrained("facebook/detr-resnet-50-panoptic")
    model = DetrForSegmentation.from_pretrained("facebook/detr-resnet-50-panoptic")

    image = Image.open(<SOURCE_IMAGE_PATH>)
    inputs = processor(images=image, return_tensors="pt")

    with torch.no_grad():
        outputs = model(**inputs)

    width, height = image.size
    target_size = torch.tensor([[height, width]])
    results = processor.post_process_segmentation(
        outputs=outputs, target_sizes=target_size)[0]
    detections = sv.Detections.from_transformers(
        transformers_results=results,
        id2label=model.config.id2label)

    mask_annotator = sv.MaskAnnotator()
    label_annotator = sv.LabelAnnotator(text_position=sv.Position.CENTER_OF_MASS)

    labels = [
        f"{class_name} {confidence:.2f}"
        for class_name, confidence
        in zip(detections['class_name'], detections.confidence)
    ]

    annotated_image = mask_annotator.annotate(
        scene=image, detections=detections)
    annotated_image = label_annotator.annotate(
        scene=annotated_image, detections=detections, labels=labels)
    ```

![segmentation-annotation](https://media.roboflow.com/supervision_detect_and_annotate_example_3.png)