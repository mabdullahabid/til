# AI Model Sizing Rule of Thumb

When running local AI models, a useful rule of thumb: you need roughly **3x the model size** in RAM to handle back-and-forth messaging.

## Memory Requirements

- A 15GB model needs approximately 48GB total RAM
- Add an extra ~3GB for a 32K context window
- Without GPU acceleration, inference is very slow

## Smaller Models for Edge Devices

Some lightweight models can run on mobile devices:
- DeepSeek R1 1.5B
- DeepSeek R1 3.2B

These are useful for offline scenarios or when sending data to cloud APIs isn't acceptable.

## The Math

```
Base model: 15GB
Conversation overhead: ~2x (30GB)
Context window (32K): ~3GB
--------------------------------
Total recommended: ~48GB
```

## The Lesson

Plan your infrastructure around the conversation memory, not just the model weights. The gap between "model loads" and "model is usable" is significant.
