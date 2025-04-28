## Pulse Module Extractor: Setup, Usage, Design Rationale, and Limitations

### Overview
The **Pulse Module Extractor** is a Streamlit-based web application that crawls help documentation websites, extracts meaningful content, and uses Google's Generative AI (`Gemini`) to identify and structure functional modules and submodules in JSON format. This document covers setup instructions, usage examples, design rationale, and known limitations.

### Setup Instructions

#### Prerequisites
- **Environment**: Google Colab or a local Python environment with internet access.
- **Dependencies**:
  - Python 3.7+
  - Libraries: `google-generativeai`, `streamlit`, `beautifulsoup4`, `requests`, `pyngrok`
- **API Keys**:
  - Google Gemini API Key (for AI processing).
  - Ngrok Auth Token (for exposing the Streamlit app publicly).

#### Installation Steps
1. **Install Dependencies**:
   In your Google Colab notebook or local terminal, run:
   ```bash
   !pip install -q -U google-generativeai streamlit beautifulsoup4 requests pyngrok
   ```

2. **Configure API Keys**:
   - Replace the placeholders in the code for `HARDCODED_GOOGLE_API_KEY` and `HARDCODED_NGROK_AUTH_TOKEN` with your actual keys.
   - **Security Warning**: Hardcoding keys is insecure. For production, use environment variables or a secrets management system (e.g., Google Colab secrets, `.env` files).

3. **Run the Code**:
   - Copy the provided Python code into a Google Colab notebook or a local `.py` file.
   - Execute the code. It will:
     - Write a `streamlit_app.py` file.
     - Launch a Streamlit server on port 8501.
     - Create an Ngrok tunnel to expose the app publicly.

4. **Access the App**:
   - After execution, a public URL will be displayed (e.g., `https://<ngrok-id>.ngrok.io`).
   - Open this URL in a browser to interact with the Streamlit app.

#### Troubleshooting
- **Gemini API Key Errors**: Ensure the key is valid and has sufficient quota. Check logs in `streamlit_log.txt`.
- **Ngrok Errors**: Verify the auth token and ensure no other Ngrok tunnels are running (`ngrok.kill()` clears existing tunnels).
- **Streamlit Failures**: Check `streamlit_log.txt` for errors related to port conflicts or missing dependencies.

### Usage Examples

#### Example 1: Analyzing a Single Help Documentation Site
1. Open the Streamlit app via the Ngrok URL.
2. In the text area, enter a URL, e.g., `https://support.neo.space/hc/en-us`.
3. Adjust the "Maximum pages to crawl" slider (default: 15).
4. Click "Extract Modules".
5. View the JSON output listing modules and submodules, along with raw text previews and visited URLs.

**Sample Input**:
```
https://support.neo.space/hc/en-us
```

**Sample Output (JSON)**:
```json
[
  {
    "module": "User Management",
    "Description": "Handles user account creation and permissions.",
    "Submodules": {
      "Account Setup": "Guides users through creating a new account.",
      "Permission Settings": "Explains how to configure user roles."
    }
  },
  {
    "module": "Workspace Configuration",
    "Description": "Manages workspace settings and integrations.",
    "Submodules": {}
  }
]
```

#### Example 2: Analyzing Multiple Documentation Sites
1. Enter multiple URLs, one per line, e.g.:
   ```
   https://support.neo.space/hc/en-us
   https://help.zluri.com/
   ```
2. Set the crawl limit to 10 pages per URL.
3. Click "Extract Modules".
4. The app crawls each site, combines the content, and generates a unified JSON output.

**Sample Output**:
- A JSON structure combining modules from both sites, with clear module/submodule descriptions.
- Expandable sections showing raw text (first 5000 characters) and all visited URLs.

### Design Rationale

#### Why Streamlit?
- **User-Friendly Interface**: Streamlit provides a simple way to create interactive web apps with minimal frontend development.
- **Rapid Prototyping**: Ideal for proof-of-concept applications, with built-in components for text input, sliders, and JSON display.
- **Python-Centric**: Aligns with the Python-based backend (`BeautifulSoup`, `requests`, `Gemini` API).

#### Why Google Gemini?
- **Advanced NLP**: Gemini's ability to understand and summarize complex documentation makes it suitable for extracting structured module information.
- **Configurable**: Allows fine-tuning via prompts and generation parameters (e.g., `temperature=0.2` for deterministic output).
- **Scalability**: Supports large context windows (up to ~900,000 characters), accommodating extensive crawled content.

#### Why Ngrok?
- **Public Access**: Ngrok exposes the local Streamlit server to the internet, essential for Colab environments without public IPs.
- **Ease of Use**: Simplifies tunneling compared to manual server setup.

#### Key Design Choices
- **Web Crawling**: Uses `BeautifulSoup` for robust HTML parsing and content extraction, focusing on meaningful containers (e.g., `<main>`, `<article>`).
- **Content Cleaning**: Removes noise (scripts, styles, navigation) to focus on documentation text.
- **Modular Architecture**: Separates crawling, content extraction, and AI processing for maintainability.
- **Error Handling**: Extensive logging and user feedback (via Streamlit) ensure transparency for debugging.

### Known Limitations
- **Hardcoded Keys**:
  - Security risk: Keys are embedded in the code, unsuitable for production or shared environments.
  - Mitigation: Use environment variables or secrets management in production.
- **Web Crawling Constraints**:
  - Limited to HTML content; JavaScript-rendered pages may not be fully crawled.
  - Crawl depth is capped (default: 15 pages) to avoid excessive load on target sites.
  - Sites with anti-scraping measures (e.g., CAPTCHAs, rate limits) may block requests.
- **Gemini API Dependence**:
  - Requires a valid API key with sufficient quota.
  - Output quality depends on Gemini's interpretation, which may miss nuanced modules if documentation is unclear.
  - Context length limit (~900,000 characters) may truncate large sites' content.
- **Content Extraction**:
  - May miss content in unconventional HTML structures (e.g., heavy use of `<div>` without semantic tags).
  - Fallback to `<body>` extraction can include noise if no specific container is found.
- **Performance**:
  - Crawling multiple sites or deep pages can be slow due to request delays (0.5s between requests).
  - Large content volumes may strain Gemini's processing time.