# genai_architecture_implementation_document_processing-
# GCP Automated RAG Pipeline with Document AI & FastMCP 🚀

![Google Cloud](https://img.shields.io/badge/Google_Cloud-4285F4?style=for-the-badge&logo=google-cloud&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white)
![Gemini](https://img.shields.io/badge/Gemini-8E75B2?style=for-the-badge&logo=google-bard&logoColor=white)

A fully automated, event-driven Generative AI document processing and Retrieval-Augmented Generation (RAG) pipeline built natively on Google Cloud. This project leverages **Document AI**, **Vertex AI (Gemini)**, **Cloud SQL (pgvector)**, and the **Model Context Protocol (MCP)** to turn unstructured data into an intelligent, highly searchable knowledge base.

## ⚠️ The Problem

Companies are drowning in unstructured, mixed-format document dumps. 
* 📉 **Operational Bottleneck:** Manually sifting through thousands of files just to isolate scanned hard copies is incredibly slow.
* 🔎 **Search Failures:** Standard enterprise search tools are virtually blind when it comes to scanned images.
* 🤖 **LLM Hallucinations:** Generative models make things up without grounded, heavily filtered data to anchor them.

## 💡 The Solution

This architecture transforms messy, unorganized uploads into a structured, vector-searchable knowledge base. It handles the complete lifecycle from ingestion and intelligent filtering to OCR extraction, vectorization, and conversational AI retrieval.

### Core Architecture Flow

1. 📥 **Event-Driven Ingestion:** Raw files land in Cloud Storage, triggering Eventarc to spin up a Pipeline Orchestrator on Cloud Run.
2. 🧹 **Automated Cleaning & OCR:** Document AI Custom Classifiers filter out irrelevant files (keeping only scanned hard copies), while Document AI Extractors pull the structured text.
3. 🧠 **Vectorization:** Vertex AI generates text embeddings, which are stored securely in Cloud SQL using the `pgvector` extension.
4. 🔌 **GenAI & MCP Integration:** A FastMCP server on Cloud Run acts as the bridge. A GenAI Agent (powered by Gemini) securely calls the MCP tools to perform vector similarity searches and synthesize answers based purely on the processed documents.
5. 📊 **Multi-File Reporting:** For complex queries, a dedicated Cloud Run service generates and exports comprehensive PDF/Markdown summaries directly to Cloud Storage.
6. 🔗 **External API Syncs:** Cloud Scheduler triggers automated fetches from ServiceNow and Confluence, feeding those tickets and wikis straight into the vector database.

---

## 🏗️ Architecture Diagram

*(Include your exported Eraser.io diagram here)*
`![Architecture Diagram](./docs/architecture.png)`

---

## 🛠️ Prerequisites

* A Google Cloud Project with billing enabled.
* [Google Cloud CLI (`gcloud`)](https://cloud.google.com/sdk/docs/install) installed and authenticated.
* Python 3.10+
* APIs enabled:
  * Cloud Run API
  * Cloud Storage API
  * Document AI API
  * Vertex AI API
  * Cloud SQL Admin API
  * Eventarc API
  * Secret Manager API

---

## 🚀 Deployment Guide

### 1. Setup Infrastructure
Create the required Cloud Storage buckets:
```bash
gsutil mb gs://your-project-raw-docs
gsutil mb gs://your-project-processed-docs
gsutil mb gs://your-project-rejected-docs
```

Deploy the Cloud SQL instance with `pgvector` enabled:
*(Use Terraform or the GCP Console to spin up a PostgreSQL 15+ instance and run `CREATE EXTENSION vector;`)*

### 2. Configure Document AI
1. Go to the Document AI Workbench.
2. Train/Deploy a Custom Document Classifier (to identify scanned hard copies).
3. Deploy an Enterprise OCR or Custom Extractor processor.
4. Save the Processor IDs as environment variables for the pipeline deployment.

### 3. Deploy Cloud Run Services

**Deploy the Pipeline Orchestrator:**
```bash
gcloud run deploy pipeline-orchestrator \
  --source ./pipeline-service \
  --region us-central1 \
  --set-env-vars DOCAI_CLASSIFIER_ID=your_id,DOCAI_EXTRACTOR_ID=your_id,DB_HOST=your_db
```

**Deploy the FastMCP Server:**
```bash
gcloud run deploy fastmcp-server \
  --source ./mcp-server \
  --region us-central1 \
  --no-allow-unauthenticated
```

### 4. Setup Eventarc Triggers
Link the raw storage bucket to the pipeline orchestrator:
```bash
gcloud eventarc triggers create doc-upload-trigger \
  --location=us-central1 \
  --destination-run-service=pipeline-orchestrator \
  --event-filters="type=google.cloud.storage.object.v1.finalized" \
  --event-filters="bucket=your-project-raw-docs" \
  --service-account=your-eventarc-sa@your-project.iam.gserviceaccount.com
```

---

## 💻 Usage

1. **Upload Documents:** Drop a mix of PDFs, images, and Word documents into the `raw-docs` bucket.
2. **Observe Pipeline:** Watch the Cloud Run logs as the system categorizes, extracts, and vectorizes the data into Cloud SQL.
3. **Query the System:** Use the GenAI Chatbot interface (pointing to the FastMCP server) to ask questions like:
   * *"Summarize the scanned invoices from Q3."*
   * *"What are the common issues reported in the latest Confluence sync and the scanned field reports?"*
4. **Generate Reports:** Request a bulk summary, and check the output bucket for the generated PDF/Markdown report.

---

## 🤝 Contributing
Contributions are welcome! Please open an issue or submit a Pull Request if you have suggestions for improving the pipeline, adding new MCP tools, or supporting more external integrations.

## 📄 License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
