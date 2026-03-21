# 3.3 Recurrent Neural Networks (RNNs)

> **Subject 3.3 of Deep Learning** — Complete study guide covering sequence modeling, RNN architectures, LSTM, GRU, advanced techniques, and real-world applications.

---

## 📋 Course Overview

Recurrent Neural Networks (RNNs) are a class of neural networks designed to process **sequential data** — data where order matters. Unlike feedforward networks that treat each input independently, RNNs maintain a **hidden state** that acts as memory, allowing information to persist across time steps.

This subject covers the full RNN landscape: from basic recurrence concepts through LSTM and GRU gating mechanisms, to advanced architectures like attention and sequence-to-sequence models, and finally practical training techniques and the transition to Transformers.

```
                         ┌─────────────────────────────────────────────┐
                         │        RNN Learning Roadmap                 │
                         └─────────────────────────────────────────────┘

  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
  │  Unit 1      │    │  Unit 2      │    │  Unit 3      │    │  Unit 4 & 5  │
  │  Sequence    │───▶│  Basic RNN   │───▶│  BPTT &      │───▶│  LSTM & GRU  │
  │  Modeling    │    │  Architecture│    │  Gradients   │    │  Gating      │
  └──────────────┘    └──────────────┘    └──────────────┘    └──────┬───────┘
                                                                     │
                                                                     ▼
  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
  │  Unit 10     │    │  Unit 9      │    │  Unit 8      │    │  Unit 6 & 7  │
  │  Beyond      │◀───│  Training    │◀───│  RNN         │◀───│  Advanced    │
  │  RNNs        │    │  Techniques  │    │  Applications│    │  & Seq2Seq   │
  └──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘

  Foundations ──────────▶ Core Architectures ──────────▶ Practice & Beyond
```

---

## 🎯 Learning Objectives

By the end of this subject, you will be able to:

1. **Explain** why sequential data requires specialized architectures beyond feedforward networks
2. **Derive** the forward and backward passes of vanilla RNNs, including BPTT
3. **Analyze** the vanishing and exploding gradient problems and their solutions
4. **Implement** LSTM and GRU cells from scratch and explain each gate's role
5. **Design** bidirectional, deep, and encoder-decoder RNN architectures
6. **Build** sequence-to-sequence models with attention mechanisms
7. **Apply** RNNs to real tasks: language modeling, sentiment analysis, translation, time series
8. **Train** RNNs effectively using gradient clipping, dropout, and normalization
9. **Compare** RNNs with Transformers and decide when to use each

---

## 📑 Table of Contents

### Unit 1: Sequence Modeling
| # | Topic | File |
|---|-------|------|
| 1.1 | Why Sequence Models | [01-why-sequence-models.md](01-Sequence-Modeling/01-why-sequence-models.md) |
| 1.2 | Types of Sequence Problems | [02-types-of-sequence-problems.md](01-Sequence-Modeling/02-types-of-sequence-problems.md) |
| 1.3 | Limitations of Feedforward Networks | [03-limitations-of-feedforward.md](01-Sequence-Modeling/03-limitations-of-feedforward.md) |
| 1.4 | The Concept of Recurrence | [04-recurrence-concept.md](01-Sequence-Modeling/04-recurrence-concept.md) |

### Unit 2: Basic RNN
| # | Topic | File |
|---|-------|------|
| 2.1 | RNN Architecture | [01-rnn-architecture.md](02-Basic-RNN/01-rnn-architecture.md) |
| 2.2 | Hidden State Dynamics | [02-hidden-state.md](02-Basic-RNN/02-hidden-state.md) |
| 2.3 | Unrolling Through Time | [03-unrolling-through-time.md](02-Basic-RNN/03-unrolling-through-time.md) |
| 2.4 | Forward Propagation in RNN | [04-forward-propagation-rnn.md](02-Basic-RNN/04-forward-propagation-rnn.md) |
| 2.5 | Parameter Sharing | [05-parameter-sharing.md](02-Basic-RNN/05-parameter-sharing.md) |
| 2.6 | RNN Task Variants | [06-rnn-task-variants.md](02-Basic-RNN/06-rnn-task-variants.md) |

### Unit 3: Backpropagation Through Time
| # | Topic | File |
|---|-------|------|
| 3.1 | BPTT Algorithm | [01-bptt-algorithm.md](03-Backpropagation-Through-Time/01-bptt-algorithm.md) |
| 3.2 | Gradient Flow Through Time | [02-gradient-flow.md](03-Backpropagation-Through-Time/02-gradient-flow.md) |
| 3.3 | Vanishing Gradient Problem | [03-vanishing-gradient-problem.md](03-Backpropagation-Through-Time/03-vanishing-gradient-problem.md) |
| 3.4 | Exploding Gradient Problem | [04-exploding-gradient-problem.md](03-Backpropagation-Through-Time/04-exploding-gradient-problem.md) |
| 3.5 | Gradient Clipping | [05-gradient-clipping.md](03-Backpropagation-Through-Time/05-gradient-clipping.md) |

### Unit 4: LSTM
| # | Topic | File |
|---|-------|------|
| 4.1 | LSTM Motivation | [01-lstm-motivation.md](04-LSTM/01-lstm-motivation.md) |
| 4.2 | LSTM Cell Architecture | [02-lstm-cell-architecture.md](04-LSTM/02-lstm-cell-architecture.md) |
| 4.3 | Gates: Forget, Input, Output | [03-gates-forget-input-output.md](04-LSTM/03-gates-forget-input-output.md) |
| 4.4 | Cell State | [04-cell-state.md](04-LSTM/04-cell-state.md) |
| 4.5 | Complete Gate Equations | [05-gate-equations.md](04-LSTM/05-gate-equations.md) |
| 4.6 | How LSTM Solves Vanishing Gradient | [06-how-lstm-solves-vanishing.md](04-LSTM/06-how-lstm-solves-vanishing.md) |
| 4.7 | Peephole Connections | [07-peephole-connections.md](04-LSTM/07-peephole-connections.md) |

### Unit 5: GRU
| # | Topic | File |
|---|-------|------|
| 5.1 | GRU Architecture | [01-gru-architecture.md](05-GRU/01-gru-architecture.md) |
| 5.2 | Reset and Update Gates | [02-reset-and-update-gates.md](05-GRU/02-reset-and-update-gates.md) |
| 5.3 | GRU Equations | [03-gru-equations.md](05-GRU/03-gru-equations.md) |
| 5.4 | LSTM vs GRU Comparison | [04-lstm-vs-gru-comparison.md](05-GRU/04-lstm-vs-gru-comparison.md) |
| 5.5 | When to Use Which | [05-when-to-use-which.md](05-GRU/05-when-to-use-which.md) |

### Unit 6: Advanced RNN Architectures
| # | Topic | File |
|---|-------|------|
| 6.1 | Bidirectional RNN | [01-bidirectional-rnn.md](06-Advanced-RNN-Architectures/01-bidirectional-rnn.md) |
| 6.2 | Deep (Stacked) RNN | [02-deep-rnn.md](06-Advanced-RNN-Architectures/02-deep-rnn.md) |
| 6.3 | Encoder-Decoder Architecture | [03-encoder-decoder.md](06-Advanced-RNN-Architectures/03-encoder-decoder.md) |
| 6.4 | Attention Mechanism Basics | [04-attention-mechanism-basics.md](06-Advanced-RNN-Architectures/04-attention-mechanism-basics.md) |
| 6.5 | Teacher Forcing | [05-teacher-forcing.md](06-Advanced-RNN-Architectures/05-teacher-forcing.md) |

### Unit 7: Sequence-to-Sequence
| # | Topic | File |
|---|-------|------|
| 7.1 | Encoder-Decoder Framework | [01-encoder-decoder-framework.md](07-Sequence-to-Sequence/01-encoder-decoder-framework.md) |
| 7.2 | Applications | [02-applications.md](07-Sequence-to-Sequence/02-applications.md) |
| 7.3 | Beam Search Decoding | [03-beam-search-decoding.md](07-Sequence-to-Sequence/03-beam-search-decoding.md) |
| 7.4 | Limitations | [04-limitations.md](07-Sequence-to-Sequence/04-limitations.md) |

### Unit 8: RNN Applications
| # | Topic | File |
|---|-------|------|
| 8.1 | Language Modeling | [01-language-modeling.md](08-RNN-Applications/01-language-modeling.md) |
| 8.2 | Text Generation | [02-text-generation.md](08-RNN-Applications/02-text-generation.md) |
| 8.3 | Sentiment Analysis | [03-sentiment-analysis.md](08-RNN-Applications/03-sentiment-analysis.md) |
| 8.4 | Named Entity Recognition | [04-named-entity-recognition.md](08-RNN-Applications/04-named-entity-recognition.md) |
| 8.5 | Machine Translation | [05-machine-translation.md](08-RNN-Applications/05-machine-translation.md) |
| 8.6 | Speech Recognition Basics | [06-speech-recognition-basics.md](08-RNN-Applications/06-speech-recognition-basics.md) |
| 8.7 | Time Series Forecasting | [07-time-series-forecasting.md](08-RNN-Applications/07-time-series-forecasting.md) |

### Unit 9: Training RNNs
| # | Topic | File |
|---|-------|------|
| 9.1 | Truncated BPTT | [01-truncated-bptt.md](09-Training-RNNs/01-truncated-bptt.md) |
| 9.2 | Gradient Clipping | [02-gradient-clipping.md](09-Training-RNNs/02-gradient-clipping.md) |
| 9.3 | Learning Rate Scheduling | [03-learning-rate-scheduling.md](09-Training-RNNs/03-learning-rate-scheduling.md) |
| 9.4 | Dropout in RNNs | [04-dropout-in-rnns.md](09-Training-RNNs/04-dropout-in-rnns.md) |
| 9.5 | Layer Normalization | [05-layer-normalization.md](09-Training-RNNs/05-layer-normalization.md) |

### Unit 10: Beyond RNNs
| # | Topic | File |
|---|-------|------|
| 10.1 | Attention Mechanisms | [01-attention-mechanisms.md](10-Beyond-RNNs/01-attention-mechanisms.md) |
| 10.2 | Transformer Preview | [02-transformer-preview.md](10-Beyond-RNNs/02-transformer-preview.md) |
| 10.3 | RNN vs Transformer | [03-rnn-vs-transformer.md](10-Beyond-RNNs/03-rnn-vs-transformer.md) |
| 10.4 | When to Still Use RNNs | [04-when-to-still-use-rnns.md](10-Beyond-RNNs/04-when-to-still-use-rnns.md) |

---

## 📊 Subject Statistics

| Metric | Value |
|--------|-------|
| Total Units | 10 |
| Total Study Files | 52 |
| Estimated Study Time | 40–50 hours |
| Prerequisites | [3.1 Neural Network Fundamentals](../3.1-Neural-Networks/), [3.2 CNNs](../3.2-CNN/) |
| Key Libraries | PyTorch, NumPy |

---

## 🔑 Key Concepts at a Glance

```
Sequential Data ──▶ Recurrence ──▶ Hidden State ──▶ BPTT Training
                                        │
                         ┌───────────────┼───────────────┐
                         ▼               ▼               ▼
                    Vanilla RNN      LSTM (3 gates)   GRU (2 gates)
                    (simple but      (cell state +    (simplified
                     vanishing       forget/input/     reset/update
                     gradients)      output gates)     gates)
                         │               │               │
                         └───────┬───────┘───────────────┘
                                 ▼
                    ┌─────────────────────────┐
                    │   Advanced Architectures │
                    ├─────────────────────────┤
                    │ • Bidirectional RNN      │
                    │ • Deep/Stacked RNN       │
                    │ • Encoder-Decoder        │
                    │ • Attention Mechanism    │
                    │ • Seq2Seq Models         │
                    └────────────┬────────────┘
                                 ▼
                    ┌─────────────────────────┐
                    │   Applications           │
                    ├─────────────────────────┤
                    │ • Language Modeling       │
                    │ • Machine Translation    │
                    │ • Sentiment Analysis     │
                    │ • Speech Recognition     │
                    │ • Time Series Forecast   │
                    └────────────┬────────────┘
                                 ▼
                         Transformers &
                         Beyond RNNs
```

---

## 🧭 Navigation

| Direction | Link |
|-----------|------|
| ⬆️ Parent | [3. Deep Learning](../README.md) |
| ⬅️ Previous | [3.2 Convolutional Neural Networks](../3.2-CNN/README.md) |
| ➡️ Next | [3.4 Transformers](../3.4-Transformers/README.md) |
| 🏠 Home | [AIML Home](../../README.md) |

---

> **Study Tip:** Follow the units in order — each builds on the previous. Start with sequence modeling intuition, master the vanilla RNN, understand why gradients are problematic, then learn how LSTM/GRU solve those problems. The advanced architectures and applications tie everything together.

*Last updated: 2024*
