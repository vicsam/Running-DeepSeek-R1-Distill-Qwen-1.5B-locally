# Running-DeepSeek-R1-Distill-Qwen-1.5B-locally

# Self-definition Introduction Model: safetensors Format and GGUF Format

## 1. .safetensors Format

### Introduction
`safetensors` is a safe and efficient machine learning model weight storage format proposed by Hugging Face. It aims to replace traditional PyTorch `.pt` or `.pth` files to mitigate security vulnerabilities associated with `pickle` serialization, such as malicious code execution.

### Core Features

#### 1. Security
- Stores only tensor data, without executable code, eliminating the risk of malicious code injection.
- Limits file header size (e.g., 100MB) and memory access scope to prevent internal overflow attacks in extreme cases.

#### 2. Efficient Loading
- **Zero-copy technology**: Supports direct memory mapping (`mmap`), avoiding unnecessary CPU data replication and achieving 2x faster loading on Linux compared to PyTorch.
- **Lazy loading**: Enables partial tensor loading, making it suitable for distributed or multi-GPU environments.

#### 3. Cross-Framework Compatibility
- Supports PyTorch, TensorFlow, and other frameworks but is primarily integrated within the Hugging Face ecosystem.
- Commonly used for secure and fast model deployment.

#### 4. Limitations
- Does not store model metadata (e.g., architecture, hyperparameters), requiring external configuration files.
- Lacks advanced serialization and quantization support for large models.

---

## 2. .gguf Format

### Introduction
`.gguf` (GPT-Generated Unified Format) was developed by **Georgi Gerganov**, the creator of `llama.cpp`, as a successor to the `GGML` format. It optimizes large language model (LLM) storage and loading efficiency through binary encoding and memory mapping techniques while supporting quantization to reduce resource consumption.

### Core Features

#### 1. Efficiency and Cross-Platform Support
- **Memory-mapped loading (mmap)**: Loads models directly from disk to memory, reducing RAM usage and startup time, especially for CPU inference.
- **Single-file deployment**: Contains all metadata (e.g., model architecture, hyperparameters), eliminating external dependencies and simplifying cross-platform sharing.

#### 2. Quantization Support
- Supports multiple quantization schemes from 2-bit to 8-bit (e.g., `Q4_K`, `Q5_K`).
- Balances model accuracy and resource usage, significantly reducing file sizes (e.g., a `70B` model can be compressed from `140GB` to `20GB`).
- Flexible quantization settings allow deployment on low-resource devices (e.g., mobile phones, edge computing).

#### 3. Scalability and Compatibility
- Uses a key-value structured metadata format, ensuring backward compatibility.
- Official tools allow conversion from PyTorch and `safetensors` formats to `.gguf`.

### Application Scenarios
- Designed for efficient deployment of large language models (e.g., `LLaMA`, `Gemma`, `Qwen`).
- More than **6,000** `.gguf` models are available on Hugging Face.
- Model file names typically follow the pattern: `Q<bit>_<variant>.gguf` (e.g., `q4_k_m.gguf`), where the quantization scheme impacts performance and accuracy.

---

## 3. Comparison Summary

| Feature           | .safetensors                          | .gguf                                      |
|------------------|----------------------------------|------------------------------------------|
| **Core Goal**    | Safe storage & fast loading    | Efficient LLM loading + quantization support |
| **Security**     | High (no executable code risk) | Medium (depends on binary analysis security) |
| **Metadata Support** | None                         | Includes full metadata |
| **Quantization Support** | None                         | Multi-level quantization support |
| **Use Cases**    | General model sharing, Hugging Face ecosystem | Large model inference, low-resource deployment |
| **Cross-Platform Support** | Depends on framework support | Self-contained, no external dependencies |
| **Major Ecosystem** | Hugging Face | `llama.cpp`, Hugging Face |

---

## 4. Getting Started with GGUF

###  Downloading a `.gguf` File
You can search for and download `.gguf` files from Hugging Face. 

### Cloning the DeepSeek-R1-Distill-Qwen-1.5B Repository

To clone the model from Hugging Face, run the following command:

```bash
git clone https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-1.5B
````
# Convert .safetensors Format to .gguf Format Using `llama.cpp`

Since **Ollama** only supports the `.gguf` format, models in `.safetensors` format need to be converted to `.gguf`.

##  Download `llama.cpp` Library

# To get started, clone the `llama.cpp` repository:

```bash
git clone https://github.com/ggerganov/llama.cpp.git
````

# Open the llama directory
```bash
cd llama.cpp
```

# Install the requirements
```bash
pip install -r requirements.txt
```

# Create the conversion file
```bash
python convert_hf_to_gguf.py -h
```
# After installation, convert to which ever quantization schemes preferable. 

Command example if on windows

```bash
python convert_hf_to_gguf.py "C:/Users/<username>/documents/ollama/DeepSeek-R1-Distill-Qwen-1.5B/" --outfile "C:/Users/<username>/documents/ollama/DeepSeek-R1-Distill-Qwen-1.5B-q8_0.gguf" --outtype q8_0
````

Notice in this conversion, the model is stored in my `ollama` folder, which is then convert to q8_0; this is preferable for me due to the capacity of my PC. The converted form of the model is then stored in the same folder but with a different name `DeepSeek-R1-Distill-Qwen-1.5B-q8_0.gguf`

After this is done, open the folder where the converted model is stored; new form should have the extension `.gguf` attached to it

Create a Modelfile in same folder where the converted version is stored; Modelfile should be a txt file.

This is what should be in your Modelfile

```bash
FROM ./my_DeepSeek-R1-Distill-Qwen-1.5B-q8_0.gguf
TEMPLATE """{{- if .System }}{{ .System }}{{ end }}
{{- range $i, $_ := .Messages }}
{{- $last := eq (len (slice $.Messages $i)) 1}}
{{- if eq .Role "user" }}<｜User｜>{{ .Content }}
{{- else if eq .Role "assistant" }}<｜Assistant｜>{{ .Content }}{{- if not $last }}<｜end▁of▁sentence｜>{{- end }}
{{- end }}
{{- if and $last (ne .Role "assistant") }}<｜Assistant｜>{{- end }}
{{- end }}"""
PARAMETER stop "<|begin▁of▁sentence|>"
PARAMETER stop "<|end▁of▁sentence|>"
PARAMETER stop "<|User|>"
PARAMETER stop "<|Assistant|>"
```

The first line that says `FROM ./DeepSeek-R1-Distill-Qwen-1.5B-q8_0.gguf`,  `DeepSeek-R1-Distill-Qwen-1.5B-q8_0.gguf` should be replaced with the name given to your converted model

After that is done save the Modelfile

# ollama create a model

```bash
ollama create DeepSeek-R1-Distill-Qwen-1.5B-q8_0.gguf -f Modelfile
```

Note: this `DeepSeek-R1-Distill-Qwen-1.5B-q8_0.gguf` should be replaced with the name given to your converted model

# Ollama running model

```bash
ollama run DeepSeek-R1-Distill-Qwen-1.5B-q8_0.gguf
```

Note: this `DeepSeek-R1-Distill-Qwen-1.5B-q8_0.gguf` should be replaced with the name given to your converted model

<img width="863" alt="image" src="https://github.com/user-attachments/assets/2a0c73e7-5d28-43de-9e5e-42c801618b6c" />

# Connecting a User Interface (UI) for DeepSeek-R1-Distill-Qwen-1.5B  

To enhance the usability of your locally running **DeepSeek-R1-Distill-Qwen-1.5B** model, you can connect it to a graphical user interface (GUI). For this guide, we’ll use **Chatbox UI**, a lightweight and intuitive tool designed for interacting with language models.  

---

## Step 1: Download and Install Chatbox UI  

Visit the official **Chatbox UI** website:  
🔗 [Chatbox AI](https://chatboxai.app/en)  

Download the appropriate version for your operating system:  
- **Windows**  
- **macOS**  
- **Linux**  

Follow the installation instructions provided on the website.  

---

## Step 2: Configure Chatbox UI  

Once installed, configure **Chatbox UI** to connect to your locally running **Ollama** model.  

### 1️⃣ Launch Chatbox UI  
- Open the application from your desktop or terminal.  

### 2️⃣ Open Settings  
- Navigate to the **Settings** menu within Chatbox UI.  
- This is usually accessible via the gear icon or a dedicated settings tab.  

### 3️⃣ Connect to Ollama  
- In **Settings**, locate the option to connect to an external model provider.  
- Select **Ollama** as the backend provider.  
