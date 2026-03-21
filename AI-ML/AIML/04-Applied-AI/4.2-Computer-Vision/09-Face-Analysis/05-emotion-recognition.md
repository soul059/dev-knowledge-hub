# Emotion Recognition

## Overview

**Facial emotion recognition (FER)** classifies facial expressions into discrete emotion categories or continuous affect dimensions. It maps facial images to emotional states — enabling applications in human-computer interaction, mental health monitoring, driver safety, and customer experience analysis.

---

## Emotion Models

```
Ekman's 6 Basic Emotions (1971):
  The most widely used categorical model

  😃 Happy      — raised cheeks, lip corners up
  😢 Sad        — inner brow raise, lip corners down
  😠 Angry      — brows lowered, lips pressed
  😨 Fear       — brows raised, eyes wide, mouth open
  😲 Surprise   — brows raised high, jaw drop
  🤢 Disgust    — nose wrinkle, upper lip raise
  😐 Neutral    — baseline, no expression

  + Sometimes extended:
    😤 Contempt  — unilateral lip corner raise

Valence-Arousal Model (Russell, 1980):
  Continuous 2D representation

  Arousal (high)
       ↑
  Fear ● ● Excitement
       │
  Sad  ● ─────────────── ● Happy     → Valence
       │                              (negative to positive)
  Bored● ● Calm
       ↓
  Arousal (low)

  Valence: negative ←→ positive (how pleasant)
  Arousal: low ←→ high (how activated/intense)
  
  Advantage: Captures subtle emotions, blends, intensities
```

---

## FACS (Facial Action Coding System)

```
Ekman & Friesen (1978) — anatomically based

  Defines 44 Action Units (AUs) based on facial muscle movements:

  AU1:  Inner Brow Raise     AU12: Lip Corner Puller (smile)
  AU2:  Outer Brow Raise     AU15: Lip Corner Depressor
  AU4:  Brow Lowerer         AU17: Chin Raiser
  AU5:  Upper Lid Raise      AU20: Lip Stretcher
  AU6:  Cheek Raise          AU23: Lip Tightener
  AU7:  Lid Tightener        AU25: Lips Part
  AU9:  Nose Wrinkle         AU26: Jaw Drop

  Emotions as AU combinations:
  
  Happy:    AU6 + AU12          (cheek raise + smile)
  Sad:      AU1 + AU4 + AU15   (inner brow + brow lower + frown)
  Surprise: AU1 + AU2 + AU5 + AU26  (brows up + eyes wide + jaw)
  Angry:    AU4 + AU5 + AU7 + AU23  (brows down + tight lips)
  
  AU detection → more fine-grained than emotion classification
  Can detect fake smiles (real: AU6+AU12, fake: AU12 only)
```

---

## Deep Learning for FER

```
Architecture:

  Face image (224×224)
       │
  ┌────▼────────────────┐
  │ Backbone CNN         │  ResNet-50 / VGGFace / EfficientNet
  │ (pretrained on faces)│
  └────┬────────────────┘
       │
  ┌────▼────┐
  │ Feature │  Global Average Pooling
  │ Pool    │  → 2048-D
  └────┬────┘
       │
  ┌────▼────┐
  │ FC + BN │  → 512 → 7 (or 8)
  │ Softmax │
  └────┬────┘
       │
  Emotion class: [happy, sad, angry, fear, surprise, disgust, neutral]

  Common enhancements:
    - Attention mechanisms (focus on expressive regions)
    - Multi-task: emotion + AU detection jointly
    - Temporal modeling for video (LSTM/Transformer)
    - Label smoothing (expressions are ambiguous)
```

---

## Challenges

```
1. Subjectivity & Ambiguity
   Same face, different annotators → different labels
   
   ┌──────────┐
   │   😏     │  Happy? Contempt? Neutral?
   │          │  Inter-annotator agreement: ~70-80%
   └──────────┘

2. Class Imbalance
   Most datasets dominated by Happy + Neutral
   
   Happy    ████████████████████  35%
   Neutral  █████████████████     30%
   Sad      █████                 10%
   Surprise ████                   8%
   Angry    ████                   8%
   Fear     ███                    5%
   Disgust  ██                     4%

3. Cultural Differences
   Expressions vary across cultures
   Datasets biased toward Western expressions

4. Posed vs Spontaneous
   Lab-collected (posed) ≠ real-world (spontaneous)
   Models trained on posed data fail in the wild

5. Occlusion & Pose
   Masks, sunglasses, extreme head angles
```

---

## Datasets

| Dataset | Images | Classes | Type |
|---------|--------|---------|------|
| FER2013 | 35,887 | 7 emotions | Wild, grayscale, 48×48 |
| AffectNet | 450K | 8 emotions + V/A | Wild, large-scale |
| RAF-DB | 30K | 7 basic + 12 compound | Wild, reliable labels |
| CK+ | 593 sequences | 7 emotions | Lab, posed, video |
| EmotioNet | 1M | 23 AUs | Wild, AU-annotated |
| AFEW | 1,809 clips | 7 emotions | Movie clips, video |

---

## Video-Based Emotion Recognition

```
Temporal modeling captures expression dynamics:

  Frame sequence:
  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐
  │ 😐 │→│ 😐 │→│ 😊 │→│ 😃 │→│ 😃 │
  └──┬─┘ └──┬─┘ └──┬─┘ └──┬─┘ └──┬─┘
     │      │      │      │      │
  ┌──▼──────▼──────▼──────▼──────▼──┐
  │        CNN per frame             │
  │     (shared weights)             │
  └──┬──────┬──────┬──────┬──────┬──┘
     │      │      │      │      │
  ┌──▼──────▼──────▼──────▼──────▼──┐
  │     LSTM / Transformer           │
  │     Temporal modeling            │
  └──────────────┬──────────────────┘
                 │
            Emotion: Happy (onset → apex)

  Expression phases:
    Onset → Apex → Offset → Neutral
    Temporal patterns help distinguish real vs fake expressions
```

---

## Python: Emotion Recognition

```python
# Using DeepFace library
from deepface import DeepFace

# Analyze single image
result = DeepFace.analyze(
    img_path="face.jpg",
    actions=["emotion"],
    enforce_detection=True
)

print(f"Dominant: {result[0]['dominant_emotion']}")
for emotion, score in result[0]['emotion'].items():
    print(f"  {emotion}: {score:.1f}%")


# Real-time emotion detection
import cv2

cap = cv2.VideoCapture(0)

while True:
    ret, frame = cap.read()
    if not ret:
        break
    
    try:
        results = DeepFace.analyze(
            frame,
            actions=["emotion"],
            enforce_detection=False,
            silent=True
        )
        
        for face in results:
            emotion = face["dominant_emotion"]
            region = face["region"]
            x, y, w, h = region["x"], region["y"], \
                          region["w"], region["h"]
            
            cv2.rectangle(frame, (x, y), (x+w, y+h),
                          (0, 255, 0), 2)
            cv2.putText(frame, emotion, (x, y-10),
                        cv2.FONT_HERSHEY_SIMPLEX, 0.9,
                        (0, 255, 0), 2)
    except:
        pass
    
    cv2.imshow("Emotion", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
```

---

## Revision Questions

1. **What are Ekman's 6 basic emotions and what facial features characterize each?**
2. **How does the valence-arousal model differ from categorical emotions?**
3. **What are Action Units (AUs) and how do they relate to emotions?**
4. **Why is FER particularly challenging compared to other classification tasks?**
5. **How does temporal modeling improve emotion recognition in video?**

---

[Previous: 04-verification-vs-identification.md](04-verification-vs-identification.md) | [Next: 06-face-models.md](06-face-models.md)
