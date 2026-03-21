# 4.2 Computer Vision

## Course Overview

Computer Vision (CV) enables machines to interpret and understand visual information from the world — images, videos, and 3D scenes. This subject covers the full pipeline from classical image processing to modern deep learning-based approaches for detection, segmentation, tracking, and recognition.

---

## Prerequisites

- Python programming
- Linear algebra (matrices, transformations)
- Deep learning fundamentals (CNNs)
- Basic probability and statistics

---

## Table of Contents

### Unit 1: Introduction to Computer Vision
| # | Topic | Link |
|---|-------|------|
| 1 | What is Computer Vision? | [01-what-is-cv.md](01-Introduction-to-CV/01-what-is-cv.md) |
| 2 | Applications of CV | [02-applications.md](01-Introduction-to-CV/02-applications.md) |
| 3 | Image Fundamentals | [03-image-fundamentals.md](01-Introduction-to-CV/03-image-fundamentals.md) |
| 4 | Color Spaces (RGB, HSV, LAB) | [04-color-spaces.md](01-Introduction-to-CV/04-color-spaces.md) |
| 5 | Images as Matrices | [05-images-as-matrices.md](01-Introduction-to-CV/05-images-as-matrices.md) |

### Unit 2: Image Processing Basics
| # | Topic | Link |
|---|-------|------|
| 1 | Image Filtering | [01-image-filtering.md](02-Image-Processing-Basics/01-image-filtering.md) |
| 2 | Convolution Operations | [02-convolution-operations.md](02-Image-Processing-Basics/02-convolution-operations.md) |
| 3 | Edge Detection (Sobel, Canny) | [03-edge-detection.md](02-Image-Processing-Basics/03-edge-detection.md) |
| 4 | Image Gradients | [04-image-gradients.md](02-Image-Processing-Basics/04-image-gradients.md) |
| 5 | Morphological Operations | [05-morphological-operations.md](02-Image-Processing-Basics/05-morphological-operations.md) |
| 6 | Histogram Operations | [06-histogram-operations.md](02-Image-Processing-Basics/06-histogram-operations.md) |

### Unit 3: Feature Detection
| # | Topic | Link |
|---|-------|------|
| 1 | Corners (Harris, Shi-Tomasi) | [01-corner-detection.md](03-Feature-Detection/01-corner-detection.md) |
| 2 | SIFT | [02-sift.md](03-Feature-Detection/02-sift.md) |
| 3 | SURF | [03-surf.md](03-Feature-Detection/03-surf.md) |
| 4 | ORB | [04-orb.md](03-Feature-Detection/04-orb.md) |
| 5 | Feature Matching | [05-feature-matching.md](03-Feature-Detection/05-feature-matching.md) |
| 6 | Homography | [06-homography.md](03-Feature-Detection/06-homography.md) |

### Unit 4: Traditional CV Methods
| # | Topic | Link |
|---|-------|------|
| 1 | Template Matching | [01-template-matching.md](04-Traditional-CV-Methods/01-template-matching.md) |
| 2 | Histogram of Oriented Gradients (HOG) | [02-hog.md](04-Traditional-CV-Methods/02-hog.md) |
| 3 | HAAR Cascades | [03-haar-cascades.md](04-Traditional-CV-Methods/03-haar-cascades.md) |
| 4 | Optical Flow | [04-optical-flow.md](04-Traditional-CV-Methods/04-optical-flow.md) |
| 5 | Image Segmentation (Watershed, GrabCut) | [05-image-segmentation.md](04-Traditional-CV-Methods/05-image-segmentation.md) |

### Unit 5: Image Classification
| # | Topic | Link |
|---|-------|------|
| 1 | Problem Formulation | [01-problem-formulation.md](05-Image-Classification/01-problem-formulation.md) |
| 2 | CNN-based Classification | [02-cnn-classification.md](05-Image-Classification/02-cnn-classification.md) |
| 3 | Transfer Learning | [03-transfer-learning.md](05-Image-Classification/03-transfer-learning.md) |
| 4 | Fine-tuning Pre-trained Models | [04-fine-tuning.md](05-Image-Classification/04-fine-tuning.md) |
| 5 | Multi-label Classification | [05-multi-label.md](05-Image-Classification/05-multi-label.md) |

### Unit 6: Object Detection
| # | Topic | Link |
|---|-------|------|
| 1 | Problem Formulation | [01-problem-formulation.md](06-Object-Detection/01-problem-formulation.md) |
| 2 | Two-stage Detectors (R-CNN Family) | [02-two-stage-detectors.md](06-Object-Detection/02-two-stage-detectors.md) |
| 3 | Single-stage Detectors (YOLO, SSD) | [03-single-stage-detectors.md](06-Object-Detection/03-single-stage-detectors.md) |
| 4 | Anchor-based vs Anchor-free | [04-anchor-methods.md](06-Object-Detection/04-anchor-methods.md) |
| 5 | Non-Maximum Suppression | [05-nms.md](06-Object-Detection/05-nms.md) |
| 6 | Evaluation Metrics (mAP, IoU) | [06-evaluation-metrics.md](06-Object-Detection/06-evaluation-metrics.md) |

### Unit 7: Image Segmentation
| # | Topic | Link |
|---|-------|------|
| 1 | Semantic Segmentation | [01-semantic-segmentation.md](07-Image-Segmentation/01-semantic-segmentation.md) |
| 2 | Instance Segmentation | [02-instance-segmentation.md](07-Image-Segmentation/02-instance-segmentation.md) |
| 3 | Panoptic Segmentation | [03-panoptic-segmentation.md](07-Image-Segmentation/03-panoptic-segmentation.md) |
| 4 | FCN, U-Net, DeepLab | [04-segmentation-architectures.md](07-Image-Segmentation/04-segmentation-architectures.md) |
| 5 | Mask R-CNN | [05-mask-rcnn.md](07-Image-Segmentation/05-mask-rcnn.md) |
| 6 | Evaluation Metrics | [06-evaluation-metrics.md](07-Image-Segmentation/06-evaluation-metrics.md) |

### Unit 8: Object Tracking
| # | Topic | Link |
|---|-------|------|
| 1 | Single Object Tracking | [01-single-object-tracking.md](08-Object-Tracking/01-single-object-tracking.md) |
| 2 | Multi-Object Tracking | [02-multi-object-tracking.md](08-Object-Tracking/02-multi-object-tracking.md) |
| 3 | Tracking Algorithms (KCF, SORT, DeepSORT) | [03-tracking-algorithms.md](08-Object-Tracking/03-tracking-algorithms.md) |
| 4 | Re-identification | [04-re-identification.md](08-Object-Tracking/04-re-identification.md) |

### Unit 9: Face Analysis
| # | Topic | Link |
|---|-------|------|
| 1 | Face Detection | [01-face-detection.md](09-Face-Analysis/01-face-detection.md) |
| 2 | Facial Landmark Detection | [02-facial-landmarks.md](09-Face-Analysis/02-facial-landmarks.md) |
| 3 | Face Recognition | [03-face-recognition.md](09-Face-Analysis/03-face-recognition.md) |
| 4 | Face Verification vs Identification | [04-verification-vs-identification.md](09-Face-Analysis/04-verification-vs-identification.md) |
| 5 | Emotion Recognition | [05-emotion-recognition.md](09-Face-Analysis/05-emotion-recognition.md) |
| 6 | DeepFace, FaceNet, ArcFace | [06-face-models.md](09-Face-Analysis/06-face-models.md) |

### Unit 10: Advanced CV Topics
| # | Topic | Link |
|---|-------|------|
| 1 | 3D Vision Basics | [01-3d-vision.md](10-Advanced-CV-Topics/01-3d-vision.md) |
| 2 | Depth Estimation | [02-depth-estimation.md](10-Advanced-CV-Topics/02-depth-estimation.md) |
| 3 | Pose Estimation | [03-pose-estimation.md](10-Advanced-CV-Topics/03-pose-estimation.md) |
| 4 | Image Generation (GANs Preview) | [04-image-generation.md](10-Advanced-CV-Topics/04-image-generation.md) |
| 5 | Video Understanding | [05-video-understanding.md](10-Advanced-CV-Topics/05-video-understanding.md) |
| 6 | Self-supervised Learning in CV | [06-self-supervised-cv.md](10-Advanced-CV-Topics/06-self-supervised-cv.md) |

---

## Learning Path

```
Unit 1-2: Foundations     → Image basics and processing
Unit 3-4: Classical CV    → Feature detection and traditional methods
Unit 5:   Classification  → CNN-based image classification
Unit 6:   Detection       → Locating objects in images
Unit 7:   Segmentation    → Pixel-level understanding
Unit 8:   Tracking        → Following objects in video
Unit 9:   Faces           → Specialized face analysis
Unit 10:  Advanced        → 3D, generation, video, SSL
```

---

## Key Libraries

| Library | Purpose |
|---------|---------|
| OpenCV | Image processing, classical CV |
| PyTorch / torchvision | Deep learning models |
| Detectron2 | Object detection & segmentation |
| Ultralytics | YOLO models |
| MediaPipe | Face/pose/hand detection |
| Albumentations | Image augmentation |
