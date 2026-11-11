# 🔬 ResearchMind — Multi-Agent Autonomous Research & Fact-Critique System

An end-to-end, multi-agent AI research pipeline architected with **LangChain, LangGraph, and Streamlit**. ResearchMind automates intelligence gathering, deep content extraction, document generation, and peer-review evaluation by coordinating specialized agents in a deterministic orchestration DAG.

---

## 📌 Table of Contents

1. [Overview & Architecture](#-overview--architecture)
2. [Agent Breakdown & Technical Flow](#-agent-breakdown--technical-flow)
3. [Tech Stack & Dependencies](#-tech-stack--dependencies)
4. [Directory Blueprint](#-directory-blueprint)
5. [Installation & Virtual Environment](#-installation--virtual-environment)
6. [Configuration & Environment Variables](#-configuration--environment-variables)
7. [Running the Application](#-running-the-application)
8. [Pipeline State Management](#-pipeline-state-management)
9. [Troubleshooting & FAQs](#-troubleshooting--faqs)
10. [License & Acknowledgments](#-license--acknowledgments)

---

## 🚀 Overview & Architecture

ResearchMind resolves research fragmentation by decoupling the research process into focused functional nodes.

Instead of relying on a single prompt to search, read, synthesize, and format, the system distributes cognitive tasks across four discrete modules executing sequentially.

### System Architecture

```text
                    ┌────────────────────────┐
                    │    User Topic Query    │
                    └───────────┬────────────┘
                                │
                                ▼
                 ┌────────────────────────────────┐
                 │      Stage 01: Search Agent    │
                 │  • Queries Tavily Search API   │
                 │  • Aggregates Top 5 Snippets   │
                 └───────────────┬────────────────┘
                                 │
                                 ▼
                 ┌────────────────────────────────┐
                 │      Stage 02: Reader Agent    │
                 │  • URL Selection & Validation  │
                 │  • BeautifulSoup DOM Scraping  │
                 └───────────────┬────────────────┘
                                 │
                                 ▼
                 ┌────────────────────────────────┐
                 │      Stage 03: Writer Chain    │
                 │  • Context Synthesizer         │
                 │  • Standard Markdown Report    │
                 └───────────────┬────────────────┘
                                 │
                                 ▼
                 ┌────────────────────────────────┐
                 │      Stage 04: Critic Chain    │
                 │  • Fact & Structure Audit      │
                 │  • Score (X/10) & Audit Trail  │
                 └───────────────┬────────────────┘
                                 │
                                 ▼
                  ┌──────────────────────────────┐
                  │ Streamlit UI / Direct Export │
                  └──────────────────────────────┘
```

### Core Workflow

- **Search:** Finds relevant information using Tavily Search API.
- **Read:** Extracts useful webpage content and removes HTML noise.
- **Write:** Synthesizes collected information into a structured research report.
- **Critique:** Reviews the generated report for factual grounding, structure, and readability.
- **Export:** Presents the final output through Streamlit with Markdown download support.

---

## 🤖 Agent Breakdown & Technical Flow

### 1. Stage 01: Search Agent (`agents.py` + `tools.py`)

**Role:** Primary discovery and information indexing.

**Mechanism:**
- Wraps `tavily-python` in a LangChain `@tool` decorator (`web_search`).
- Queries search indexes for authoritative domains.
- Extracts titles, canonical URLs, and contextual snippets.
- Limits snippets to approximately 300 characters each.

**Output:** Formatted string containing top-ranked metadata pairs.

### 2. Stage 02: Reader Agent (`agents.py` + `tools.py`)

**Role:** Deep text extraction and noise reduction.

**Mechanism:**
- Takes candidate URLs identified in Stage 1.
- Executes `scrape_url` using Python Requests.
- Uses custom browser headers (`User-Agent: Mozilla/5.0`).
- Implements 8-second network timeouts.

**DOM Cleansing:**

BeautifulSoup parses the HTML tree and removes unnecessary elements:

- `<script>`
- `<style>`
- `<nav>`
- `<footer>`

The system extracts approximately 3,000 clean text characters to control token consumption.

### 3. Stage 03: Writer Chain (`agents.py`)

**Role:** Analytical synthesis and structural composition.

**Mechanism:**

Operates as a declarative LangChain Expression Language (LCEL) chain:

```python
writer_prompt | LLM | StrOutputParser()
```

**Document Schema:**

1. Executive Introduction
2. Key Findings (minimum three structured, high-signal data points)
3. Analytical Conclusion
4. Explicit Sources (cited canonical URLs)

### 4. Stage 04: Critic Chain (`agents.py`)

**Role:** Automated quality gatekeeper and adversarial reviewer.

**Mechanism:**

Evaluates generated output independently for:

- Factual grounding
- Structure adherence
- Readability

**Output Schema:**

- Score: X/10
- Strengths
- Areas to Improve
- Single-line Summary Verdict

---

## 🛠️ Tech Stack & Dependencies

| Layer | Component / Library | Purpose |
|---|---|---|
| Core Framework | LangChain (>=0.3.0), LangChain Core | Declarative chaining, agents, prompts, output parsers |
| Agent Graph | LangGraph | Multi-agent execution runtime |
| Model Engines | Google GenAI, Groq, OpenAI | Swappable LLM backends |
| Web Retrieval | Tavily Python | AI-optimized web search |
| HTML Parser | BeautifulSoup4, Requests | HTML parsing and content extraction |
| Frontend UI | Streamlit (>=1.38.0) | Interactive dashboard |
| CLI Formatting | Rich | Terminal logging and execution headers |
| Environment | Python-dotenv | Credential management |
| Package Engine | uv / pip | Dependency resolution and virtual environments |

---

## 📁 Directory Blueprint

```text
Multi_agent_research_system/
│
├── app.py                  # Streamlit application
├── pipeline.py             # CLI pipeline orchestrator
├── agents.py               # Agent factories, prompts, and LCEL chains
├── tools.py                # Tavily search and webpage scraper
│
├── requirements.txt        # Project dependencies
├── .env                    # Secret credentials (excluded from Git)
├── .env.example            # Environment variable template
├── .gitignore              # Git ignore configuration
│
└── README.md               # Project documentation
```

---

## ⚙️ Installation & Virtual Environment

Ensure you have **Python 3.12+** installed on your workstation.

### Option 1: Using uv (Recommended)

`uv` provides fast environment setup and dependency resolution.

**Step 1: Create a virtual environment**

```bash
uv venv
```

**Step 2: Activate the environment**

Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

macOS/Linux:

```bash
source .venv/bin/activate
```

**Step 3: Install dependencies**

```bash
uv pip install -r requirements.txt
```

### Option 2: Using Standard pip

**Step 1: Create a virtual environment**

```bash
python -m venv .venv
```

**Step 2: Activate the environment**

Windows:

```powershell
.venv\Scripts\activate
```

Linux/macOS:

```bash
source .venv/bin/activate
```

**Step 3: Upgrade pip**

```bash
pip install --upgrade pip
```

**Step 4: Install dependencies**

```bash
pip install -r requirements.txt
```

---

## 🔑 Configuration & Environment Variables

Create a `.env` file in the root directory of the project.

```env
# Web Search Configuration (Required)
TAVILY_API_KEY=your_tavily_api_key

# LLM Engine Options
GOOGLE_API_KEY=your_google_api_key
GROQ_API_KEY=your_groq_api_key
OPENAI_API_KEY=your_openai_api_key
```

### Getting API Keys

| Provider | Purpose | Official Website |
|---|---|---|
| Tavily | Web Search API | https://tavily.com/ |
| Google AI Studio | Gemini Models | https://aistudio.google.com/ |
| Groq | LLaMA Inference | https://console.groq.com/ |
| OpenAI | GPT Models | https://platform.openai.com/ |

### Switching LLM Providers in `agents.py`

The project supports different LLM backends. Configure the provider that matches your API key.

#### Option A: Google Gemini

```python
from langchain_google_genai import ChatGoogleGenerativeAI
import os

llm = ChatGoogleGenerativeAI(
    model="gemini-2.0-flash",
    temperature=0,
    google_api_key=os.getenv("GOOGLE_API_KEY")
)
```

#### Option B: Groq LLaMA

```python
from langchain_groq import ChatGroq
import os

llm = ChatGroq(
    model="llama-3.1-8b-instant",
    temperature=0,
    groq_api_key=os.getenv("GROQ_API_KEY")
)
```

#### Option C: OpenAI

```python
from langchain_openai import ChatOpenAI
import os

llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0,
    api_key=os.getenv("OPENAI_API_KEY")
)
```

**Security Note:** Never upload your `.env` file or expose API keys in public repositories.

---

## 🖥️ Running the Application

### 1. Interactive Streamlit Dashboard

The web application provides:

- Interactive research topic input.
- Real-time pipeline status tracking.
- Collapsible intermediate execution steps.
- Final Markdown research report.
- Instant report download.

Run:

```bash
uv run streamlit run app.py
```

Open your browser:

```text
http://localhost:8501
```

Enter your research topic into the command card and click:

**⚡ Run Research Pipeline**

### 2. Standalone Command-Line Interface (CLI)

For lightweight automation, terminal scripting, or CI environments:

```bash
uv run pipeline.py
```

Enter your topic when prompted:

```text
Enter a research topic: <topic>
```

The CLI displays:

- Real-time execution steps.
- Search payloads.
- Parsed webpage content.
- Generated research report.
- Critic feedback.

---

## 📊 Pipeline State Management

The execution workflow uses an in-memory dictionary in `pipeline.py` and persistent session states through `st.session_state` in `app.py`.

### State Structure

```python
state = {
    "search_results": str,   # Aggregated titles, snippets, and source URLs
    "scraped_content": str,   # Raw cleaned text from target page DOM
    "report": str,            # Synthesized Markdown research document
    "feedback": str           # Critic audit scores and critique notes
}
```

This decoupled state structure allows individual components to be unit-tested or swapped for custom nodes without breaking dependent steps.

---

## ❓ Troubleshooting & FAQs

### Q1: `groq.BadRequestError: Error code: 400 - model decommissioned`

**Root Cause:** Older LLaMA models are periodically deprecated by providers.

**Resolution:** Update the model identifier in `agents.py` to an active supported model, such as:

```python
model="llama-3.1-8b-instant"
```

### Q2: `GoogleModelNotFoundError: 404 NOT_FOUND`

**Root Cause:** SDK routing mismatch between Google API endpoints or outdated model identifiers.

**Resolution:** Verify the current model name and SDK compatibility. The project configuration references:

```python
model="gemini-2.0-flash"
```

### Q3: `requests.exceptions.HTTPError: 403 Forbidden`

**Root Cause:** Some enterprise websites block automated requests.

**Resolution:**

The scraper uses browser-like headers:

```python
headers = {
    "User-Agent": "Mozilla/5.0"
}
```

If blocking persists, the scraper catches the exception through its `try-except` handling and returns an error string to preserve pipeline execution.

### Q4: How do I prevent API keys from leaking?

**Resolution:**

Ensure your `.gitignore` contains:

```gitignore
.env
.venv/
__pycache__/
*.pyc
```

Check ignored files:

```bash
git status --ignored
```

Never commit real API keys to GitHub.

---

## 📄 License & Acknowledgments

This project is licensed under the **MIT License** — you are free to modify, distribute, and integrate this system in private or commercial software.

### Acknowledgments

- Architecture inspired by multi-agent cooperative workflows in LangChain and LangGraph.
- Search engine integration powered by Tavily AI.
- Frontend dashboard powered by Streamlit.
- Language model integrations powered by Google Gemini, Groq, and OpenAI.
- Typography and interface styling using Syne, DM Sans, and DM Mono.

---

**Built with Python, LangChain, LangGraph, and Streamlit.**

*ResearchMind — Turning research into structured intelligence.*