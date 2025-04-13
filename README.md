# DS-5690-Project: Assessing Code Documentation Quality from Popular Code LLMs <=70b parameters Using LLM as a Judge approach with Majority Voting

---

## 1. Problem Statement & Overview

### Problem Statement and Project Overview

The project aims to evaluate how good local models are in both code understanding and documentation. It also investigates whether increasing the number of parameters leads to better results, providing insights for anyone interested in running a code assistant locally. This project presents a systematic approach to assess the quality of automated code documentation. It integrates data preprocessing, prompt engineering, and an evaluation strategy to uncover insights into how different code LLMs perform in generating useful and understandable documentation.

---

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

---

### Reason for Choosing Models ≤70B

The decision to focus on models with 70B parameters or fewer is primarily driven by hardware constraints:

- **Hardware Limitations:**  
  Modern high-end laptops, those from Apple(M4), typically offer around ~128GB of VRAM (Highest for a single laptop).
- **Single Machine Feasibility:**  
  Within this VRAM limit, models up to 70B parameters are optimal for running on a single machine, making them both accessible and practical since most people cannot build and manage compute clusters.

---

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

**[Dataset :](https://huggingface.co/datasets/codeparrot/github-code)**

---

### Random Sampling and Preprocessing

- **Code Cleaning:**  
  - Remove comments, docstrings, and extra blank lines while keeping code structure intact.  
  - Discard snippets with embedded license information since many LLMs filter these out to protect copyright.
- **Quality Checks:**  
  - Keep code samples with a length between 500 and 2000 characters.  
  - Only include snippets containing at least one function or class definition.
- **Sampling Criteria:**  
  - Target Languages: Python, C, Java, JavaScript, C++.
  - Random Sampling 25 snippets per language.
  - **Parameters:**  
    - min_definitions = 1         # Require at least one function or class definition  
    - min_length = 500            # Minimum length (in characters) after cleaning  
    - max_length = 2000           # Maximum length (in characters) after cleaning

- **[Sampled Data](https://github.com/suryachand2k1/DS-5690-Project/blob/main/github_code_sample_random_5langs.csv)**

---

### Code Model Selection and Environment Setup

| Model Name       | Parameter Size |
|------------------|----------------|
| qwen2.5-coder    | 32B            |
| codellama        | 70B            |
| deepseek-coder   | 33B            |
| codegemma        | 7B             |
| codestral        | 22B            |

Models are locally used via Ollama. For more information, please visit **[Ollama](https://ollama.com)**.

---

### Documentation Prompts Strategy

- **Prompt 1: Detailed Documentation Guidelines**
  - **Persona:**  
    Establishes the role of a Code Documentation Expert to ensure authoritative and knowledgeable instructions.
  - **Step-by-Step Instructions:**  
    Outlines a clear process:
    - Analyze the entire source code.
    - Generate an overall file summary covering high-level purpose, main functionalities, and design.
    - Provide detailed documentation for each significant code block, including purpose, inputs, outputs, step-by-step breakdowns, and examples.
  - **Few-Shot Examples:**  
    Offers multiple examples across different languages (Python, JavaScript, Java, C/C++) to demonstrate the expected output format and style.
  - **Markdown Structuring:**  
    Uses Markdown to structure and present the instructions, making them clear, segmented, and easy for LLMs to follow.

- **Prompt 2: Enforcing Complete Code Documentation**
  - **Instruction Reinforcement:**  
    Requires strict adherence to the guidelines from Prompt 1 by instructing the LLM to include the complete, newly documented code.
  - **Output Clarity:**  
    Specifies that the final response must contain the fully documented code with no omissions, preserving the original functionality.
  - **Strict Rule Enforcement:**  
    Emphasizes following the documentation rules exactly as previously provided.
  - **Markdown Structuring:**  
    Again, employs Markdown to clearly delineate instructions and ensure that the response is structured and easily parsed by the LLM.

For full details on each prompt, refer to the following:
- [Prompt 1 Details](https://github.com/suryachand2k1/DS-5690-Project/blob/main/engineered_prompt-1.md)  
- [Prompt 2 Details](https://github.com/suryachand2k1/DS-5690-Project/blob/main/engineered_prompt-2.md)

---

### CSV Processing with Ollama Models

- **Workflow Overview:**
  - Loads a CSV of code samples and uses pre-downloaded Ollama models to generate documented code.
  - Two engineered prompts guide the transformation process: one with detailed documentation instructions and another enforcing complete, integrated output.
- **Conversation Management:**
  - Each code sample is processed as a new conversation—Ollama starts a fresh chat without any previous history.
  - A helper function maintains conversation history during the processing of a single code sample (storing the user message and model response as a list of conversations and passing them in) , ensuring clarity and proper tracking while keeping sessions isolated.
- **Incremental Output & Metrics:**
  - Responses and evaluation metrics (such as processing duration and token rates) are saved incrementally to CSV files.
  - Short pauses are added between requests to avoid overloading the API.

For more details, see the full code [here](https://github.com/suryachand2k1/DS-5690-Project/blob/main/DS-5690_Documenting.ipynb).

---

### Documentation Evaluation Prompt Strategy

- **Evaluation Focus:**  
  Assess only the added documentation against well-defined, binary (Yes/No) criteria without considering code functionality.
- **Structured Breakdown of 14 Binary Criteria:**
  1. **Overall File Summary - Presence:**  
     Is there an overall summary at the top of the file?
  2. **Overall File Summary - Content:**  
     Does the summary explain the purpose, main functionalities, and design?
  3. **Code Block Documentation - Purpose Explained:**  
     Is the role of each significant code block clearly described?
  4. **Code Block Documentation - Inputs/Arguments Detailed:**  
     Are the inputs or parameters listed and explained?
  5. **Code Block Documentation - Outputs/Return Values Detailed:**  
     Is the output or return value defined?
  6. **Code Block Documentation - Step-by-Step Explanation Provided:**  
     Is there a sequential explanation of the code block’s workings?
  7. **Code Block Documentation - Example Included:**  
     Is a representative example provided when applicable?
  8. **Language-Specific Formatting - Correct Syntax:**  
     Is the appropriate comment or docstring syntax used for the language?
  9. **Language-Specific Formatting - Correct Placement:**  
     Are documentation blocks placed immediately before or inside the relevant code blocks?
  10. **Preservation of Original Code - Functionality Unchanged:**  
      Has the original code logic been left intact?
  11. **Preservation of Original Code - Structure Preserved:**  
      Is the original formatting and structure maintained?
  12. **Overall Compliance - Full Adherence:**  
      Does the documented code fully follow the provided instructions?
  13. **Original Code Inclusion - Presence of Original Code Section:**  
      Is the complete original code included as a separate section?
  14. **Complete Preservation of Original Code - Completeness:**  
      Is there no modification or omission of the original code?
- **Scoring Approach:**  
  - Each criterion is answered with Yes/No (1 point for Yes).
  - All individual scores are aggregated for a final score out of 14.
- **Use of Markdown in Prompt Engineering:**  
  Markdown formatting is applied to:
  - Clearly organize the evaluation criteria.
  - Enhance readability and ensure that the LLM accurately interprets the instructions.

For further details, please refer to the full scoring prompt [here](https://github.com/suryachand2k1/DS-5690-Project/blob/main/engineered_prompt-scoring.md).

---

### Scoring and Aggregation of Evaluation Data

This scoring pipeline assesses the quality of generated documentation by using a unified, engineered scoring prompt across all scoring models. Each documented code sample is evaluated against 14 binary (Yes/No) criteria, resulting in a final score that objectively measures documentation quality. All scoring models employ the same scoring prompt to ensure consistency.

#### Scoring Models

| Scoring Model               | Description                                                                                       | Scoring Code Link                                                                                                                                                             |
|-----------------------------|---------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **GPT-4o**                  | Evaluates documentation quality via a binary rubric using OpenAI's GPT-4.                         | [Scoring Code](https://github.com/suryachand2k1/DS-5690-Project/blob/main/DS-5690_Scoring(Open%20AI).ipynb)                                                                 |
| **DeepSeek-V3**             | Assesses documentation quality using the DeepSeek API with the same unified scoring prompt.       | [Scoring Code](https://github.com/suryachand2k1/DS-5690-Project/blob/main/DS-5690_Scoring(DeepSeek)%20.ipynb)                                                                 |
| **Gemini (gemini-2.0-flash)** | Rates documentation with Google's Gemini model by applying identical binary criteria.             | [Scoring Code](https://github.com/suryachand2k1/DS-5690-Project/blob/main/DS-5690_Scoring(Gemini)%20.ipynb)                                                                    |
| **Anthropic's Claude**      | Uses Anthropic's Claude to evaluate documentation quality based on the unified scoring prompt.    | [Scoring Code](https://github.com/suryachand2k1/DS-5690-Project/blob/main/DS-5690_Scoring(Anthropic).ipynb)                                                                    |

#### Data Aggregation

The evaluation data from multiple systems (Anthropic, DeepSeek, Gemini, and GPT-4o) is consolidated into a single CSV by:
- Extracting model names, scores, and reasons from system-specific folders.
- Merging the data using a base file from the Anthropic-Scoring folder.
- Simultaneously compiling inference performance metrics—such as processing times and token speeds—from documentation models into another CSV for efficiency comparisons.

For the complete aggregation process, refer to the [Aggregation Code](https://github.com/suryachand2k1/DS-5690-Project/blob/main/Align_the_score_csv.ipynb).

---

## Results and Discussions

### Initial Prediction of Model Performance Based on Specifications

Below is a table displaying our documentation models used:

| Model Name         | Parameter Size |
|--------------------|----------------|
| deepseek-coder     | 33B            |
| qwen2.5-coder      | 32B            |
| codellama          | 70B            |
| codestral          | 22B            |
| codegemma          | 7B             |

**Question:**  
How would you order these models on our documentation task?  

<!-- Leave space for your predictions -->

---

<!-- Leave several blank lines below the question for clarity -->

  
  
  
  
  
  
  
Based on my initial assumption, here is the ranking (from best to worst predicted performance):

| Model Name         | Parameter Size |
|--------------------|----------------|
| codellama          | 70B            |
| deepseek-coder     | 33B            |
| qwen2.5-coder      | 32B            |
| codestral          | 22B            |
| codegemma          | 7B             |

---

### Observed Documentation Quality Results from Scoring LLMs

After analyzing the scorings from our evaluation LLMs (GPT-4o, DeepSeek, Gemini, and Anthropic's Claude), I observed the following trends in documentation quality:

![Collage maker project](https://github.com/user-attachments/assets/192b2ec0-faa2-4bb0-ac24-f1edf032bd5b)

---

### Critical Analysis

#### Consistent High Scorers

Models such as **qwen2.5-coder:32b** and **codestral** consistently received high scores, indicating detailed and well-structured documentation.

---

#### Moderate Performers

**deepseek-coder:33b** and **codegemma:7b** showed moderate performance with some variability between individual samples.

---

#### Unexpected Underperformance

Despite having the highest parameter count, **codellama:70b** underperformed—often returning only short summaries or default messages (e.g., stating that documentation cannot be provided due to harmful/unethical or proprietary content) instead of full documentation.

---

#### Additional Observations

- The aggregated scores reveal a clear divergence in documentation quality. High-scoring models delivered more complete and instructive outputs, while codellama frequently fell short of expectations.
- All scoring systems consistently ranked the models similarly, reinforcing the reliability of our unified, binary evaluation rubric.
- These results suggest that on the code documentation task, a higher parameter count does not necessarily equate to better performance.

---

### Supplementary: Inference Speeds

The table below shows inference performance metrics for each model, measured on a Mac Machine with an **M3 Ultra Chip** and **256GB of VRAM**. Results may vary depending on system load, code complexity, and other factors, but these figures provide a comparative baseline for average processing times and token speeds.

<img width="741" alt="Screenshot 2025-04-13 at 12 43 52 PM" src="https://github.com/user-attachments/assets/94b4cfbf-7838-40bf-a205-1978a9912946" />

> **Note:**  
> - All measurements were taken on a Mac with **M3 Ultra** and **256GB VRAM**.  
> - The **Avg Prompt1** and **Avg Prompt2** columns capture the average time (in seconds) it took each model to complete two main inference steps (or prompts).  
> - **Avg Overall Speed (tokens/s)** represents an aggregated throughput measure across both prompts.

---
