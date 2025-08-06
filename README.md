# AI Company Summarizer

This project contains a Python script that automates the process of summarizing companies. It reads a list of company names from a Google Sheet, uses an AI model via the OpenRouter API to generate a concise summary for each based on public information, and writes the results back to a new tab in the same Google Sheet.

This tool is designed to be run in a Google Colab environment for ease of use and setup.

---

## ✨ Features

* **Automated Summarization**: Automatically generates summaries for a list of companies.
* **Google Sheets Integration**: Reads input directly from and writes output back to Google Sheets.
* **Real-time Information**: Utilizes AI models with web-browsing capabilities to provide summaries based on the latest public information.
* **Robust Error Handling**: Implements a retry mechanism (`tenacity`) to handle transient network errors and API rate limits.
* **Secure**: Uses Google Colab Secrets for API key management, avoiding hardcoded credentials.
* **Flexible**: Easily configurable to use different AI models available on OpenRouter.

---

## 🛠️ Setup and Installation

Follow these steps to set up the environment and run the script.

### 1. Google Cloud & Google Sheets API Setup

To allow the script to access your Google Sheet, you need to create a **Service Account** and enable the necessary APIs.

1.  **Create a Google Cloud Project**: Go to the [Google Cloud Console](https://console.cloud.google.com/) and create a new project or select an existing one.
2.  **Enable APIs**: In your project, navigate to **APIs & Services > Library** and enable both the **Google Drive API** and the **Google Sheets API**.
3.  **Create a Service Account**:
    * Go to **IAM & Admin > Service Accounts** and click **"+ CREATE SERVICE ACCOUNT"**.
    * Give it a name (e.g., `company-summarizer-bot`).
    * Grant it the **"Editor"** role to allow it to modify your Google Sheets.
4.  **Generate a JSON Key**:
    * After creating the service account, go to its **Keys** tab.
    * Click **Add Key > Create new key**, select **JSON**, and click **Create**. A JSON file will be downloaded. **Keep this file secure.**
5.  **Share Your Google Sheet**:
    * Open the JSON key file and copy the `client_email` (e.g., `...gserviceaccount.com`).
    * Open your Google Sheet, click **Share**, and paste this email address, giving it **Editor** access.

### 2. Environment Setup (Google Colab)

1.  **Upload JSON Key**: Upload the downloaded `service_account.json` file to a folder in your Google Drive (e.g., a folder named `gcp`).
2.  **Set API Keys**:
    * In your Colab notebook, click the **key icon (🔑)** on the left to open Secrets.
    * Add a new secret with the name `OPENROUTER_API_KEY` and paste your [OpenRouter](https://openrouter.ai/) API key as the value.
3.  **Install Libraries**: The first cell of the notebook will install all required libraries:
    ```bash
    !pip install openai pygsheets google-auth-oauthlib tenacity pandas -q
    ```

---

## 🚀 How to Use

1.  **Open the Notebook**: Open the `ai_company.ipynb` file in Google Colab.
2.  **Mount Google Drive**: Run the cell that mounts your Google Drive to allow access to the JSON key file.
3.  **Configure the Script**: In the cell marked **`# @title 2. Configuration`**, update the following variables:
    * `GOOGLE_SHEET_NAME`: The name of your Google Sheet.
    * `SOURCE_TAB_NAME`: The tab containing your list of companies.
    * `COMPANY_NAME_COLUMN`: The exact header name of the column with company names.
    * `SERVICE_ACCOUNT_FILE`: The full path to your `service_account.json` file in Google Drive (e.g., `'/content/drive/MyDrive/gcp/service_account.json'`).
4.  **Run the Notebook**: Execute the cells in order. The script will print its progress and notify you when it's complete.
5.  **Check the Results**: A new tab (defaulting to "Company Summaries") will be created in your Google Sheet with the generated summaries.

---

## 🧠 Prompt Design and Approach

A key aspect of this project is ensuring the AI provides accurate, relevant, and consistently formatted responses. This was achieved through careful prompt engineering and selecting the right model.

### 1. Prompt Design and How We Ensured Reliable Responses

The core of the AI's instruction is the prompt found within the `get_company_summary` function:

```python
prompt = f"""
Provide a concise, one-paragraph summary (maximum 3 sentences) of what the company '{company_name}' does.
Base the summary on the most current, publicly available information from their official website, LinkedIn page, or recent news articles.
Focus on their core business, main products, or services.
"""
```

This prompt was designed with the following principles to ensure reliability:

* **Specificity and Constraints**: The prompt explicitly requests a `"concise, one-paragraph summary (maximum 3 sentences)"`. This constraint prevents verbose, unhelpful answers and ensures the output is consistently brief and to the point.
* **Information Sourcing**: The instruction `"Base the summary on the most current, publicly available information..."` is crucial. It directs the model to leverage its web-browsing capabilities to find up-to-date information, rather than relying solely on potentially outdated training data. This is vital for accuracy.
* **Focus and Relevance**: The line `"Focus on their core business, main products, or services"` narrows the scope of the summary. It tells the model to ignore irrelevant details (like minor news or stock performance) and concentrate on the fundamental purpose of the company.
* **System Role**: The script also sets a system message: `{"role": "system", "content": "You are an expert business analyst..."}`. This primes the AI to adopt a professional, analytical tone, improving the quality and style of the summaries.
* **Model Selection**: The script is configured to use a model with web-browsing capabilities (like `phind/phind-70b-v1`). This is the **most critical factor** for reliability, as it ensures the information used for summarization is current and not just a hallucination from static training data.
* **Robust Error Handling**: The script uses the **`tenacity`** library to automatically retry failed API calls, handling transient network errors and ensuring that a single failure does not stop the entire process.

### 2. Brief Explanation of Prompt Design and Approach

Our approach was to treat the AI not as a simple text generator, but as an automated analyst. The prompt was designed to be a clear set of instructions: be concise (max 3 sentences), use current public information (from websites, LinkedIn), and focus only on the core business. This direct, constrained approach, combined with a model capable of real-time web search, ensures the responses are consistently relevant, accurate, and reliable.

---

## 💻 Technology Stack

* **Python**: The core language for the script.
* **Google Colab**: The development and execution environment.
* **OpenRouter API**: The gateway to access various large language models.
* **Google Sheets API**: For programmatic access to read and write spreadsheet data.
* **Google Drive API**: Used for accessing the service account key file.
* **Libraries**:
    * `pygsheets`: A user-friendly wrapper for the Google Sheets API.
    * `openai`: The official client for interacting with OpenAI-compatible APIs like OpenRouter.
    * `pandas`: For data manipulation and easy writing to Google Sheets.
    * `tenacity`: For robust API call retries.
