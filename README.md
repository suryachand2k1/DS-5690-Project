# DS-5690-Project: Assessing Code Documentation Quality from Popular Code LLMs <=70b parameters Using LLM as a Judge approach with Majority Voting

## 1. Problem Statement & Overview


### Problem Statement and Project Overview
The project aims to evaluate how good local models are in both code understanding and documentation. It also investigates whether increasing the number of parameters leads to better results, providing insights for anyone interested in running a code assistant locally. This project presents a systematic approach to assess the quality of automated code documentation. It integrates data preprocessing, prompt engineering, and a evaluation strategy to uncover insights into how different code LLMs perform in generating useful and understandable documentation.

### Proposed Approach
To tackle this challenge, the project follows these key steps:
- **Dataset Preparation:**  
  Leverage and clean data from the *codeparrot/github-code* dataset to ensure a diverse and high-quality set of code samples.
- **Prompt Engineering:**  
  Develop tailored prompt instructions that guide the LLMs to produce detailed documentation for the code examples.
- **Model Selection:**  
  Evaluate multiple local code-tailored LLMs with parameters up to 70B:
  - *qwen2.5-coder:32b*
  - *codellama:70b*
  - *deepseek-coder:33b*
  - *codegemma:7b*
  - *code:22b*
- **Evaluation Framework:**  
  Use an LLM-as-a-judge strategy, utilizing state-of-the-art scoring models (GPT-4o, Deepseek v3, Claude 3.7 sonnet, Gemini-2.0-flash) with a majority voting mechanism to ensure robust and objective evaluation of the generated documentation.

### Reason for Choosing Models ≤70B
The decision to focus on models with 70B parameters or fewer is primarily driven by hardware constraints:
- **Hardware Limitations:**  
  Modern high-end laptops, those from Apple(M4), typically offer around ~128GB of VRAM (Highest for a single laptop).
- **Single Machine Feasibility:**  
  Within this VRAM limit, models up to 70B parameters are optimal for running on a single machine, making them both accessible and practical since most people cannot build and manage compute clusters.


## 2. Methodology

### Dataset Overview

The **CodeParrot GitHub Code Dataset** is a large-scale collection of open-source code files, gathered from public GitHub repositories via BigQuery. It is designed for tasks like language modeling and code generation, supporting a diverse array of programming languages and licenses.

**Key Highlights:**
- **Scale:**  
  - ~115 million files  
  - Approximately 1TB uncompressed (~300GB compressed)
- **Content Details:**  
  - Includes code, repository names, file paths, programming languages, licenses, and file sizes.
- **Diversity:**  
  - Covers around 30 programming languages (e.g., Python, JavaScript, Java, C/C++, Go, etc.).
  - Features various license types (e.g., MIT, Apache-2.0, GPL-3.0).
- **Considerations:**  
  - Variability in file quality and potential presence of sensitive content (e.g., passwords, API keys).

**Dataset :** https://huggingface.co/datasets/codeparrot/github-code







