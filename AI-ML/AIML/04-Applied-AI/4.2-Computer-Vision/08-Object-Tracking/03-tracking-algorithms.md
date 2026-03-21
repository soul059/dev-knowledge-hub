# Tracking Algorithms: KCF, SORT, DeepSORT

## Overview

This chapter provides a deeper implementation-level view of three key tracking algorithms: **KCF** (correlation filter for SOT), **SORT** (Kalman + IoU for MOT), and **DeepSORT** (SORT + appearance). Understanding their internals is essential for choosing and customizing trackers.

---

## KCF (Kernelized Correlation Filters)

```
Henriques et al. (2015)

Core idea: Learn a ridge regression filter in frequency domain
           using circulant matrix structure for O(n log n) speed

Training:
  Given target patch x (centered on object):
  
  1. Generate circular shifts of x (all translations)
     → Circulant matrix C(x)
  
  2. Assign Gaussian-shaped labels y:
     Center shift → high response
     Far shifts → low response
     
     y = [0.01, 0.1, 0.5, 1.0, 0.5, 0.1, 0.01, ...]
     
  3. Solve ridge regression in frequency domain:
     α̂ = ŷ / (k̂_xx + λ)
     
     k̂_xx = DFT of kernel correlation
     λ = regularization
     
     All operations are element-wise in frequency domain!
     → No matrix inversion needed

Detection:
  For search region z:
  
  1. Compute kernel correlation between z and stored template x
  2. Response map: f̂(z) = k̂_xz ⊙ α̂
  3. Peak of response map = target location
  
  ┌──────────────────┐     ┌──────────────────┐
  │  Search region z │ →   │ Response map f(z) │
  │    ┌────┐        │     │       ★           │  ★ = peak
  │    │obj │        │     │                   │
  │    └────┘        │     │                   │
  └──────────────────┘     └──────────────────┘

Online update:
  x_model ← (1-η) × x_model + η × x_new
  α_model ← (1-η) × α_model + η × α_new
  η = learning rate (typically 0.02)

Features used:
  Raw pixels: fast, basic
  HOG features: much better accuracy
  CN (Color Names): adds color information
  Deep features: best accuracy, slower
```

---

## SORT: Implementation Details

```
Kalman Filter Model:

  State vector: x = [u, v, s, r, u̇, v̇, ṡ]^T
    u, v = bounding box center
    s = area (width × height)
    r = aspect ratio (width / height) — assumed constant
    u̇, v̇, ṡ = velocities

  Transition matrix F (constant velocity model):
  ┌                     ┐
  │ 1 0 0 0 1 0 0 │
  │ 0 1 0 0 0 1 0 │
  │ 0 0 1 0 0 0 1 │   x_{t+1} = F × x_t
  │ 0 0 0 1 0 0 0 │
  │ 0 0 0 0 1 0 0 │
  │ 0 0 0 0 0 1 0 │
  │ 0 0 0 0 0 0 1 │
  └                     ┘

  Observation matrix H:
  ┌             ┐
  │ 1 0 0 0 0 0 0 │    Observe: [u, v, s, r]
  │ 0 1 0 0 0 0 0 │
  │ 0 0 1 0 0 0 0 │    z = H × x
  │ 0 0 0 1 0 0 0 │
  └             ┘

Association — IoU cost matrix:

  For N tracks and M detections:
  
  Cost[i,j] = 1 - IoU(track_i_predicted, detection_j)
  
  Apply threshold: if IoU < 0.3, set cost = ∞ (no match)
  
  Solve with Hungarian algorithm → O(max(N,M)³)

Track lifecycle:
  Birth: Unmatched detection → tentative track (hits=1)
         After min_hits=3 consecutive matches → confirmed
  
  Death: Track without match → age++
         If age > max_age=1: delete track
         (very aggressive — lost for 1 frame = deleted)
```

---

## DeepSORT: Implementation Details

```
Deep Appearance Descriptor:

  Network: Wide ResNet variant trained on person re-ID dataset
  Input: 128×64 cropped detection
  Output: 128-D L2-normalized feature vector
  
  ┌──────────┐     ┌──────┐     ┌──────────┐
  │ Detection│ → │ CNN  │ → │ 128-D     │
  │ crop     │     │      │     │ embedding │
  │ 128×64   │     │      │     │ ‖f‖=1    │
  └──────────┘     └──────┘     └──────────┘

Feature gallery per track:
  Store last L=100 appearance features
  Gallery_k = {f_k^1, f_k^2, ..., f_k^L}

Appearance cost:
  d_app(i, j) = min_{f ∈ Gallery_i} (1 - f^T × f_j)
  
  Take minimum cosine distance over gallery
  → Robust to brief appearance changes

Combined cost:
  d(i,j) = λ × d_motion(i,j) + (1-λ) × d_app(i,j)
  
  d_motion = Mahalanobis distance from Kalman prediction
  λ = 0 in practice (appearance-only matching works best!)

Matching cascade (prioritize recent tracks):
  for age = 0, 1, 2, ..., max_age:
    tracks_at_age = {tracks with this age}
    Hungarian match between tracks_at_age and remaining dets
    Remove matched detections from pool
  
  → Recently seen tracks get first pick of detections
  → Prevents old tracks from stealing matches

Final IoU matching:
  Unmatched tracks + unmatched detections → IoU-based matching
  → Catches associations missed by appearance (partial views)
```

---

## Comparison

| Feature | KCF | SORT | DeepSORT |
|---------|:---:|:---:|:---:|
| Task | SOT | MOT | MOT |
| Core method | Correlation filter | Kalman + IoU | Kalman + Re-ID |
| Speed | 300 FPS | 260 FPS* | 40 FPS* |
| Appearance | HOG features | None | Deep CNN |
| # Targets | 1 | Many | Many |
| ID switches | N/A | High | Low |
| Handles occlusion | Poorly | Poorly | Moderate |
| Online learning | Yes (template update) | Yes (Kalman) | Yes (gallery) |

*Tracking only, excludes detection time

---

## Python: DeepSORT Implementation

```python
from deep_sort_realtime.deepsort_tracker import DeepSort

tracker = DeepSort(
    max_age=30,           # Frames before track deletion
    n_init=3,             # Hits before track confirmation
    max_cosine_distance=0.3,  # Appearance threshold
    nn_budget=100         # Gallery size per track
)

# Each frame: get detections from YOLO or similar
# detections: list of ([x,y,w,h], confidence, class)
detections = [
    ([100, 100, 50, 80], 0.95, "person"),
    ([300, 150, 60, 90], 0.87, "person"),
]

# Update tracker
tracks = tracker.update_tracks(detections, frame=frame)

for track in tracks:
    if not track.is_confirmed():
        continue
    
    track_id = track.track_id
    bbox = track.to_ltrb()  # [x1, y1, x2, y2]
    print(f"Track {track_id}: {bbox}")
```

---

## Revision Questions

1. **How does KCF achieve real-time speeds using the frequency domain?**
2. **What is the Kalman Filter state vector in SORT and what does each component represent?**
3. **How does the Hungarian algorithm solve the assignment problem in tracking?**
4. **What is the matching cascade in DeepSORT and why is it needed?**
5. **Why does DeepSORT set λ=0 (appearance-only matching)?**

---

[Previous: 02-multi-object-tracking.md](02-multi-object-tracking.md) | [Next: 04-re-identification.md](04-re-identification.md)
