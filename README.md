# 🤖 AI Web Search Agent

A powerful AI-powered research agent built with LangChain and Google's Gemini that can autonomously search the web, scrape content, and provide comprehensive answers with source citations.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Installation](#installation)
- [Usage](#usage)
- [Benchmarking](#benchmarking)
- [Project Structure](#project-structure)
- [How It Works](#how-it-works)
- [Cost Analysis](#cost-analysis)
- [Future Enhancements](#future-enhancements)
- [Contributing](#contributing)
- [License](#license)

## 🎯 Overview

This project demonstrates how to build an autonomous AI agent that can:
- Search the web using DuckDuckGo (no API key required)
- Scrape and read full webpage content
- Perform multiple searches with different queries
- Analyze and synthesize information from various sources
- Provide comprehensive answers with citations
- Track performance metrics and costs

**Agent Architecture:** ReAct (Reason + Act)

## ✨ Features

### Core Capabilities
- 🔍 **Autonomous Web Search** - Agent decides what to search and when
- 📄 **Full Page Scraping** - Extracts complete webpage content (not just snippets)
- 🧠 **Multi-Step Reasoning** - Performs 3-5 searches per query for thoroughness
- 📊 **Performance Benchmarking** - Track execution time, searches, and costs
- 💾 **Multiple Export Formats** - Save results as TXT, Markdown, or JSON
- 🛡️ **Error Handling** - Graceful fallbacks when websites block scrapers
- 🆓 **Cost-Effective** - Uses free DuckDuckGo API and Gemini free tier

### Advanced Features
- **Batch Processing** - Test multiple queries and compare performance
- **Efficiency Scoring** - 0-10 rating based on speed and thoroughness
- **Cost Estimation** - Track token usage and API costs
- **Flexible Configuration** - Easily swap LLM models or search engines

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        USER QUERY                            │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              GEMINI AI (ReAct Agent)                         │
│                                                              │
│  Think → Act → Observe → Think → Act → ...                  │
│                                                              │
│  1. Understands question                                     │
│  2. Plans search strategy                                    │
│  3. Calls web_search tool (3-5 times)                       │
│  4. Analyzes all results                                     │
│  5. Synthesizes comprehensive answer                         │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                   WEB SEARCH TOOL                            │
│                                                              │
│  1. DuckDuckGo Search (get URLs)                            │
│  2. Visit each webpage                                       │
│  3. Scrape full content (up to 4000 chars/page)            │
│  4. Format and return to agent                              │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│          COMPREHENSIVE ANSWER WITH SOURCES                   │
└─────────────────────────────────────────────────────────────┘
```

## 📦 Installation

### Prerequisites
- Python 3.8+
- Google API Key (for Gemini)

### Step 1: Clone the Repository
```bash
git clone https://github.com/yourusername/web-search-agent.git
cd web-search-agent
```

### Step 2: Create Virtual Environment
```bash
python -m venv .venv

# Windows
.venv\Scripts\activate

# Linux/Mac
source .venv/bin/activate
```

### Step 3: Install Dependencies
```bash
pip install -r requirements.txt
```

### Step 4: Set Up Environment Variables
Create a `.env` file in the project root:
```env
GOOGLE_API_KEY=your_gemini_api_key_here
```

**Get your API key:** https://makersuite.google.com/app/apikey

## 🚀 Usage

### Basic Usage

```python
from langchain_core.messages import HumanMessage

# Ask a question
query = "What are the latest developments in AI?"
result = agent.invoke({'messages': [HumanMessage(content=query)]})

# Get the answer
answer = result['messages'][-1].content[0]['text']
print(answer)
```

### With Benchmarking

```python
# Run with performance tracking and save results
result = run_with_benchmark(
    "What is quantum computing?",
    save_format='md'  # Options: 'txt', 'md', 'json'
)
```

### Batch Testing

```python
# Test multiple queries
test_questions = [
    "What is machine learning?",
    "How does blockchain work?",
    "What are the benefits of cloud computing?"
]

batch_results = batch_benchmark(test_questions, save_results=True)
```

### Export Results

```python
# Save as Markdown
save_as_markdown(query, answer)

# Save as JSON
save_as_json(query, answer)
```

## 📊 Benchmarking

The project includes comprehensive performance tracking:

### Metrics Tracked
- **Execution Time** - Total duration from query to answer
- **Number of Searches** - How many web searches performed
- **LLM Calls** - Number of AI model invocations
- **Cost Estimation** - Approximate API costs
- **Efficiency Score** - 0-10 rating based on speed and thoroughness

### Example Benchmark Report

```
================================================================================
BENCHMARK REPORT
================================================================================
Query: What is quantum computing?
Status: ✅ Success

Performance Metrics:
  • Total Duration: 18.45 seconds
  • Number of Searches: 4
  • Estimated LLM Calls: 7
  • Average Time per Search: 4.61s

Cost Estimation (Gemini Flash - approximate):
  • Estimated Tokens: ~20000 tokens
  • Estimated Cost: $0.00 (within free tier)
  
Efficiency Score: 8.5/10
================================================================================
```

## 📁 Project Structure

```
web-search-agent/
├── web_search_agent.ipynb    # Main Jupyter notebook
├── README.md                  # This file
├── requirements.txt           # Python dependencies
├── .env                       # API keys (not committed)
├── .gitignore                # Git ignore rules
├── Results/                   # Saved answers and benchmarks
│   ├── agent_answer_*.md
│   ├── agent_answer_*.json
│   └── benchmark_*.txt
└── .venv/                     # Virtual environment (not committed)
```

## 🔍 How It Works

### 1. Tool Definition
The `web_search` tool combines DuckDuckGo search with web scraping:

```python
@tool
def web_search(query: str, num_results: int = 3) -> str:
    # 1. Search DuckDuckGo
    results = DDGS().text(query=query, max_results=num_results)
    
    # 2. Scrape each result
    for result in results:
        loader = WebBaseLoader(result['href'])
        docs = loader.load()
        # Process and format content
    
    return formatted_results
```

### 2. Agent Creation
Uses LangChain's `create_agent` with ReAct architecture:

```python
agent = create_agent(
    model=llm,                # Gemini AI
    tools=[web_search],       # Available tools
    system_prompt=prompt      # Instructions
)
```

### 3. ReAct Loop
The agent autonomously:
1. **Thinks** - "I need to search for this topic"
2. **Acts** - Calls `web_search("query")`
3. **Observes** - Reads the results
4. **Thinks** - "I need more specific information"
5. **Acts** - Calls `web_search("more specific query")`
6. **Repeats** - Until sufficient information gathered
7. **Synthesizes** - Combines all findings into comprehensive answer

## 💰 Cost Analysis

### Current Setup (Gemini Flash)
- **Search API:** Free (DuckDuckGo)
- **LLM API:** Free tier (15 requests/min, 1.5M tokens/month)
- **Cost per query:** $0.00 (within free tier)

### At Scale (1000 queries/day)
- **Estimated tokens:** ~20M tokens/month
- **Gemini Flash cost:** ~$0-5/month
- **Total cost:** ~$5/month

### Comparison with Alternatives
| Solution | Cost/Month (1000 queries/day) |
|----------|-------------------------------|
| **This Agent** | $0-5 |
| ChatGPT Plus | $20 (limited to personal use) |
| GPT-4 API | $50-150 |
| Claude API | $30-100 |

## 🔮 Future Enhancements

### Potential Improvements
- [ ] Add local LLM support (Ollama) for complete privacy
- [ ] Implement caching to avoid redundant searches
- [ ] Add support for more search engines (Google, Bing)
- [ ] Create a web interface (Streamlit/Gradio)
- [ ] Add image and video search capabilities
- [ ] Implement conversation memory for follow-up questions
- [ ] Add multi-language support
- [ ] Create Docker containerization
- [ ] Add unit tests and CI/CD pipeline

### Advanced Features
- [ ] RAG integration with vector database
- [ ] Multi-agent collaboration
- [ ] Custom domain-specific agents (legal, medical, etc.)
- [ ] Automated fact-checking and source verification

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

### Development Setup
1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🙏 Acknowledgments

- **LangChain** - Framework for building LLM applications
- **Google Gemini** - Powerful and cost-effective LLM
- **DuckDuckGo** - Privacy-focused free search API
- **Beautiful Soup / WebBaseLoader** - Web scraping capabilities

## 📧 Contact

For questions or feedback, please open an issue on GitHub.

## ⭐ Star History

If you find this project useful, please consider giving it a star!

---

**Built with ❤️ using LangChain and Gemini**

