# Face Recognition — Detection, Embedding & Matching

> **Unit 8 · CNN Applications · Topic 5/6**

---

## Table of Contents

1. [Overview & Pipeline](#1-overview--pipeline)
2. [Face Detection & Alignment](#2-face-detection--alignment)
3. [Verification vs Identification](#3-verification-vs-identification)
4. [Siamese Networks](#4-siamese-networks)
5. [Triplet Loss & FaceNet](#5-triplet-loss--facenet)
6. [ArcFace & Angular Margin Losses](#6-arcface--angular-margin-losses)
7. [Worked Examples](#7-worked-examples)
8. [PyTorch Implementation](#8-pytorch-implementation)
9. [Summary Table](#9-summary-table)
10. [Revision Questions](#10-revision-questions)
11. [Navigation](#11-navigation)

---

## 1. Overview & Pipeline

```
Face Recognition Pipeline:

  ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
  │  Input   │     │  Face    │     │  Face    │     │  Feature │
  │  Image   │────►│ Detection│────►│Alignment │────►│Extraction│
  └──────────┘     └──────────┘     └──────────┘     └────┬─────┘
                                                          │
                   ┌──────────────────────────────────────┘
                   │  128-512 dim embedding vector
                   ▼
              ┌──────────┐
              │ Matching  │
              │ (cosine   │────► Identity / Verified
              │ similarity│
              │ or L2)    │
              └──────────┘

Each face → fixed-size embedding vector in "face space"
  Same person → embeddings CLOSE together
  Different person → embeddings FAR apart
```

---

## 2. Face Detection & Alignment

```
Step 1: DETECTION — Find all faces in image
  Methods:
  • MTCNN (Multi-Task Cascaded CNN) — classic, reliable
  • RetinaFace — state-of-the-art, detects landmarks
  • BlazeFace — lightweight, for mobile

Step 2: LANDMARK DETECTION — Find key facial points
  5-point:  2 eyes, nose, 2 mouth corners
  68-point: full face outline (for precise alignment)
  
  ┌────────────────┐
  │    o       o   │  ← eye centers
  │                │
  │       o        │  ← nose tip
  │                │
  │    o       o   │  ← mouth corners
  └────────────────┘

Step 3: ALIGNMENT — Normalize face geometry
  Use landmarks to compute affine transform:
  • Center face
  • Align eyes horizontally
  • Scale to standard size (e.g., 112×112)
  
  ┌──────┐    Align    ┌──────┐
  │ ╱╲   │   ─────►   │      │
  │ ° °  │            │ ° °  │  Standardized
  │  △   │            │  △   │  112×112
  │ ─── │            │ ───  │
  └──────┘            └──────┘
  Tilted                Aligned
```

---

## 3. Verification vs Identification

```
VERIFICATION (1:1 matching):
  "Is this person who they claim to be?"
  
  Input:  Face image + claimed identity
  Output: Yes/No (match or no match)
  
  Method: Compare face embedding with stored template
  Decision: similarity > threshold → match
  
  Application: Phone unlock, passport control, secure login

IDENTIFICATION (1:N matching):
  "Who is this person?"
  
  Input:  Face image + gallery of N known faces
  Output: Identity (or "unknown")
  
  Method: Compare face embedding against all N gallery embeddings
  Decision: find closest match, check if distance < threshold
  
  Application: Surveillance, photo tagging, access control

  ┌──────────────────────────────────────────────┐
  │ Verification:  O(1) comparison               │
  │ Identification: O(N) comparisons             │
  │                 (or O(log N) with indexing)   │
  └──────────────────────────────────────────────┘
```

---

## 4. Siamese Networks

```
Siamese Network: Twin networks with SHARED WEIGHTS

  Image A ──► CNN_θ ──► Embedding A (128-d)
                                              ├── Distance(A,B) → Same/Different?
  Image B ──► CNN_θ ──► Embedding B (128-d)
  
  Same θ (shared weights)!

  Training with Contrastive Loss:
    L = (1-y) · ½ · D² + y · ½ · max(0, m - D)²

    y = 0: same person    → minimize D (pull together)
    y = 1: different person → maximize D (push apart, up to margin m)
    D = ||f(xᵢ) - f(xⱼ)||₂

  ┌─────────────────────────────────────────┐
  │  Embedding Space:                       │
  │                                         │
  │    ●● ←── Same person (close)          │
  │    ●●                                  │
  │                     ○                  │
  │         ▲▲ ←── Same person             │
  │         ▲▲                             │
  │              ○ ←── Different (far)     │
  │                                         │
  │    ●,▲ = different identities          │
  │    ○ = negative examples               │
  └─────────────────────────────────────────┘
```

---

## 5. Triplet Loss & FaceNet

### Triplet Loss

```
Triplet: (Anchor, Positive, Negative)
  Anchor:   reference face
  Positive: same person, different photo
  Negative: different person

Goal: D(anchor, positive) + margin < D(anchor, negative)

L = max(0, ||f(a) - f(p)||² - ||f(a) - f(n)||² + α)

  Where α = margin (typically 0.2-0.5)

  Before training:          After training:
  ┌────────────────┐       ┌────────────────┐
  │  n             │       │        n       │
  │    a           │       │                │
  │      p         │       │  a·p           │
  │                │       │                │
  │ (no structure) │       │ (a,p close;    │
  │                │       │  n far away)   │
  └────────────────┘       └────────────────┘
```

### FaceNet Architecture

```
FaceNet (Schroff et al., 2015):

  Image (160×160×3)
      │
      ▼
  Inception/ResNet Backbone
      │
      ▼
  Global Average Pooling
      │
      ▼
  FC → 128-dim
      │
      ▼
  L2 Normalize → unit sphere embedding
      │
      ▼
  128-dim embedding (||f(x)||₂ = 1)

Key design choices:
  1. L2 normalization → embeddings lie on unit hypersphere
  2. Euclidean distance on unit sphere ∝ angular distance
  3. Semi-hard triplet mining (most informative triplets)

Semi-hard negative mining:
  Select negatives where:
  D(a,p) < D(a,n) < D(a,p) + α
  
  Easy negatives (D(a,n) >> D(a,p)): too easy, no learning signal
  Hard negatives (D(a,n) < D(a,p)): can collapse training
  Semi-hard:     just right for stable learning
```

---

## 6. ArcFace & Angular Margin Losses

### Evolution of Loss Functions

```
Softmax Loss → SphereFace → CosFace → ArcFace

Standard Softmax:
  L = -log(exp(W_y^T f) / Σⱼ exp(W_j^T f))
  
  Problem: Learned features are NOT discriminative enough for faces
  (softmax maximizes inter-class separation but doesn't enforce tight clusters)

Angular Margin Losses — add margin penalty in angular space:

SphereFace (A-Softmax):
  L = -log(exp(||f||cos(mθ_y)) / ...)     multiply angle by m
  
CosFace (LMCL):
  L = -log(exp(s(cos θ_y - m)) / ...)     subtract m from cosine

ArcFace (Additive Angular Margin):
  L = -log(exp(s·cos(θ_y + m)) / ...)     add m to angle
```

### ArcFace Detail

```
ArcFace Loss:

Standard classification:
  logit_j = W_j^T · f = ||W_j|| · ||f|| · cos(θ_j)

With L2-normalized weights and features:
  logit_j = s · cos(θ_j)     (s = scale factor, e.g., 64)

ArcFace modifies the target class logit:
  logit_y = s · cos(θ_y + m)    ← add angular margin m

  L = -log(exp(s·cos(θ_y + m)) / (exp(s·cos(θ_y + m)) + Σⱼ≠y exp(s·cos(θ_j))))

  Where:
    θ_j = angle between feature f and weight W_j
    m = angular margin (typically 0.5 radians)
    s = scale factor (typically 64)

Geometric interpretation:
  Without margin:  Decision boundary at θ = π/2 (90°)
  With ArcFace:    Decision boundary at θ = π/2 - m
  
  → Features must be CLOSER to their class center
  → Tighter intra-class clusters, wider inter-class gaps

  ┌─────────────────────────────────┐
  │  Angular space:                 │
  │                                 │
  │     Class A    margin    Class B│
  │    ●●●●●●   |← m →|  ▲▲▲▲▲▲ │
  │    ●●●●●●            ▲▲▲▲▲▲  │
  │                                 │
  │  Without margin:                │
  │    ●●●●●●●●●●▲▲▲▲▲▲▲▲▲▲     │
  │    (overlapping!)              │
  └─────────────────────────────────┘
```

---

## 7. Worked Examples

### Example: Verification Decision

```
Given two face embeddings (L2-normalized, 4-dim for simplicity):
  f_a = [0.5, 0.5, 0.5, 0.5]    (Person A)
  f_b = [0.48, 0.52, 0.49, 0.51] (Claimed to be A)
  
Cosine similarity:
  cos(f_a, f_b) = (0.5×0.48 + 0.5×0.52 + 0.5×0.49 + 0.5×0.51) / (1.0 × 1.0)
                = (0.24 + 0.26 + 0.245 + 0.255) / 1
                = 1.0 / 1.0 = 1.0

Wait — L2 norms:
  ||f_a|| = √(4×0.25) = 1.0
  ||f_b|| = √(0.2304+0.2704+0.2401+0.2601) = √1.001 ≈ 1.0005
  
  cos_sim = 1.0 / (1.0 × 1.0005) ≈ 0.9995

L2 distance:
  d = √((0.02)² + (-0.02)² + (0.01)² + (-0.01)²) = √0.001 ≈ 0.0316

Threshold = 0.4 (typical for L2 on unit sphere):
  d = 0.0316 < 0.4 → VERIFIED ✓ (same person)

If f_c = [−0.3, 0.7, −0.5, 0.4] (Different person):
  d(a,c) = √(0.64+0.04+1.0+0.01) = √1.69 = 1.3
  d = 1.3 > 0.4 → NOT VERIFIED ✗ (different person)
```

---

## 8. PyTorch Implementation

### ArcFace Loss

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import math

class ArcFaceLoss(nn.Module):
    def __init__(self, in_features, num_classes, s=64.0, m=0.5):
        super().__init__()
        self.s = s
        self.m = m
        self.W = nn.Parameter(torch.FloatTensor(num_classes, in_features))
        nn.init.xavier_uniform_(self.W)

    def forward(self, features, labels):
        # L2 normalize features and weights
        features = F.normalize(features, dim=1)
        W = F.normalize(self.W, dim=1)

        # Cosine similarity
        cos_theta = F.linear(features, W)  # (B, num_classes)
        cos_theta = cos_theta.clamp(-1, 1)

        # ArcFace: add angular margin to target class
        theta = torch.acos(cos_theta)
        target_logits = torch.cos(theta[range(len(labels)), labels] + self.m)

        # Replace target class logits
        logits = cos_theta.clone()
        logits[range(len(labels)), labels] = target_logits

        # Scale
        logits *= self.s

        # CrossEntropy
        return F.cross_entropy(logits, labels)


# Usage
backbone = torch.hub.load('pytorch/vision', 'resnet50', pretrained=True)
backbone.fc = nn.Linear(2048, 512)  # 512-dim embedding

arc_loss = ArcFaceLoss(in_features=512, num_classes=10000, s=64, m=0.5)

# Training step
features = backbone(images)         # (B, 512)
features = F.normalize(features)    # L2 normalize
loss = arc_loss(features, labels)
loss.backward()
```

### Triplet Loss

```python
class TripletLoss(nn.Module):
    def __init__(self, margin=0.3):
        super().__init__()
        self.margin = margin

    def forward(self, anchor, positive, negative):
        d_pos = (anchor - positive).pow(2).sum(1)  # ||a-p||²
        d_neg = (anchor - negative).pow(2).sum(1)  # ||a-n||²
        loss = F.relu(d_pos - d_neg + self.margin)
        return loss.mean()
```

### Face Verification Pipeline

```python
import torch
import torch.nn.functional as F

class FaceVerifier:
    def __init__(self, model, threshold=0.4):
        self.model = model
        self.threshold = threshold
        self.model.eval()

    @torch.no_grad()
    def get_embedding(self, face_image):
        """Extract L2-normalized embedding."""
        embedding = self.model(face_image.unsqueeze(0))
        return F.normalize(embedding, dim=1).squeeze(0)

    def verify(self, face1, face2):
        """Verify if two faces belong to the same person."""
        emb1 = self.get_embedding(face1)
        emb2 = self.get_embedding(face2)

        # Cosine similarity
        cos_sim = F.cosine_similarity(emb1.unsqueeze(0),
                                       emb2.unsqueeze(0)).item()
        # L2 distance
        l2_dist = (emb1 - emb2).norm().item()

        return {
            'is_same': l2_dist < self.threshold,
            'cosine_similarity': cos_sim,
            'l2_distance': l2_dist,
        }

    def identify(self, face, gallery_embeddings, gallery_labels):
        """Identify a face against a gallery."""
        emb = self.get_embedding(face)
        distances = torch.cdist(emb.unsqueeze(0),
                                 gallery_embeddings).squeeze(0)
        min_dist, min_idx = distances.min(0)

        if min_dist.item() < self.threshold:
            return gallery_labels[min_idx.item()], min_dist.item()
        return "unknown", min_dist.item()
```

---

## 9. Summary Table

| Concept | Key Idea |
|---|---|
| **Pipeline** | Detect → align → extract embedding → match |
| **Verification** | 1:1 — is this the claimed person? |
| **Identification** | 1:N — who is this person in gallery? |
| **Siamese network** | Twin CNNs with shared weights, contrastive loss |
| **Triplet loss** | max(0, d(a,p) - d(a,n) + margin) |
| **FaceNet** | Inception + triplet loss → 128-d L2-normalized embedding |
| **ArcFace** | Add angular margin m to target class angle; s·cos(θ+m) |
| **Semi-hard mining** | Select negatives with d(a,p) < d(a,n) < d(a,p)+α |
| **L2 normalization** | Embeddings on unit hypersphere; cosine sim = dot product |

---

## 10. Revision Questions

1. **Describe the face recognition pipeline** from input image to identity output. What role does alignment play?

2. **Compare verification and identification.** What is the computational complexity of each? How does the threshold affect FAR/FRR?

3. **Explain triplet loss.** What are anchor, positive, and negative? Why is semi-hard mining important? What happens with easy vs hard negatives?

4. **Derive the ArcFace loss.** Starting from standard softmax, show each modification. Why does adding angular margin improve face recognition?

5. **Given 3 face embeddings** a=[0.6,0.8], p=[0.65,0.76], n=[-0.5,0.87], compute triplet loss with margin=0.3. Should the model still learn from this triplet?

---

## 11. Navigation

| Previous | Up | Next |
|---|---|---|
| [← Instance Segmentation](./04-instance-segmentation.md) | [Unit 8 Index](../08-CNN-Applications/) | [Medical Imaging →](./06-medical-imaging.md) |

---

> **References:**  
> - Schroff, F. et al. (2015). *FaceNet: A Unified Embedding for Face Recognition and Clustering.* CVPR.  
> - Deng, J. et al. (2019). *ArcFace: Additive Angular Margin Loss for Deep Face Recognition.* CVPR.  
> - Taigman, Y. et al. (2014). *DeepFace: Closing the Gap to Human-Level Performance in Face Verification.* CVPR.
