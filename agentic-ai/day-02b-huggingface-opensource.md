# Day 2B: Hugging Face & Open-Source LLM Ecosystem

## Overview
Master the Hugging Face ecosystem and open-source LLM tools for building cost-effective, customizable agents that can run locally or on your own infrastructure.

## Learning Objectives
- Use Hugging Face Transformers library
- Deploy models with Hugging Face Inference API
- Run local models with Ollama and LM Studio
- Use open-source models (Llama, Mistral, Phi)
- Build agents with HuggingFace Hub
- Fine-tune models for specific tasks

## The Hugging Face Ecosystem

### Core Components

1. **Transformers**: Library for using pre-trained models
2. **Datasets**: Large collection of datasets
3. **Inference API**: Serverless inference
4. **Spaces**: Host ML apps and demos
5. **Hub**: Model and dataset repository
6. **AutoTrain**: No-code model training

## Hands-On Exercises

### Exercise 1: Using Hugging Face Transformers

```python
# huggingface_basic.py
from transformers import pipeline, AutoModelForCausalLM, AutoTokenizer
import torch

# Method 1: Using Pipelines (Easiest)
def use_pipeline():
    """Use HuggingFace pipeline for text generation"""

    # Text generation pipeline
    generator = pipeline(
        "text-generation",
        model="microsoft/phi-2",
        device=0 if torch.cuda.is_available() else -1
    )

    result = generator(
        "Explain what an AI agent is:",
        max_length=200,
        num_return_sequences=1,
        temperature=0.7
    )

    print(result[0]['generated_text'])

# Method 2: Using Model and Tokenizer Directly (More Control)
def use_model_directly():
    """Use model and tokenizer for more control"""

    model_name = "microsoft/phi-2"
    tokenizer = AutoTokenizer.from_pretrained(model_name)
    model = AutoModelForCausalLM.from_pretrained(
        model_name,
        torch_dtype=torch.float16,
        device_map="auto"
    )

    prompt = "What are the key components of an AI agent?"
    inputs = tokenizer(prompt, return_tensors="pt").to(model.device)

    outputs = model.generate(
        **inputs,
        max_length=200,
        temperature=0.7,
        do_sample=True,
        top_p=0.9
    )

    response = tokenizer.decode(outputs[0], skip_special_tokens=True)
    print(response)

# Method 3: Chat Models with Conversation
def use_chat_model():
    """Use chat-tuned models"""

    pipe = pipeline(
        "text-generation",
        model="meta-llama/Llama-2-7b-chat-hf",
        torch_dtype=torch.float16,
        device_map="auto"
    )

    messages = [
        {"role": "system", "content": "You are a helpful AI assistant."},
        {"role": "user", "content": "Explain neural networks simply."}
    ]

    response = pipe(messages, max_length=500)
    print(response[0]['generated_text'])

# Run examples
if __name__ == "__main__":
    use_pipeline()
```

### Exercise 2: Hugging Face Inference API

```python
# hf_inference_api.py
from huggingface_hub import InferenceClient
import os

class HuggingFaceAgent:
    """Agent using HuggingFace Inference API"""

    def __init__(self, model="meta-llama/Llama-2-70b-chat-hf"):
        self.client = InferenceClient(
            token=os.getenv("HF_TOKEN")
        )
        self.model = model

    def chat(self, message, history=None):
        """Chat with model via Inference API"""

        messages = history or []
        messages.append({"role": "user", "content": message})

        response = self.client.chat_completion(
            messages=messages,
            model=self.model,
            max_tokens=500,
            temperature=0.7
        )

        assistant_message = response.choices[0].message.content
        messages.append({"role": "assistant", "content": assistant_message})

        return assistant_message, messages

    def generate(self, prompt, **kwargs):
        """Generate text"""
        return self.client.text_generation(
            prompt,
            model=self.model,
            **kwargs
        )

# Usage
agent = HuggingFaceAgent()
response, history = agent.chat("What is machine learning?")
print(response)

# Follow-up
response, history = agent.chat("Give me an example", history=history)
print(response)
```

### Exercise 3: Local Models with Ollama

```python
# ollama_agent.py
import ollama
import json

class OllamaAgent:
    """Agent using Ollama for local LLM inference"""

    def __init__(self, model="llama2"):
        self.model = model
        self.conversation_history = []

    def chat(self, message):
        """Chat with local model"""

        self.conversation_history.append({
            "role": "user",
            "content": message
        })

        response = ollama.chat(
            model=self.model,
            messages=self.conversation_history
        )

        assistant_message = response['message']['content']

        self.conversation_history.append({
            "role": "assistant",
            "content": assistant_message
        })

        return assistant_message

    def generate_with_tools(self, prompt, tools):
        """Generate with function calling"""

        response = ollama.chat(
            model=self.model,
            messages=[{"role": "user", "content": prompt}],
            tools=tools
        )

        # Check if model wants to call a tool
        if response['message'].get('tool_calls'):
            return response['message']['tool_calls']

        return response['message']['content']

    def stream_chat(self, message):
        """Stream response"""

        stream = ollama.chat(
            model=self.model,
            messages=[{"role": "user", "content": message}],
            stream=True
        )

        full_response = ""
        for chunk in stream:
            content = chunk['message']['content']
            print(content, end='', flush=True)
            full_response += content

        print()
        return full_response

# Usage
agent = OllamaAgent(model="llama2")
response = agent.chat("Explain AI agents in simple terms")
print(response)

# Streaming
agent.stream_chat("What are the benefits of local LLMs?")
```

### Exercise 4: Building Agent with Open-Source Models

```python
# opensource_agent.py
from transformers import pipeline, AutoModelForCausalLM, AutoTokenizer
import torch
from typing import List, Dict

class OpenSourceAgent:
    """Complete agent using open-source models"""

    def __init__(self, model_name="microsoft/Phi-3-mini-4k-instruct"):
        self.model_name = model_name
        self.tokenizer = AutoTokenizer.from_pretrained(model_name)
        self.model = AutoModelForCausalLM.from_pretrained(
            model_name,
            torch_dtype=torch.float16,
            device_map="auto",
            trust_remote_code=True
        )
        self.conversation = []

    def add_message(self, role: str, content: str):
        """Add message to conversation"""
        self.conversation.append({"role": role, "content": content})

    def generate_response(self, max_length=500):
        """Generate response using conversation history"""

        # Format conversation for the model
        prompt = self._format_conversation()

        inputs = self.tokenizer(prompt, return_tensors="pt").to(self.model.device)

        outputs = self.model.generate(
            **inputs,
            max_length=max_length,
            temperature=0.7,
            do_sample=True,
            top_p=0.9,
            pad_token_id=self.tokenizer.eos_token_id
        )

        response = self.tokenizer.decode(outputs[0], skip_special_tokens=True)

        # Extract just the new response
        response = response[len(prompt):].strip()

        return response

    def _format_conversation(self):
        """Format conversation for model"""
        formatted = ""
        for msg in self.conversation:
            if msg["role"] == "system":
                formatted += f"System: {msg['content']}\n"
            elif msg["role"] == "user":
                formatted += f"User: {msg['content']}\n"
            elif msg["role"] == "assistant":
                formatted += f"Assistant: {msg['content']}\n"

        formatted += "Assistant:"
        return formatted

    def chat(self, user_message: str) -> str:
        """Chat interface"""
        self.add_message("user", user_message)
        response = self.generate_response()
        self.add_message("assistant", response)
        return response

# Usage
agent = OpenSourceAgent()
agent.add_message("system", "You are a helpful AI assistant expert in technology.")

response = agent.chat("What is the difference between AI and ML?")
print(f"Agent: {response}")

response = agent.chat("Can you give me examples?")
print(f"Agent: {response}")
```

### Exercise 5: Using Hugging Face Datasets for RAG

```python
# hf_datasets_rag.py
from datasets import load_dataset
from sentence_transformers import SentenceTransformer
import numpy as np

class HuggingFaceRAG:
    """RAG using Hugging Face datasets and models"""

    def __init__(self):
        # Load embedding model from HuggingFace
        self.embedder = SentenceTransformer('sentence-transformers/all-MiniLM-L6-v2')
        self.documents = []
        self.embeddings = []

    def load_from_dataset(self, dataset_name, split="train", text_column="text"):
        """Load documents from HuggingFace dataset"""

        dataset = load_dataset(dataset_name, split=split)

        # Extract text
        self.documents = [item[text_column] for item in dataset][:1000]  # Limit for demo

        # Create embeddings
        print("Creating embeddings...")
        self.embeddings = self.embedder.encode(self.documents)
        print(f"Loaded {len(self.documents)} documents")

    def add_documents(self, documents: List[str]):
        """Add custom documents"""
        self.documents.extend(documents)
        new_embeddings = self.embedder.encode(documents)

        if len(self.embeddings) == 0:
            self.embeddings = new_embeddings
        else:
            self.embeddings = np.vstack([self.embeddings, new_embeddings])

    def search(self, query: str, top_k: int = 3):
        """Search for relevant documents"""

        query_embedding = self.embedder.encode([query])[0]

        # Calculate cosine similarity
        similarities = np.dot(self.embeddings, query_embedding) / (
            np.linalg.norm(self.embeddings, axis=1) * np.linalg.norm(query_embedding)
        )

        # Get top-k indices
        top_indices = np.argsort(similarities)[-top_k:][::-1]

        results = [
            {
                "document": self.documents[i],
                "score": similarities[i]
            }
            for i in top_indices
        ]

        return results

# Usage
rag = HuggingFaceRAG()

# Add custom documents
rag.add_documents([
    "Python is a high-level programming language.",
    "Machine learning is a subset of artificial intelligence.",
    "Neural networks are inspired by biological neural networks.",
    "Deep learning uses multiple layers of neural networks."
])

# Search
results = rag.search("What is Python?", top_k=2)
for i, result in enumerate(results, 1):
    print(f"{i}. {result['document']} (score: {result['score']:.3f})")
```

## Popular Open-Source Models

### Small Models (< 10B parameters)
- **Phi-3 Mini (3.8B)**: Microsoft's efficient model
- **Gemma 7B**: Google's open model
- **Mistral 7B**: Strong performance, efficient
- **Llama 3 8B**: Meta's latest small model

### Medium Models (10B-30B)
- **Mixtral 8x7B**: Mixture of experts
- **Llama 3 70B**: Powerful reasoning
- **Command R+**: Cohere's chat model

### Large Models (> 30B)
- **Llama 3 405B**: Meta's largest
- **Falcon 180B**: TII's open model

## Local Deployment Tools

### 1. Ollama
```bash
# Install
curl -fsSL https://ollama.com/install.sh | sh

# Run a model
ollama run llama2

# Pull a model
ollama pull mistral

# List models
ollama list
```

### 2. LM Studio
- GUI application for running local LLMs
- Easy model download and switching
- Built-in chat interface
- API server mode

### 3. vLLM (Production Inference)
```python
from vllm import LLM, SamplingParams

llm = LLM(model="facebook/opt-125m")
sampling_params = SamplingParams(temperature=0.8, top_p=0.95)

outputs = llm.generate(["Tell me about AI"], sampling_params)
for output in outputs:
    print(output.outputs[0].text)
```

### 4. Text Generation Inference (TGI)
```bash
# Docker
docker run --gpus all -p 8080:80 \
  -v $PWD/data:/data \
  ghcr.io/huggingface/text-generation-inference:latest \
  --model-id meta-llama/Llama-2-7b-chat-hf
```

## Model Comparison

| Model | Size | Speed | Quality | Use Case |
|-------|------|-------|---------|----------|
| Phi-3 Mini | 3.8B | ⚡⚡⚡ | ⭐⭐⭐ | Edge devices, fast inference |
| Mistral 7B | 7B | ⚡⚡ | ⭐⭐⭐⭐ | General purpose |
| Llama 3 8B | 8B | ⚡⚡ | ⭐⭐⭐⭐ | Chat, reasoning |
| Mixtral 8x7B | 47B | ⚡ | ⭐⭐⭐⭐⭐ | Complex tasks |

## Fine-Tuning with Hugging Face

```python
# fine_tune_example.py
from transformers import AutoModelForCausalLM, AutoTokenizer, Trainer, TrainingArguments
from datasets import load_dataset

def fine_tune_model():
    """Fine-tune a model on custom data"""

    # Load model and tokenizer
    model = AutoModelForCausalLM.from_pretrained("microsoft/phi-2")
    tokenizer = AutoTokenizer.from_pretrained("microsoft/phi-2")

    # Load dataset
    dataset = load_dataset("your-dataset")

    # Tokenize dataset
    def tokenize_function(examples):
        return tokenizer(examples["text"], truncation=True, padding="max_length")

    tokenized_dataset = dataset.map(tokenize_function, batched=True)

    # Training arguments
    training_args = TrainingArguments(
        output_dir="./results",
        num_train_epochs=3,
        per_device_train_batch_size=4,
        save_steps=10_000,
        save_total_limit=2,
    )

    # Trainer
    trainer = Trainer(
        model=model,
        args=training_args,
        train_dataset=tokenized_dataset["train"],
        eval_dataset=tokenized_dataset["test"]
    )

    # Train
    trainer.train()

    # Save
    trainer.save_model("./fine-tuned-model")
```

## Advantages of Open-Source

✅ **Cost**: Free to use, run locally
✅ **Privacy**: Data stays on your infrastructure
✅ **Customization**: Fine-tune for your needs
✅ **No Rate Limits**: Scale as needed
✅ **Offline**: Works without internet

## Disadvantages

❌ **Hardware**: Requires GPUs for larger models
❌ **Maintenance**: You manage infrastructure
❌ **Quality**: May lag behind GPT-4/Claude
❌ **Setup**: More complex than API calls

## Best Practices

1. **Start Small**: Test with 7B models first
2. **Quantization**: Use 4-bit/8-bit for memory efficiency
3. **Batching**: Batch requests for throughput
4. **Caching**: Cache model outputs
5. **Monitoring**: Track GPU usage and latency

## Resources

- [Hugging Face Documentation](https://huggingface.co/docs)
- [Transformers Library](https://github.com/huggingface/transformers)
- [Ollama Documentation](https://ollama.com/docs)
- [vLLM Documentation](https://docs.vllm.ai/)
- [Open LLM Leaderboard](https://huggingface.co/spaces/HuggingFaceH4/open_llm_leaderboard)

## Daily Challenge

Build an agent that:
1. Runs locally using Ollama or Hugging Face
2. Uses a 7B model (Mistral, Llama, or Phi)
3. Implements RAG with Hugging Face embeddings
4. Has no external API dependencies
5. Compares performance with GPT-3.5

---

**Progress**: Day 2B completed - Open-Source Mastery!

[← Previous: Day 2](./day-02.md) | [Back to Overview](../README.md) | [Next: Day 3 →](./day-03.md)
