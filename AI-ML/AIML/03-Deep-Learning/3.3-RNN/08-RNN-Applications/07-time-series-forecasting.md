# Time Series Forecasting

> **Unit 8, Chapter 7** — LSTM for time series, multi-step forecasting, sliding window approach, and practical example.

---

## 📋 Overview

**Time series forecasting** predicts future values based on historical patterns. LSTMs excel at capturing temporal dependencies, seasonality, and trends in data like stock prices, weather, energy consumption, and sensor readings.

---

## 📐 Sliding Window Approach

```
Original time series: [v₁, v₂, v₃, v₄, v₅, v₆, v₇, v₈, v₉, v₁₀]

Window size = 4, Forecast horizon = 2:

Sample 1: Input: [v₁, v₂, v₃, v₄]  → Target: [v₅, v₆]
Sample 2: Input: [v₂, v₃, v₄, v₅]  → Target: [v₆, v₇]
Sample 3: Input: [v₃, v₄, v₅, v₆]  → Target: [v₇, v₈]
Sample 4: Input: [v₄, v₅, v₆, v₇]  → Target: [v₈, v₉]

   ┌──────────┐  ┌───┐
   │  Window   │  │Out│
   │  (Input)  │  │put│
   └──────────┘  └───┘
   ├──v₁─v₂─v₃─v₄──┤─v₅─v₆─┤
        ├──v₂─v₃─v₄─v₅──┤─v₆─v₇─┤
             ├──v₃─v₄─v₅─v₆──┤─v₇─v₈─┤
   ─────────────────────────────────────▶ time
```

---

## 💻 Complete PyTorch Example

```python
import torch
import torch.nn as nn
import numpy as np

class LSTMForecaster(nn.Module):
    def __init__(self, input_dim=1, hidden_dim=64, num_layers=2,
                 output_dim=1, forecast_horizon=1):
        super().__init__()
        self.lstm = nn.LSTM(input_dim, hidden_dim, num_layers,
                            batch_first=True, dropout=0.2)
        self.fc = nn.Linear(hidden_dim, output_dim * forecast_horizon)
        self.forecast_horizon = forecast_horizon
        self.output_dim = output_dim
    
    def forward(self, x):
        """x: (batch, seq_len, input_dim)"""
        output, (h_n, _) = self.lstm(x)
        # Use last hidden state
        prediction = self.fc(output[:, -1, :])
        return prediction.view(-1, self.forecast_horizon, self.output_dim)

def create_sequences(data, window_size, horizon):
    """Create sliding window sequences"""
    X, y = [], []
    for i in range(len(data) - window_size - horizon + 1):
        X.append(data[i:i+window_size])
        y.append(data[i+window_size:i+window_size+horizon])
    return np.array(X), np.array(y)

# Generate synthetic data (sine wave with noise)
np.random.seed(42)
t = np.linspace(0, 20 * np.pi, 1000)
data = np.sin(t) + 0.1 * np.random.randn(1000)

# Normalize
mean, std = data.mean(), data.std()
data_norm = (data - mean) / std

# Create sequences
window_size = 50
horizon = 10
X, y = create_sequences(data_norm, window_size, horizon)

# Convert to tensors
X_train = torch.FloatTensor(X[:700]).unsqueeze(-1)  # (700, 50, 1)
y_train = torch.FloatTensor(y[:700]).unsqueeze(-1)  # (700, 10, 1)
X_test = torch.FloatTensor(X[700:]).unsqueeze(-1)
y_test = torch.FloatTensor(y[700:]).unsqueeze(-1)

# Train
model = LSTMForecaster(input_dim=1, hidden_dim=64, forecast_horizon=horizon)
optimizer = torch.optim.Adam(model.parameters(), lr=0.001)
criterion = nn.MSELoss()

for epoch in range(100):
    model.train()
    pred = model(X_train)
    loss = criterion(pred, y_train)
    optimizer.zero_grad()
    loss.backward()
    torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)
    optimizer.step()
    
    if epoch % 20 == 0:
        model.eval()
        with torch.no_grad():
            test_pred = model(X_test)
            test_loss = criterion(test_pred, y_test)
        print(f"Epoch {epoch}: train_loss={loss.item():.4f}, test_loss={test_loss.item():.4f}")

print(f"\nFinal test MSE: {test_loss.item():.6f}")
print(f"Final test RMSE: {test_loss.item()**0.5:.6f}")
```

---

## 📊 Multi-Step Forecasting Strategies

```
┌──────────────────┬──────────────────────────────────────────┐
│ Strategy         │ Description                              │
├──────────────────┼──────────────────────────────────────────┤
│ Direct           │ Predict all future steps at once         │
│                  │ Output = FC(h_T) → [ŷ_{T+1},...,ŷ_{T+k}]│
├──────────────────┼──────────────────────────────────────────┤
│ Recursive        │ Predict one step, feed back as input     │
│                  │ ŷ_{T+1} → input → ŷ_{T+2} → ...        │
│                  │ Error accumulates!                       │
├──────────────────┼──────────────────────────────────────────┤
│ Seq2Seq          │ Encoder reads history, decoder generates │
│                  │ future autoregressively                  │
└──────────────────┴──────────────────────────────────────────┘
```

---

## 📋 Summary Table

| Concept | Description |
|---------|-------------|
| Sliding Window | Create fixed-size input/output pairs from series |
| Normalization | Essential — use Z-score or min-max scaling |
| LSTM Advantage | Captures seasonality, trends, long patterns |
| Multi-step | Direct (all at once) or recursive (one at a time) |
| Metrics | MSE, RMSE, MAE, MAPE |
| Key Tip | Always use gradient clipping for time series |

---

## ❓ Revision Questions

1. **How does the sliding window technique convert a time series into supervised learning samples?**
2. **Why is data normalization critical for LSTM time series models?**
3. **Compare direct and recursive multi-step forecasting. What are the trade-offs?**
4. **How would you handle multiple input features (multivariate time series)?**
5. **What window size would you choose for data with weekly seasonality sampled daily?**

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬅️ Previous | [Speech Recognition](06-speech-recognition-basics.md) |
| ➡️ Next | [Truncated BPTT](../09-Training-RNNs/01-truncated-bptt.md) |
| ⬆️ Unit Overview | [README](../README.md) |
