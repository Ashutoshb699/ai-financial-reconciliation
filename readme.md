# AI Finance Controller: 4-Tier Reconciliation Engine

**Author:** Ashutosh Bajpai  
**Built for:** Razorpay AI Builder Hackathon

##  The Problem
For every transaction processed by a payment gateway, merchants must reconcile the gateway's settlement report against their own internal ERP ledgers. Because these are two independent systems, they frequently fall out of sync due to missing ID prefixes, customer name typos, hidden international processing fees, and tax deductions. Finance teams currently spend hundreds of manual hours auditing these discrepancies.

## The Solution
This project is an automated, on-premise B2B reconciliation engine that bridges the gap between Internal ERPs and Gateway Settlements. It processes financial data through a strict **4-Tier Pipeline**, isolating anomalies and using a Batched AI Agent to resolve complex fee variances, drastically reducing manual review time.

### The 4-Tier Architecture
1. **Tier 1 (Deterministic SQL):** instantly clears perfect matches (exact Order ID + Amount) using highly optimized parameterized SQL queries.
2. **Tier 2 (Algorithmic Fuzzy Matching):** Utilizes `RapidFuzz` (`token_set_ratio` and `partial_ratio`) to catch prefix mismatching and typographical errors in customer names, executing locally without AI overhead.
3. **Tier 3 (AI Anomaly Agent):** Forwards severe discrepancies (e.g., unexplained 3.5% fee variances) to a Google Gemini LLM. The AI acts as a financial auditor, classifying the variance and returning a confidence score via structured JSON.
4. **Tier 4 (Human Intervention Queue):** Unresolvable ghost transactions and anomalies that the AI rejects are isolated in a native UI queue for manual approval/rejection.

## 🛠️ Tech Stack & Engineering Decisions
**Tech Stack**
**Backend:** Python, Flask

**Database:** SQLite (instance/reconciliation.sqlite) with sqlite3.Row factory mapping

**AI Engine:** Google GenAI SDK (gemini-1.5-flash / compatible model) with JSON schema enforcement

**Matching Algorithms:** RapidFuzz

**Frontend:** Bootstrap 5, Jinja2 Templating, Custom CSS Dashboards

## ⚡ Key Features (Designed for Scale)
* **LLM Batch Processing:** To bypass strict API rate limits (15 RPM on free tiers) and optimize execution speed, Tier 3 bundles all unmatched transactions into a single JSON array payload, resolving dozens of records in a single API call.
* **Graceful Degradation:** If the Gemini API key is missing, hits a rate limit, or times out, the system does not crash. It seamlessly falls back to a deterministic simulation or pushes records safely to the Tier 4 manual queue.
* **Auditability:** Every reconciled row stores a confidence score ($0.0$ to $1.0$) and a plaintext AI reasoning trail in the database for financial compliance.

## ⚙️ Local Setup Instructions

Local Setup Guide
Follow these steps to clone, configure, and run the project locally on your machine.

1. Clone the Repository
Open your terminal and clone the repository to your local system:

git clone https://github.com/your-username/razorpay-buildathon.git
cd razorpay-buildathon

2. Create and Activate a Virtual Environment
It is recommended to run the app inside an isolated Python virtual environment:

**Windows**
python -m venv venv
venv\Scripts\activate

**macOS / Linux**
python3 -m venv venv
source venv/bin/activate

3. Install Dependencies
Install all required Python packages:
pip install -r requirements.txt
(If a requirements.txt is missing, ensure you have installed: flask, google-genai, rapidfuzz)

4. Configure Environment Variables
Create a .env file in the root directory of the project to securely store your API credentials.

Create a file named .env in the project root.

Add your Google Gemini API key using the exact key-value pair below:

GEMINI_API_KEY=your_actual_gemini_api_key_here

5. Initialize the Database

Ensure the instance folder exists in your project root. The application will automatically create and seed the SQLite database (reconciliation.sqlite) upon running the initialization scripts or triggering the pipeline from the UI.

6. Run the Application

Start the Flask development server:
python app.py

Open your web browser and navigate to:
http://127.0.0.1:5000
