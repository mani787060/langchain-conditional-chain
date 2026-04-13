# LangChain Conditional Routing (LCEL)
[![LangChain](https://img.shields.io/badge/Framework-LangChain-green)](https://www.langchain.com/)
[![Python](https://img.shields.io/badge/Python-3.9%2B-blue)](https://www.python.org/)
[![Gemini](https://img.shields.io/badge/Model-Google%20Gemini-4285F4)](https://ai.google.dev/)

## 🏗️ Project Overview
This repository demonstrates how to build **Dynamic AI Workflows** using LangChain's routing capabilities. In advanced AI systems, one prompt does not fit all. This project showcases how to implement a "Router" that analyzes the user's intent and programmatically decides which specialized sub-chain should handle the request. This approach maximizes model efficiency and ensures highly relevant responses.

---

## 🛠️ Key Technical Implementations

### 1. Intent-Based Routing
* **The Router Step:** Using a lightweight LLM call or logic gate to categorize the input (e.g., "Billing," "Technical Support," or "General Inquiry").
* **Conditional Branching:** Implementing `RunnableBranch` to map categories to specific `Prompt | Model` combinations.

### 2. Specialized Sub-Chains
* **Modular Experts:** Each branch acts as a "Specialist" with its own System Prompt tailored for a specific domain.
* **Fallback Mechanisms:** Ensuring the system remains robust by providing a general-purpose chain for inputs that don't fit specific categories.

### 3. Advanced LCEL Logic
* **Functional Routing:** Using standard Python functions within the LCEL pipe to handle complex branching logic that goes beyond simple keyword matching.
* **Efficient Token Usage:** By routing to shorter, more specific prompts, we reduce total token consumption compared to using one giant "do-everything" prompt.

---

## 💻 Tech Stack
* **Language:** Python 3.9+
* **Orchestration:** LangChain (`langchain-core`, `langchain-google-genai`)
* **Model:** Google Gemini Pro
* **Environment:** `python-dotenv`

