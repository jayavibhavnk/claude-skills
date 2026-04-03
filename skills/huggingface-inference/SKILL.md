---
name: huggingface-inference
description: Run inference with Hugging Face models including text generation, embeddings, and vision models.
metadata:
  priority: 8
  docs:
    - "https://huggingface.co/docs/inference-endpoints"
    - "https://huggingface.co/docs/api-inference"
  pathPatterns:
    - "**/inference/**"
    - "**/huggingface/**"
    - "**/embeddings/**"
  bashPatterns:
    - '\bhuggingface-cli\b'
    - '\btransformers\b'
    - '\bdiffusers\b'
  promptSignals:
    phrases:
      - "huggingface"
      - "hf endpoint"
      - "model inference"
    anyOf:
      - "text generation"
      - "embeddings"
      - "image generation"
      - "summarization"
      - "classification"
---

## Hugging Face Inference

### Using Inference API

#### Text Generation
```python
from huggingface_hub import InferenceClient

client = InferenceClient(model="mistralai/Mistral-7B-Instruct-v0.2")

response = client.text_generation(
    prompt="Once upon a time",
    max_new_tokens=100,
    temperature=0.7
)
```

#### Embeddings
```python
from huggingface_hub import InferenceClient

client = InferenceClient(model="BAAI/bge-large-en-v1.5")

embedding = client.feature_extraction(
    text="Hello, world!"
)
```

### Using Transformers Locally

```python
from transformers import pipeline

# Text generation
generator = pipeline("text-generation", model="gpt2")
result = generator("Once upon a time", max_new_tokens=50)

# Summarization
summarizer = pipeline("summarization", model="facebook/bart-large-cnn")
summary = summarizer(long_text, max_length=100)

# Classification
classifier = pipeline("zero-shot-classification")
labels = ["education", "business", "tech"]
result = classifier(text, labels)
```

### Image Generation

```python
from diffusers import StableDiffusionPipeline

pipe = StableDiffusionPipeline.from_pretrained("stabilityai/stable-diffusion-2-1")
pipe.to("cuda")

image = pipe("a photo of an astronaut riding a horse").images[0]
```

### Inference Endpoints

```bash
# Create inference endpoint
huggingface-cli endpoint create

# Configure for production
huggingface-cli endpoint set-config --name my-endpoint \
  --compute-type gpu-small \
  --model-name mistralai/Mistral-7B-Instruct-v0.2
```

### Best Practices

1. **Batch processing** - Process multiple inputs together
2. **Streaming** - Use for long-form text generation
3. **Caching** - Cache model weights when possible
4. **Quantization** - Use 4-bit/8-bit for memory efficiency

### Model Selection Guide

| Task | Recommended Models |
|------|-------------------|
| Text generation | Mistral-7B, Llama-3, Gemma |
| Embeddings | BGE-large, E5, Instructor |
| Image gen | SDXL, Flux, Playground |
| Vision | GPT-4V, LLaVA, CogVLM |
| Speech | Whisper, Bark, MusicGen |

### Performance Optimization

- Use `accelerate` library for efficient inference
- Apply 4-bit quantization with `bitsandbytes`
- Use Flash Attention 2 for faster transformers
- Enable KV cache for autoregressive models
