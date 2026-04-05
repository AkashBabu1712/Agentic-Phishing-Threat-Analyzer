
# Project: Agentic Phishing & Threat Analyzer
---

Instead of relying on static blocklists, this system acts like an autonomous Level 1 Security Operations Center (SOC) analyst. It ingests a suspicious message, investigates the components, and outputs a structured, evidence-based threat report.

## Core Architecture
---

* **Phase 1: Ingestion & Entity Extraction**
    * Build a Python pipeline to ingest raw text payloads, such as `.eml` files (email headers and bodies) or SMS text dumps.
    * Use traditional data parsing to clean the text and extract key indicators of compromise (IOCs) like URLs, IP addresses, and claimed brand identities.
* **Phase 2: Agentic Context Retrieval**
    * This is where the project elevates from a basic script to an advanced system. Implement a tool-use pipeline using frameworks like LangChain or CrewAI.
    * If a URL is detected in Phase 1, the agent autonomously triggers a script to safely check the domain registration age, track URL redirection chains, or fetch a snapshot of the destination webpage.
* **Phase 3: Reasoning & Explanation Generation**
    * Pass the original message *plus* the retrieved external context into a strong reasoning model.
    * Prompt the model to evaluate the threat using a Chain-of-Thought approach: looking for urgency cues, domain spoofing, and mismatched sender intents.
    * The final output must be a strictly typed JSON object containing a risk score (0-100), threat categories (e.g., Credential Harvesting, Malware), and a human-readable explanation of *why* it was flagged.

## Recommended Tech Stack
---

* **Backend & Orchestration:** Python, FastAPI (for exposing the analyzer as an endpoint), and LangChain.
* **The AI Engine:** To show real technical depth, avoid simply wrapping a standard proprietary API. Implement an open-weight reasoning model locally or via cloud endpoints. Models that excel in complex logic—like the NVIDIA Nemotron architectures often used in advanced Kaggle reasoning competitions, or recent Llama 3 and DeepSeek variants—are incredibly effective at parsing adversarial text.
* **Evaluation Baseline:** Use pandas and scikit-learn to handle batch testing. Calculate baseline metrics (Accuracy, F1-Score) against a known public phishing dataset to empirically prove the tool's efficacy.

## GitHub Portfolio Strategy
---

To ensure this repository signals immediate job-readiness to cybersecurity hiring managers, structure it meticulously:

1.  **Architecture Diagram:** Include a clear visual flowchart in the `README.md` showing how the data moves from ingestion to the agentic retrieval and finally to the LLM reasoning engine.
2.  **Benchmark Metrics:** Dedicate a section of the documentation to your statistical evaluation. Show how your agentic LLM approach compares against traditional machine learning baselines (like Random Forest or Naive Bayes) on the same dataset. 
3.  **Containerization:** Include a `Dockerfile` and `docker-compose.yml` so an engineering manager can spin up your analyzer locally with a single command.

---

## 1. The Repository Architecture
Setting up a clean, modular directory structure from day one signals that you write production-ready code.

```text
phishing-agent-analyzer/
├── data/
│   ├── raw_samples/         # .eml files and SMS text dumps
│   └── evaluation_set.csv   # Ground-truth dataset for testing
├── parsers/
│   ├── email_parser.py      # Extracts headers, body, and attachments
│   └── sms_parser.py        # Normalizes text and extracts URLs
├── agents/
│   ├── tools.py             # Functions: url_unshortener, domain_age_checker
│   └── threat_agent.py      # The core LangChain/CrewAI logic loop
├── evaluation/
│   └── metrics.py           # Calculates F1, Precision, Recall
├── main.py                  # FastAPI application entry point
├── requirements.txt
└── README.md
```

## 2. The Unified Data Model
Emails and SMS messages look entirely different, so the first step is normalizing them into a single schema before handing them to the LLM. Using `Pydantic` in Python is the industry standard for this.

```python
from pydantic import BaseModel
from typing import List, Optional

class ExtractedIOCs(BaseModel):
    urls: List[str]
    ip_addresses: List[str]
    emails: List[str]

class NormalizedPayload(BaseModel):
    source_type: str  # "email" or "sms"
    sender_info: str  # Email address or phone number
    raw_text: str     # The actual message body
    iocs: ExtractedIOCs
    urgency_flags: Optional[List[str]] # e.g., "urgent", "account suspended"
```

## 3. The Agentic Routing Logic
Once the data is parsed into that schema, the agent takes over. Because you are handling two different modalities, your agent needs to route its tool-use dynamically:

* **If `source_type == "email"`:** The agent should heavily scrutinize the `sender_info` against the claimed brand in the `raw_text` (e.g., the email says "PayPal" but the sender is "admin@secure-pay-update.com").
* **If `source_type == "sms"`:** SMS phishing (Smishing) relies on shortened URLs. The agent must immediately trigger a `url_unshortener` tool to expand the link and check the final destination's domain age before making a judgment.

### 4. The Statistical Baseline (The Differentiator)
The biggest trap with GenAI projects is building something that looks cool but fails under scrutiny. To bridge the gap into a senior-level portfolio piece, apply standard statistical rigor. 

Grab a standard phishing dataset (there are plenty of great tabular datasets for this on Kaggle). Before you finalize your agent, write a script in `evaluation/metrics.py` to run 100 samples through your system and calculate the exact accuracy, precision, and F1-score. Proving empirically that your LLM-driven approach catches zero-day threats better than a standard machine learning classifier is what gets you hired.

***
