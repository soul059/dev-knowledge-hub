# Applications of Computer Vision

## Overview

Computer vision is one of the most widely deployed AI technologies, powering everything from smartphone cameras to autonomous vehicles. This chapter surveys the major application domains with concrete examples.

---

## Application Domains

```
┌─────────────────────────────────────────────────────┐
│              CV Applications by Domain               │
├──────────────┬──────────────────────────────────────┤
│ Autonomous   │ Self-driving cars, drones, robots     │
│ Vehicles     │ Lane detection, obstacle avoidance    │
├──────────────┼──────────────────────────────────────┤
│ Healthcare   │ X-ray analysis, tumor detection,      │
│              │ retinal screening, surgical guidance   │
├──────────────┼──────────────────────────────────────┤
│ Security     │ Surveillance, face recognition,       │
│              │ anomaly detection, license plates      │
├──────────────┼──────────────────────────────────────┤
│ Retail       │ Cashier-less stores, product search,  │
│              │ virtual try-on, inventory tracking     │
├──────────────┼──────────────────────────────────────┤
│ Manufacturing│ Defect detection, quality control,    │
│              │ robotic assembly, safety monitoring    │
├──────────────┼──────────────────────────────────────┤
│ Agriculture  │ Crop disease detection, yield         │
│              │ estimation, weed identification        │
├──────────────┼──────────────────────────────────────┤
│ Entertainment│ AR/VR, face filters, motion capture,  │
│              │ image generation (DALL-E, Midjourney)  │
└──────────────┴──────────────────────────────────────┘
```

---

## Autonomous Driving Pipeline

```
Camera/LiDAR Input
       │
       ├── [Object Detection]     → cars, pedestrians, cyclists
       ├── [Lane Detection]       → lane markings, road boundaries
       ├── [Semantic Segmentation]→ road, sidewalk, sky, buildings
       ├── [Depth Estimation]     → distance to obstacles
       ├── [Sign Recognition]     → speed limits, stop signs
       └── [Tracking]             → follow objects across frames
              │
              ▼
       [Planning & Control] → steer, brake, accelerate
```

---

## Medical Imaging

```
Application examples:

  Chest X-ray:
    Input: X-ray image → [CNN] → "Pneumonia detected" (95% confidence)
    Models: CheXNet (DenseNet-121), COVID-Net

  Retinal screening:
    Input: Fundus photo → [Segmentation] → vessel map + lesion detection
    Detects: Diabetic retinopathy, glaucoma, macular degeneration

  Histopathology:
    Input: Tissue slide (gigapixel) → [Patch classification] → Cancer/Normal
    Challenge: Images are 100,000 × 100,000 pixels!

  MRI/CT:
    Input: 3D volume → [3D CNN] → Tumor segmentation + volume estimation
```

---

## Quick Python Example

```python
import cv2

# Face detection using OpenCV (one of the most common CV tasks)
face_cascade = cv2.CascadeClassifier(
    cv2.data.haarcascades + "haarcascade_frontalface_default.xml"
)

img = cv2.imread("photo.jpg")
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

faces = face_cascade.detectMultiScale(gray, scaleFactor=1.1, minNeighbors=5)
print(f"Found {len(faces)} faces")

for (x, y, w, h) in faces:
    cv2.rectangle(img, (x, y), (x+w, y+h), (0, 255, 0), 2)

cv2.imwrite("detected_faces.jpg", img)
```

---

## CV Task → Application Mapping

| CV Task | Application Examples |
|---------|---------------------|
| Classification | Medical diagnosis, content moderation, species ID |
| Object Detection | Autonomous driving, surveillance, retail analytics |
| Segmentation | Medical imaging, satellite analysis, photo editing |
| Face Recognition | Phone unlock, security, attendance systems |
| OCR | Document scanning, license plate reading, receipts |
| Pose Estimation | Sports analytics, fitness apps, animation |
| Image Generation | Art creation, data augmentation, design tools |

---

## Revision Questions

1. **Name three CV applications in healthcare.**
2. **What CV tasks are needed for autonomous driving?**
3. **How is computer vision used in manufacturing quality control?**
4. **What is the difference between CV tasks needed for security vs retail?**
5. **Why are medical imaging datasets particularly challenging?**

---

[Previous: 01-what-is-cv.md](01-what-is-cv.md) | [Next: 03-image-fundamentals.md](03-image-fundamentals.md)
