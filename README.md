# LangChain Conditional Chains using LCEL

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue.svg)](https://www.python.org/)
[![LangChain](https://img.shields.io/badge/Framework-LangChain-green.svg)](https://www.langchain.com/)
[![Google Gemini](https://img.shields.io/badge/LLM-Google%20Gemini-4285F4.svg)](https://ai.google.dev/)
[![Pydantic](https://img.shields.io/badge/Validation-Pydantic-E92063.svg)](https://docs.pydantic.dev/)

A beginner-friendly Generative AI project demonstrating **conditional routing with LangChain Expression Language (LCEL)**. The application uses Google Gemini to classify feedback as positive or negative and then routes it to the appropriate response-generation chain using `RunnableBranch`.

## 📌 Project Overview

Not every user input should be processed in the same way. In many AI applications, the system must first understand the input and then choose the most suitable action.

This project demonstrates how to build a conditional workflow that:

1. Accepts feedback text as input.
2. Classifies the feedback as positive or negative.
3. Uses Pydantic to validate the classification output.
4. Routes the result to the appropriate response-generation chain.
5. Generates a suitable reply based on the detected sentiment.
6. Displays the workflow structure using LangChain's graph functionality.

The current example uses the feedback:

`This is a beautiful phone`

The model classifies the feedback and generates an appropriate response based on its sentiment.

## 🎯 Project Objectives

- Understand conditional routing in LangChain.
- Learn how to use `RunnableBranch` and `RunnableLambda`.
- Build workflows using LangChain Expression Language (LCEL).
- Classify text using a language model.
- Generate structured output with Pydantic.
- Validate sentiment categories using Python's `Literal` type.
- Route inputs to different prompt templates based on classification results.
- Visualize the composed workflow using `get_graph()`.

## 🏗️ Workflow Architecture

```text
                  Feedback Text
                       |
                       v
                 Prompt Template
                       |
                       v
                  Google Gemini
                       |
                       v
               Pydantic Output Parser
                       |
                       v
             Sentiment: Positive / Negative
                       |
                       v
                 RunnableBranch
                   /       \
                  /         \
          Positive?         Negative?
              |                 |
              v                 v
       Positive Prompt    Negative Prompt
              |                 |
              v                 v
         Google Gemini     Google Gemini
              |                 |
              v                 v
       Positive Reply      Negative Reply
                  \         /
                   \       /
                 Final Response
```

If the sentiment does not match either configured category, the workflow has a fallback response.

## ⚙️ Technical Implementation

### 1. Integrating Google Gemini

The project uses LangChain's Google Gemini integration to generate classifications and responses.

```python
from langchain_google_genai import ChatGoogleGenerativeAI

model = ChatGoogleGenerativeAI(
    model="gemini-3.1-flash-lite-preview"
)
```

The configured model is a preview model. Availability and API access may change, so check Google's documentation if you encounter a model-access error.

### 2. Defining the Sentiment Schema

Pydantic defines the expected structure of the classification output.

```python
from pydantic import BaseModel, Field
from typing import Literal

class Feedback(BaseModel):
    sentiment: Literal["positive", "negative"] = Field(
        description="Give the sentiment of the feedback"
    )
```

The `Feedback` model specifies that the sentiment must be either `positive` or `negative`.

Using `Literal` restricts the accepted values and helps prevent unexpected categories from entering the routing stage.

### 3. Creating the Pydantic Output Parser

```python
from langchain_core.output_parsers import PydanticOutputParser

parser2 = PydanticOutputParser(
    pydantic_object=Feedback
)
```

The parser provides formatting instructions for the model and parses its response into a Pydantic object.

This allows the next stage to access the classification through the `sentiment` attribute.

### 4. Building the Classification Prompt

```python
from langchain_core.prompts import PromptTemplate

prompt1 = PromptTemplate(
    template=(
        "Classify the sentiment of the following feedback text "
        "into positive or negative:\n"
        "{feedback}\n"
        "{format_instruction}"
    ),
    input_variables=["feedback"],
    partial_variables={
        "format_instruction": parser2.get_format_instructions()
    }
)
```

The prompt receives the user's feedback and includes the output-format instructions generated by the Pydantic parser.

The classifier chain connects the prompt, model, and parser:

```python
classifier_chain = prompt1 | model | parser2
```

The output is a validated `Feedback` object containing the predicted sentiment.

### 5. Creating Specialized Response Prompts

The project defines two different prompts.

**Positive feedback prompt:**

```python
prompt2 = PromptTemplate(
    template="Write an appropriate response to this positive feedback\n{feedback}",
    input_variables=["feedback"]
)
```

**Negative feedback prompt:**

```python
prompt3 = PromptTemplate(
    template="Write an appropriate response to this negative feedback\n{feedback}",
    input_variables=["feedback"]
)
```

Each prompt is designed for a different sentiment category.

### 6. Implementing Conditional Routing with `RunnableBranch`

```python
from langchain_core.runnables import RunnableBranch, RunnableLambda
from langchain_core.output_parsers import StrOutputParser

parser = StrOutputParser()

branch_chain = RunnableBranch(
    (
        lambda x: x.sentiment == "positive",
        prompt2 | model | parser
    ),
    (
        lambda x: x.sentiment == "negative",
        prompt3 | model | parser
    ),
    RunnableLambda(lambda x: "could not find sentiment")
)
```

`RunnableBranch` evaluates its conditions in order and executes the first matching branch.

- **Positive feedback:** Runs the positive-response chain.
- **Negative feedback:** Runs the negative-response chain.
- **Fallback:** Returns a default message if neither condition matches.

The `StrOutputParser` converts the generated reply into a plain string.

### 7. Combining the Complete Workflow

```python
chain = classifier_chain | branch_chain
```

The first chain classifies the feedback. Its Pydantic result is then passed to the conditional branch, which chooses the appropriate response-generation chain.

Execute the complete workflow using:

```python
result = chain.invoke({
    "feedback": "This is a beautiful phone"
})

print(result)
```

The final output is a model-generated response appropriate to the detected sentiment.

### 8. Visualizing the Workflow

```python
chain.get_graph().print_ascii()
```

This prints an ASCII representation of the workflow, helping you understand how classification and conditional routing are connected.

## 🛠️ Tech Stack

| Technology | Purpose |
|---|---|
| Python | Main programming language |
| LangChain Core | Prompt templates and runnable components |
| LangChain Expression Language | Workflow composition |
| `RunnableBranch` | Conditional execution |
| `RunnableLambda` | Python-based fallback logic |
| Google Gemini | Sentiment classification and response generation |
| Pydantic | Structured output validation |
| `PydanticOutputParser` | Parses the sentiment classification |
| `StrOutputParser` | Converts the final model response into a string |
| `python-dotenv` | Loads environment variables |

## 📂 Repository Structure

```text
langchain-conditional-chain/
│
├── conditional_chain.py   # Conditional LCEL workflow
├── README.md              # Project documentation
└── .env                   # Local API key configuration (not committed)
```

## 🚀 Getting Started

### Prerequisites

- Python 3.9 or later
- A Google Gemini API key
- Internet access for API requests

### Step 1: Clone the Repository

```bash
git clone https://github.com/mani787060/langchain-conditional-chain.git
cd langchain-conditional-chain
```

### Step 2: Create a Virtual Environment

**Windows:**

```bash
python -m venv .venv
.venv\Scripts\activate
```

**macOS/Linux:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### Step 3: Install Dependencies

```bash
python -m pip install --upgrade pip
pip install langchain-core langchain-google-genai pydantic python-dotenv
```

### Step 4: Configure the API Key

Obtain an API key through [Google AI Studio](https://aistudio.google.com/).

Create a `.env` file in the repository root:

```env
GOOGLE_API_KEY=your_gemini_api_key_here
```

Replace the placeholder with your actual API key.

**Security reminder:** Never upload your real API key to GitHub.

Add the following entries to `.gitignore`:

```gitignore
.env
.venv/
__pycache__/
*.pyc
```

### Step 5: Run the Application

```bash
python conditional_chain.py
```

The script prints the generated response and the ASCII representation of the chain graph.

## 💡 Example Use Cases

The same workflow can be adapted to several applications:

- **Customer support:** Route positive and negative customer feedback to different response templates.
- **Product reviews:** Classify reviews and generate suitable replies.
- **Feedback analysis:** Process user comments before sending them to different processing stages.
- **Support ticket routing:** Extend the classifier to categories such as billing, technical issues, and general inquiries.
- **Content moderation workflows:** Route text for additional review based on defined classification rules.

These are potential extensions of the current project, not features already implemented in the repository.

## 🔄 Conditional Chains vs. Sequential Chains

| Sequential chain | Conditional chain |
|---|---|
| Executes connected stages in sequence. | Chooses a processing path based on conditions. |
| Later stages use outputs from earlier stages. | A routing decision determines which branch executes. |
| Useful for report generation followed by summarization. | Useful for sentiment-based response generation. |
| Often follows a linear workflow. | Supports branching and fallback logic. |

## 🌱 Possible Improvements

You can extend this project by:

- Adding a neutral sentiment category.
- Supporting more categories, such as questions, complaints, and suggestions.
- Using structured output with a dedicated schema for routing.
- Adding exception handling for API and parsing errors.
- Logging classification results for analysis.
- Evaluating sentiment classification against a labeled test dataset.
- Building a simple interface for users to submit feedback.
- Adding a human-review path for uncertain classifications.

## 📚 Learning Resources

- [LangChain Documentation](https://python.langchain.com/docs/introduction/)
- [LangChain Expression Language](https://python.langchain.com/docs/concepts/lcel/)
- [Pydantic Documentation](https://docs.pydantic.dev/)
- [Google Gemini API Documentation](https://ai.google.dev/gemini-api/docs)
- [Python Documentation](https://docs.python.org/3/)
