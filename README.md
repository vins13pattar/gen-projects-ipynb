# gen-projects-ipynb

Three hands-on Generative AI notebooks (MicroDegree course projects). Each one turns messy text into **validated, structured JSON** using an LLM, then **measures** the quality of the output instead of eyeballing it.

Common techniques across all three:

- **Pydantic schemas** to force and validate structured output
- **Provider switch**: OpenAI or Anthropic via one `call_llm()` helper
- **Prompt evolution**: naive → rules → few-shot → chain-of-thought / automated iteration
- **Evaluation in code** against a golden dataset or checklist

| Notebook | What it does |
|---|---|
| [automated-test-case-generator.ipynb](automated-test-case-generator.ipynb) | Generates QA test cases from a user story |
| [meeting-summerizer.ipynb](meeting-summerizer.ipynb) | Extracts action items, decisions and blockers from a meeting transcript |
| [IT_Support_Ticket_Triage_&_Auto_Response_Bot (1).ipynb](<IT_Support_Ticket_Triage_&_Auto_Response_Bot (1).ipynb>) | Triages IT tickets and drafts first responses |

---

## 1. Automated Test Case Generator

Takes a user story with acceptance criteria (sample: **US-214**, login with mobile number + OTP) and produces a structured test suite.

- `TestCase` / `TestSuite` schemas (ID pattern `TC-001`, type: positive / negative / boundary / edge, priority, steps, expected result)
- Prompt progression: zero-shot → domain rules (equivalence partitioning, boundary value analysis, negative tests) → few-shot example
- **Coverage measured in code**: acceptance criteria covered, test types covered, and a `MUST_TEST` checklist
- **Fill the gaps**: code finds what is missing and asks the model to write only those tests
- **Export** to CSV and Excel (`US-214_test_cases.csv` / `.xlsx`), ready for Jira or Excel

## 2. Sprint & Meeting Intelligence Summarizer

Summarizes a Sprint 14 planning transcript that has six deliberate "traps" (reassigned tasks, changed decisions, tentative ideas, unowned work, etc.).

- `MeetingSummary` schema: executive summary, action items (task / owner / due), decisions, blockers, open questions
- Prompt progression: naive → one rule per trap → chain-of-thought (`working_notes`) → few-shot
- `trap_report()` and `score()` compare output with a `GOLD` answer key
- Token counting (`tiktoken`) and **chunk + map-reduce** for transcripts longer than the context window
- `check_owners()` fact-checks that every owner is a real attendee; output is formatted as Slack-ready markdown

## 3. IT Support Ticket Triage & Auto Response Bot

Classifies helpdesk tickets and drafts the first reply.

- `TicketTriage` schema: category (Access / Network / Software / Hardware), priority (P1–P3), summary, requester, confidence
- `ResponseDraft` schema: subject, body, next steps, `escalate_to_l2`
- 12+ ticket **golden dataset** and a P1/P2/P3 priority rubric
- **Validate-and-retry**: validation errors are fed back to the model
- **Temperature lab**: runs the same ticket 5 times to measure consistency
- Batch evaluation: category/priority accuracy and a prediction crosstab
- Built-in **mock LLM**, so the notebook runs without API keys

---

## How to use

### 1. Setup

```bash
git clone https://github.com/vins13pattar/gen-projects-ipynb.git
cd gen-projects-ipynb
python3 -m venv .venv && source .venv/bin/activate
pip install jupyter openai anthropic pydantic python-dotenv pandas openpyxl tiktoken
```

(Each notebook also has a `%pip install` cell at the top that installs what it needs.)

### 2. Configure API keys

```bash
cp .env.example .env
```

Edit `.env`:

```
PROVIDER=openai            # or: anthropic
OPENAI_API_KEY=sk-...
OPENAI_MODEL=gpt-5.1
ANTHROPIC_API_KEY=sk-ant-...
ANTHROPIC_MODEL=claude-haiku-4-5-20251001
```

`.env` is git-ignored. Never commit it. Only the key for the provider you pick is needed.

### 3. Run

Open a notebook in VS Code or Jupyter (`jupyter lab`), choose the `.venv` kernel, and **Run All**. Cells are ordered as a lesson, so run them top to bottom.

- **Google Colab**: the ticket triage notebook reads keys from Colab *Secrets* (`OPENAI_API_KEY`, `ANTHROPIC_API_KEY`).
- **No key?** The ticket triage notebook falls back to a mock LLM so you can still follow along.
- To try your own input, replace `STORY` (test generator), `TRANSCRIPT` / `ATTENDEES` (meeting summarizer) or `TICKETS` (triage bot).

### Outputs

The test case generator writes `US-214_test_cases.csv` and `.xlsx` into the working folder. These are git-ignored; regenerate them by running the notebook.

## Repo layout

```
.
├── automated-test-case-generator.ipynb
├── meeting-summerizer.ipynb
├── IT_Support_Ticket_Triage_&_Auto_Response_Bot (1).ipynb
├── .env.example
├── .gitignore
└── README.md
```
