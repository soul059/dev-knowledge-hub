# Model Servers (TF Serving, TorchServe, Triton)

## Overview

Model servers are purpose-built infrastructure for serving ML models at scale. Unlike custom REST APIs, model servers handle model loading, versioning, batching, GPU management, and protocol optimization out of the box. This chapter covers the three major model servers — TensorFlow Serving, TorchServe, and NVIDIA Triton Inference Server — their architecture, when to use each, and example configurations.

---

## Custom API vs Model Server

```
  ┌──────────────────────────────────────────────────────────────────┐
  │                                                                  │
  │  Custom API (FastAPI / Flask)         Model Server               │
  │  ┌──────────────────────────┐        ┌────────────────────────┐ │
  │  │ You build everything:    │        │ You provide the model: │ │
  │  │  • HTTP handler          │        │  • model.savedmodel/   │ │
  │  │  • Input parsing         │        │  • model.mar           │ │
  │  │  • Model loading         │        │  • model.onnx          │ │
  │  │  • Batching logic        │        │                        │ │
  │  │  • GPU management        │        │ Server handles:        │ │
  │  │  • Version switching     │        │  • HTTP + gRPC         │ │
  │  │  • Health checks         │        │  • Batching            │ │
  │  │  • Scaling               │        │  • GPU scheduling      │ │
  │  └──────────────────────────┘        │  • Model versioning    │ │
  │                                       │  • A/B testing         │ │
  │  ✅ Full control                      │  • Health checks       │ │
  │  ❌ More code to maintain             │  • Metrics             │ │
  │  ❌ No built-in batching              └────────────────────────┘ │
  │                                       ✅ Production-optimized    │
  │                                       ✅ GPU-aware               │
  │                                       ❌ Less flexible           │
  └──────────────────────────────────────────────────────────────────┘
```

---

## TensorFlow Serving

Optimized server for TensorFlow SavedModel format. First-class support for model versioning and gRPC.

```
  ┌──────────────────────────────────────────────────────┐
  │              TensorFlow Serving                       │
  │                                                       │
  │  ┌──────────────┐     ┌────────────────────────────┐│
  │  │ REST / gRPC  │────►│  Model Manager             ││
  │  │ Client       │     │  ┌──────┐ ┌──────┐        ││
  │  └──────────────┘     │  │ v1   │ │ v2   │ ← hot  ││
  │                        │  │      │ │      │  reload ││
  │                        │  └──────┘ └──────┘        ││
  │                        └────────────────────────────┘│
  └──────────────────────────────────────────────────────┘
```

**Directory structure:**

```
models/
└── my_model/
    ├── 1/                    # version 1
    │   └── saved_model.pb
    │       └── variables/
    └── 2/                    # version 2 (auto-loaded)
        └── saved_model.pb
            └── variables/
```

**Docker launch:**

```bash
docker run -p 8501:8501 -p 8500:8500 \
  --mount type=bind,source=/path/to/models,target=/models \
  -e MODEL_NAME=my_model \
  tensorflow/serving
```

**Configuration file (models.config):**

```protobuf
model_config_list {
  config {
    name: "churn_model"
    base_path: "/models/churn_model"
    model_platform: "tensorflow"
    model_version_policy {
      specific {
        versions: 1
        versions: 2
      }
    }
  }
  config {
    name: "recommender"
    base_path: "/models/recommender"
    model_platform: "tensorflow"
  }
}
```

**Client call:**

```python
import requests
import json

data = {
    "instances": [
        {"age": 35.0, "income": 75000.0, "tenure": 24.0}
    ]
}

response = requests.post(
    "http://localhost:8501/v1/models/churn_model:predict",
    json=data,
)
print(response.json())
# {"predictions": [[0.87]]}
```

---

## TorchServe

Model server for PyTorch models. Packages models as Model Archive (.mar) files with custom handlers.

```
  ┌──────────────────────────────────────────────────────┐
  │                   TorchServe                          │
  │                                                       │
  │  ┌──────────┐  ┌────────────┐  ┌──────────────────┐ │
  │  │ Frontend │─►│ Workers    │─►│ Model Archive    │ │
  │  │ (Netty)  │  │ (Python   │  │ (.mar file)      │ │
  │  │ REST +   │  │  processes)│  │ • model.pt       │ │
  │  │ gRPC     │  │            │  │ • handler.py     │ │
  │  └──────────┘  └────────────┘  │ • requirements   │ │
  │                                 └──────────────────┘ │
  │  Management API: register, unregister, scale workers │
  └──────────────────────────────────────────────────────┘
```

**Package and serve:**

```bash
# Package model as .mar archive
torch-model-archiver \
  --model-name churn_model \
  --version 1.0 \
  --serialized-file model.pt \
  --handler handler.py \
  --export-path model_store/

# Start TorchServe
torchserve --start \
  --model-store model_store \
  --models churn_model=churn_model.mar \
  --ncs
```

**Custom handler:**

```python
# handler.py
import torch
import json
from ts.torch_handler.base_handler import BaseHandler


class ChurnHandler(BaseHandler):
    def preprocess(self, data):
        inputs = []
        for row in data:
            body = row.get("body") or row.get("data")
            if isinstance(body, (bytes, bytearray)):
                body = json.loads(body)
            inputs.append([body["age"], body["income"], body["tenure"]])
        return torch.tensor(inputs, dtype=torch.float32)
    
    def inference(self, data):
        with torch.no_grad():
            return self.model(data)
    
    def postprocess(self, inference_output):
        probs = torch.sigmoid(inference_output).tolist()
        return [{"probability": round(p[0], 4)} for p in probs]
```

**Management API:**

```bash
# Scale workers
curl -X PUT "http://localhost:8081/models/churn_model?min_worker=4"

# Register new model version
curl -X POST "http://localhost:8081/models?url=churn_model_v2.mar"
```

---

## NVIDIA Triton Inference Server

Framework-agnostic server supporting TensorFlow, PyTorch, ONNX, TensorRT, and custom backends. Best for multi-model, multi-GPU deployments.

```
  ┌──────────────────────────────────────────────────────────────┐
  │                  Triton Inference Server                      │
  │                                                               │
  │  ┌──────────┐   ┌───────────────────────────────────────┐   │
  │  │ HTTP /   │──►│ Model Repository                      │   │
  │  │ gRPC /   │   │                                       │   │
  │  │ C API    │   │ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ │   │
  │  └──────────┘   │ │ TF   │ │ PT   │ │ ONNX │ │ TRT  │ │   │
  │                  │ │model │ │model │ │model │ │model │ │   │
  │  Features:       │ └──────┘ └──────┘ └──────┘ └──────┘ │   │
  │  • Dynamic       │                                       │   │
  │    batching      │ ┌────────────────────────────────┐    │   │
  │  • Model         │ │ Ensemble Pipeline (chained)    │    │   │
  │    ensemble      │ │ preprocess → model → postproc  │    │   │
  │  • Concurrent    │ └────────────────────────────────┘    │   │
  │    model exec    └───────────────────────────────────────┘   │
  │  • GPU/CPU                                                    │
  │    scheduling                                                 │
  └──────────────────────────────────────────────────────────────┘
```

**Model repository structure:**

```
model_repository/
├── churn_onnx/
│   ├── config.pbtxt
│   ├── 1/
│   │   └── model.onnx
│   └── 2/
│       └── model.onnx
└── recommender_tf/
    ├── config.pbtxt
    └── 1/
        └── model.savedmodel/
```

**config.pbtxt:**

```protobuf
name: "churn_onnx"
platform: "onnxruntime_onnx"
max_batch_size: 64

input [
  {
    name: "input"
    data_type: TYPE_FP32
    dims: [3]
  }
]

output [
  {
    name: "output"
    data_type: TYPE_FP32
    dims: [1]
  }
]

dynamic_batching {
  preferred_batch_size: [8, 16, 32]
  max_queue_delay_microseconds: 100
}

instance_group [
  {
    count: 2
    kind: KIND_GPU
    gpus: [0]
  }
]
```

**Docker launch:**

```bash
docker run --gpus all -p 8000:8000 -p 8001:8001 -p 8002:8002 \
  -v /path/to/model_repository:/models \
  nvcr.io/nvidia/tritonserver:23.10-py3 \
  tritonserver --model-repository=/models
```

**Python client:**

```python
import tritonclient.http as httpclient
import numpy as np

client = httpclient.InferenceServerClient(url="localhost:8000")

inputs = httpclient.InferInput("input", [1, 3], "FP32")
inputs.set_data_from_numpy(np.array([[35.0, 75000.0, 24.0]], dtype=np.float32))

result = client.infer("churn_onnx", inputs=[inputs])
output = result.as_numpy("output")
print(f"Prediction: {output}")
```

---

## Comparison Table

```
  ┌──────────────────┬──────────────────┬──────────────────┬──────────────────┐
  │ Feature          │ TF Serving       │ TorchServe       │ Triton           │
  ├──────────────────┼──────────────────┼──────────────────┼──────────────────┤
  │ Frameworks       │ TensorFlow only  │ PyTorch only     │ TF, PT, ONNX,   │
  │                  │                  │                  │ TensorRT, custom │
  │ Protocol         │ REST + gRPC      │ REST + gRPC      │ REST + gRPC + C  │
  │ Batching         │ ✅ Server-side   │ ✅ Server-side   │ ✅ Dynamic batch │
  │ GPU support      │ ✅               │ ✅               │ ✅ Multi-GPU     │
  │ Model versioning │ ✅ Auto-reload   │ ✅ via .mar      │ ✅ Auto-reload   │
  │ Multi-model      │ ✅               │ ✅               │ ✅ + Ensemble    │
  │ Custom logic     │ ❌ Limited       │ ✅ Handlers      │ ✅ Python backend│
  │ Model format     │ SavedModel       │ .mar archive     │ Any (pluggable)  │
  │ Complexity       │ Low              │ Medium           │ High             │
  │ Best for         │ Pure TF stacks   │ Pure PyTorch     │ Multi-framework, │
  │                  │                  │ stacks           │ high-perf GPU    │
  │ Maintained by    │ Google           │ Meta (PyTorch)   │ NVIDIA           │
  └──────────────────┴──────────────────┴──────────────────┴──────────────────┘
```

---

## When to Use What

```
  ┌──────────────────────────────────────────────────────────────────┐
  │  Decision Guide                                                  │
  │                                                                  │
  │  Only TensorFlow models?                                         │
  │    YES ──► TF Serving (simplest for TF ecosystem)                │
  │    NO  ──► Only PyTorch models?                                  │
  │              YES ──► TorchServe                                  │
  │              NO  ──► Multiple frameworks or need GPU scheduling?  │
  │                        YES ──► Triton Inference Server           │
  │                        NO  ──► Custom API (FastAPI) may suffice  │
  │                                                                  │
  │  Need dynamic batching at scale?  ──► Triton or TF Serving       │
  │  Need model ensembles?            ──► Triton                     │
  │  Need custom pre/postprocessing?  ──► TorchServe or custom API   │
  └──────────────────────────────────────────────────────────────────┘
```

---

## Revision Questions

1. **What advantages do model servers provide over custom REST APIs?**
2. **How does TF Serving handle model versioning and hot-reloading?**
3. **What is a TorchServe handler, and why is it needed?**
4. **When would you choose Triton over TF Serving or TorchServe?**
5. **What is dynamic batching and why does it improve GPU utilization?**

---

[Previous: 02-rest-apis-for-ml.md](02-rest-apis-for-ml.md) | [Next: 04-containerizing-models.md](04-containerizing-models.md)
