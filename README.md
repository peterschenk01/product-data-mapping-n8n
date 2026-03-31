# Product Data Mapping Workflow (n8n)

This project provides an **n8n-based workflow** for extracting product data from **Excel and PDF files** and transforming it into a structured **ERP-compatible Excel format (`.xlsx`)**.

It includes:

* A **local web interface** for uploading test files
* A **Docker-based environment** for running all services locally
* An exported **n8n workflow** ready for import and customization

---

## Architecture Overview

The system consists of the following services:

| Service | URL                                              | Description            |
| ------- | ------------------------------------------------ | ---------------------- |
| n8n     | [http://localhost:5678](http://localhost:5678)   | Workflow orchestration |
| webapp  | [http://localhost:8000](http://localhost:8000)   | File upload interface  |
| ollama  | [http://localhost:11434](http://localhost:11434) | Local LLM (optional)   |

---

## Requirements

Ensure the following tools are installed:

* [Docker](https://www.docker.com/) 
* [Docker Compose](https://docs.docker.com/compose/)

---

## Quickstart

### 1. Configure Environment Variables

Create a local environment file:

```bash
cp .env.example .env
```

Update `.env` with values appropriate for your setup.

---

### 2. Start the Services

Run the default stack:

```bash
docker compose up --build -d
```

Stop the services:

```bash
docker compose stop
```

---

### 3. (Optional) Enable Ollama

To use a local language model:

```bash
docker compose --profile full up --build -d
docker exec -it ollama ollama pull llama3.2:3b
```

Notes:

* Initial model download may take time
* Requires ~10 GB of disk space

---

### 4. Initialize n8n

1. Open: [http://localhost:5678](http://localhost:5678)

2. Complete the initial setup

3. Import the workflow:

   ```
   n8n/customer-contact_n8n_workflow.json
   ```

4. Configure required credentials

5. Activate the workflow

#### Ollama Configuration

If using the local Ollama service, set the model base URL to:

```
http://ollama:11434
```

If not using Ollama, replace the LLM node with another supported provider.

---

### 5. Test the Setup

Open the web interface:

```
http://localhost:8000
```

You can:

* Upload test files (PDF or Excel)
* Trigger the webhook workflow
* Download the processed ERP-formatted Excel output

---

## Known Limitations

### General

* No authentication is enforced for webhook endpoints
* The included `llama3.2:3b` model is not reliable for structured extraction
* Multi-product and variant detection is not yet automated

### PDF Processing

* Uses basic text extraction instead of a dedicated parser
* OCR-based PDFs are not handled robustly
* Structured extraction from complex layouts is limited

---

## Recommended Improvements

* Integrate a proper PDF parsing library (e.g., PDFPlumber, Tesseract OCR)
* Upgrade to a more capable LLM for structured output
* Implement product/variant detection logic
* Add authentication to webhook endpoints
* Introduce automated tests

---

## Model Choice: `llama3.2:3b`

The workflow uses the `llama3.2:3b` model via Ollama for local inference.

### Why this model was chosen

* **Local execution (data privacy)**
  All processing runs locally without sending potentially sensitive product data to external APIs.

* **Lightweight and fast**
  The 3B parameter size allows the model to run efficiently on standard development machines with reasonable latency.

* **Easy integration with Ollama**
  Ollama provides a simple API and seamless Docker integration, making it well-suited for prototyping within an n8n workflow.

* **Good enough for prototyping**
  The model can extract basic structured information from semi-structured inputs (Excel, simple PDFs), which is sufficient for demonstrating the workflow.

### Limitations

* **Limited accuracy for complex extraction**
  The model struggles with:

  * Highly unstructured PDFs
  * Multi-product documents
  * Precise structured output

* **Inconsistent formatting**
  Output may require post-processing to ensure reliable field mapping.

### Recommended alternatives (for production)

For more reliable results, consider:

* Larger local models (e.g. `llama3.1:8b`, `mixtral`)
* Hosted models with stronger structured output capabilities (e.g. GPT-4-class models)