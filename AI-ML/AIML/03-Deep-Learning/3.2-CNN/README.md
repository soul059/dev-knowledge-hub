# 🖼️ Convolutional Neural Networks (CNN)

## Course Overview

This module covers Convolutional Neural Networks — the architecture that revolutionized computer vision. From understanding convolution operations to classic and modern architectures (LeNet to EfficientNet), training strategies, and applications (classification, detection, segmentation), these notes provide comprehensive coverage of CNNs.

---

## 📋 Table of Contents

### Unit 1: Introduction to CNNs
| # | Topic | File |
|---|-------|------|
| 1 | Why CNNs for Images? | [01-why-cnns-for-images.md](01-Introduction-to-CNNs/01-why-cnns-for-images.md) |
| 2 | Limitations of FC Networks for Images | [02-limitations-fc-networks.md](01-Introduction-to-CNNs/02-limitations-fc-networks.md) |
| 3 | Local Connectivity | [03-local-connectivity.md](01-Introduction-to-CNNs/03-local-connectivity.md) |
| 4 | Parameter Sharing | [04-parameter-sharing.md](01-Introduction-to-CNNs/04-parameter-sharing.md) |
| 5 | Translation Invariance | [05-translation-invariance.md](01-Introduction-to-CNNs/05-translation-invariance.md) |

### Unit 2: Convolution Operation
| # | Topic | File |
|---|-------|------|
| 1 | 2D Convolution Intuition | [01-2d-convolution-intuition.md](02-Convolution-Operation/01-2d-convolution-intuition.md) |
| 2 | Kernel/Filter Concept | [02-kernel-filter-concept.md](02-Convolution-Operation/02-kernel-filter-concept.md) |
| 3 | Stride and Padding | [03-stride-and-padding.md](02-Convolution-Operation/03-stride-and-padding.md) |
| 4 | Feature Maps | [04-feature-maps.md](02-Convolution-Operation/04-feature-maps.md) |
| 5 | Convolution Arithmetic | [05-convolution-arithmetic.md](02-Convolution-Operation/05-convolution-arithmetic.md) |
| 6 | Multiple Input Channels | [06-multiple-input-channels.md](02-Convolution-Operation/06-multiple-input-channels.md) |
| 7 | Multiple Output Channels | [07-multiple-output-channels.md](02-Convolution-Operation/07-multiple-output-channels.md) |

### Unit 3: Pooling Layers
| # | Topic | File |
|---|-------|------|
| 1 | Max Pooling | [01-max-pooling.md](03-Pooling-Layers/01-max-pooling.md) |
| 2 | Average Pooling | [02-average-pooling.md](03-Pooling-Layers/02-average-pooling.md) |
| 3 | Global Pooling | [03-global-pooling.md](03-Pooling-Layers/03-global-pooling.md) |
| 4 | Pooling vs Strided Convolution | [04-pooling-vs-strided-conv.md](03-Pooling-Layers/04-pooling-vs-strided-conv.md) |
| 5 | Spatial Dimension Reduction | [05-spatial-dimension-reduction.md](03-Pooling-Layers/05-spatial-dimension-reduction.md) |

### Unit 4: CNN Architecture Components
| # | Topic | File |
|---|-------|------|
| 1 | Convolutional Layer | [01-convolutional-layer.md](04-CNN-Architecture-Components/01-convolutional-layer.md) |
| 2 | Activation Layer | [02-activation-layer.md](04-CNN-Architecture-Components/02-activation-layer.md) |
| 3 | Pooling Layer | [03-pooling-layer.md](04-CNN-Architecture-Components/03-pooling-layer.md) |
| 4 | Fully Connected Layer | [04-fully-connected-layer.md](04-CNN-Architecture-Components/04-fully-connected-layer.md) |
| 5 | Flatten Operation | [05-flatten-operation.md](04-CNN-Architecture-Components/05-flatten-operation.md) |
| 6 | Batch Normalization in CNNs | [06-batch-normalization-cnns.md](04-CNN-Architecture-Components/06-batch-normalization-cnns.md) |

### Unit 5: Classic CNN Architectures
| # | Topic | File |
|---|-------|------|
| 1 | LeNet-5 | [01-lenet-5.md](05-Classic-CNN-Architectures/01-lenet-5.md) |
| 2 | AlexNet | [02-alexnet.md](05-Classic-CNN-Architectures/02-alexnet.md) |
| 3 | VGGNet | [03-vggnet.md](05-Classic-CNN-Architectures/03-vggnet.md) |
| 4 | GoogLeNet/Inception | [04-googlenet-inception.md](05-Classic-CNN-Architectures/04-googlenet-inception.md) |
| 5 | ResNet | [05-resnet.md](05-Classic-CNN-Architectures/05-resnet.md) |
| 6 | DenseNet | [06-densenet.md](05-Classic-CNN-Architectures/06-densenet.md) |
| 7 | Architecture Evolution | [07-architecture-evolution.md](05-Classic-CNN-Architectures/07-architecture-evolution.md) |

### Unit 6: Modern CNN Architectures
| # | Topic | File |
|---|-------|------|
| 1 | EfficientNet | [01-efficientnet.md](06-Modern-CNN-Architectures/01-efficientnet.md) |
| 2 | MobileNet | [02-mobilenet.md](06-Modern-CNN-Architectures/02-mobilenet.md) |
| 3 | ShuffleNet | [03-shufflenet.md](06-Modern-CNN-Architectures/03-shufflenet.md) |
| 4 | NASNet | [04-nasnet.md](06-Modern-CNN-Architectures/04-nasnet.md) |
| 5 | Vision Transformer (ViT) Introduction | [05-vision-transformer-vit.md](06-Modern-CNN-Architectures/05-vision-transformer-vit.md) |
| 6 | Architecture Search | [06-architecture-search.md](06-Modern-CNN-Architectures/06-architecture-search.md) |

### Unit 7: Training CNNs
| # | Topic | File |
|---|-------|------|
| 1 | Data Augmentation for Images | [01-data-augmentation-images.md](07-Training-CNNs/01-data-augmentation-images.md) |
| 2 | Transfer Learning | [02-transfer-learning.md](07-Training-CNNs/02-transfer-learning.md) |
| 3 | Fine-tuning Strategies | [03-fine-tuning-strategies.md](07-Training-CNNs/03-fine-tuning-strategies.md) |
| 4 | Learning Rate Policies | [04-learning-rate-policies.md](07-Training-CNNs/04-learning-rate-policies.md) |
| 5 | Mixed Precision Training | [05-mixed-precision-training.md](07-Training-CNNs/05-mixed-precision-training.md) |

### Unit 8: CNN Applications
| # | Topic | File |
|---|-------|------|
| 1 | Image Classification | [01-image-classification.md](08-CNN-Applications/01-image-classification.md) |
| 2 | Object Detection | [02-object-detection.md](08-CNN-Applications/02-object-detection.md) |
| 3 | Semantic Segmentation | [03-semantic-segmentation.md](08-CNN-Applications/03-semantic-segmentation.md) |
| 4 | Instance Segmentation | [04-instance-segmentation.md](08-CNN-Applications/04-instance-segmentation.md) |
| 5 | Face Recognition | [05-face-recognition.md](08-CNN-Applications/05-face-recognition.md) |
| 6 | Medical Imaging | [06-medical-imaging.md](08-CNN-Applications/06-medical-imaging.md) |

### Unit 9: Visualizing CNNs
| # | Topic | File |
|---|-------|------|
| 1 | Filter Visualization | [01-filter-visualization.md](09-Visualizing-CNNs/01-filter-visualization.md) |
| 2 | Feature Map Visualization | [02-feature-map-visualization.md](09-Visualizing-CNNs/02-feature-map-visualization.md) |
| 3 | Grad-CAM | [03-grad-cam.md](09-Visualizing-CNNs/03-grad-cam.md) |
| 4 | Saliency Maps | [04-saliency-maps.md](09-Visualizing-CNNs/04-saliency-maps.md) |
| 5 | Activation Maximization | [05-activation-maximization.md](09-Visualizing-CNNs/05-activation-maximization.md) |
| 6 | Understanding What CNNs Learn | [06-understanding-cnns-learn.md](09-Visualizing-CNNs/06-understanding-cnns-learn.md) |

### Unit 10: Advanced Topics
| # | Topic | File |
|---|-------|------|
| 1 | Depthwise Separable Convolutions | [01-depthwise-separable-conv.md](10-Advanced-Topics/01-depthwise-separable-conv.md) |
| 2 | Dilated/Atrous Convolutions | [02-dilated-atrous-conv.md](10-Advanced-Topics/02-dilated-atrous-conv.md) |
| 3 | Deformable Convolutions | [03-deformable-conv.md](10-Advanced-Topics/03-deformable-conv.md) |
| 4 | Attention in CNNs | [04-attention-in-cnns.md](10-Advanced-Topics/04-attention-in-cnns.md) |
| 5 | Neural Style Transfer | [05-neural-style-transfer.md](10-Advanced-Topics/05-neural-style-transfer.md) |

---

## 🎯 Learning Objectives

After completing this module, you will be able to:
- Understand convolution, pooling, and CNN building blocks
- Explain and implement classic and modern CNN architectures
- Train CNNs with transfer learning and data augmentation
- Apply CNNs to classification, detection, and segmentation tasks
- Visualize and interpret what CNNs learn

---

## 🏗️ CNN Architecture (ASCII)

```
┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐
│  Input  │──►│  Conv   │──►│  Pool   │──►│  Conv   │──►│  Pool   │
│  Image  │   │ + ReLU  │   │         │   │ + ReLU  │   │         │
└─────────┘   └─────────┘   └─────────┘   └─────────┘   └─────────┘
                                                              │
                                                              ▼
                                         ┌─────────┐   ┌─────────┐
                                         │ Softmax │◄──│  Fully  │
                                         │ Output  │   │Connected│
                                         └─────────┘   └─────────┘
```

---

[← Previous: Neural Networks](../3.1-Neural-Networks-Fundamentals/README.md) | [🏠 Back to Main Course](../../README.md) | [Next: RNN →](../3.3-RNN/README.md)
