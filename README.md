# ZenTrades AI - Document Processing Pipelines

This repository contains automated workflows built in [n8n](https://n8n.io/) to process onboarding and demo data for various trade businesses (HVAC, Plumbing, Pest Control, etc.). The pipelines leverage the **Groq API** to analyze text files and generate structured JSON outputs, including account memos and agent specifications.

## 📁 Directory Structure & Storage

For the pipelines to read and write correctly, your local file system must match this structure. All files processed by the "Read/Write Files from Disk" nodes look for these specific folders:

```text
ZenTrades AI/
├── inputs/
│   ├── demo/
│   │   ├── demo_001_arctic_hvac.txt
│   │   └── ... (up to 005)
│   └── onboarding/
│       ├── onboarding_001_arctic_hvac.txt
│       └── ... (up to 005)
├── outputs/
│   ├── accounts/
│   │   ├── ACC-001_memo_v1.json
│   │   ├── ACC-001_memo_v2.json
│   │   └── ... 
│   └── agent_spec/
│       ├── 001_arctic_hvac_agent_spec_v1.json
│       └── ...
🏷️ Naming Conventions
To ensure the JavaScript code nodes properly parse and map the files, please adhere to the following naming conventions:

Input Files: [type]_[id]_[company_name].txt

Example: demo_001_arctic_hvac.txt or onboarding_003_fireguard.txt

Account Output Files: ACC-[id]_memo_[version].json

Example: ACC-001_memo_v1.json

Agent Spec Output Files: [id]_[company_name]_agent_spec_[version].json

Example: 001_arctic_hvac_agent_spec_v1.json

⚙️ Prerequisites & Setup
To reproduce and run this project, you will need:

n8n instance: Running locally (via Docker/npm) or on n8n cloud. Note: Since these use "Read/Write Files from Disk" nodes, a local Docker setup with mounted volumes for /inputs and /outputs is recommended.

Groq API Key: You need an active API key from Groq to handle the LLM HTTP Requests.

Import Workflows: Import the JSON workflow files (if exported from n8n) into your n8n workspace.

Configuring the Groq API Nodes
In both pipelines, the HTTP Request nodes are configured to hit https://api.groq.com/openai/v1/chat/completions. You must configure the authentication in these nodes by adding your Groq API key to the header (e.g., Authorization: Bearer YOUR_API_KEY).

🚀 How to Run the Pipelines
The project consists of two distinct workflows.

Pipeline 1: Two-Stage Sequential Processing (Top Workflow)
Purpose: This pipeline reads a specific file, processes it through the Groq LLM, saves an intermediate output (like a v1 memo), runs a second Groq LLM prompt on that new data, and saves a final output (like a v2 memo or agent spec).

Steps to Run:

Open the workflow in n8n.

Ensure your target input file is correctly placed in the inputs/ directory.

Click the Test Workflow or run the first node (Read/Write Files from Disk) manually.

The pipeline will sequentially execute the JavaScript formatting, hit the Groq API twice, and write two files to the outputs/ directory (Disk1 and Disk2 write nodes).

Pipeline 2: Batch Processing (Bottom Workflow)
Purpose: This pipeline is designed for bulk processing. It reads a batch of 5 input files simultaneously, extracts the text, formats it via JavaScript, sends 5 parallel requests to the Groq API, and writes the 5 resulting JSON files to the disk.

Steps to Run:

Ensure all 5 company files (001 through 005) are located in their respective inputs/ folders.

Open the workflow in n8n.

Click the 'Execute workflow' button at the bottom of the screen (or the trigger node on the far left).

The pipeline will read all 5 items, process them through the HTTP Request2 node using Groq, and use the final Read/Write Files from Disk5 node to save all 5 outputs into the outputs/ folder dynamically based on their file names.

Maintained by Aryaman for the ZenTrades AI Assignment.