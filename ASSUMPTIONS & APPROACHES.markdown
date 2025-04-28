## Pulse Module Extractor: Approach, Assumptions, and Edge Case Handling

### Overview
The **Pulse Module Extractor** is a Streamlit-based application that crawls help documentation websites, extracts content, and uses Google's `Gemini` API to generate structured JSON output of functional modules and submodules. This document outlines the approach, assumptions, and edge case handling strategies.

### Approach
- **Modular Design**:
  - The application is split into distinct functions: `crawl_website` (web crawling), `extract_meaningful_content` (content extraction), and `generate_modules_with_gemini` (AI processing).
  - This separation enhances maintainability, testing, and potential reuse of components.
- **User Feedback**:
  - Streamlit provides real-time feedback via a progress bar, status messages, and error alerts.
  - Logs are written to both the console and `streamlit_log.txt` for debugging.
- **Polite Crawling**:
  - A 0.5-second delay between requests (`DELAY_BETWEEN_REQUESTS`) prevents overwhelming target servers.
  - A custom User-Agent (`PulseModuleExtractorAI/1.0`) identifies the crawler to site administrators.
- **Structured Output**:
  - Gemini is prompted to return a JSON list of modules with `module`, `Description`, and `Submodules` keys.
  - The JSON format ensures machine-readable output suitable for downstream processing.
- **Error Transparency**:
  - Extensive error handling logs issues (e.g., invalid URLs, failed requests) and displays them in the Streamlit UI.
  - Validation of Gemini's JSON output prevents malformed results from reaching the user.

### Assumptions
- **Website Structure**:
  - Assumes documentation sites use standard HTML with semantic tags (e.g., `<main>`, `<article>`, or identifiable classes like `content`).
  - Expects content to be primarily text-based and accessible without JavaScript rendering.
- **Gemini Capabilities**:
  - Assumes Gemini can accurately identify modules and submodules from clear, well-structured documentation.
  - Relies on Gemini to produce valid JSON output when given a structured prompt.
- **Network Reliability**:
  - Assumes stable internet connectivity for crawling websites and making Gemini API calls.
  - Expects target websites to be accessible without requiring authentication or CAPTCHAs.
- **Single-Threaded Execution**:
  - Assumes sequential crawling (one URL at a time, one page at a time) is sufficient for the use case.
  - Does not implement parallel processing, prioritizing simplicity and server politeness.
- **API Key Availability**:
  - Assumes users have valid Gemini API and Ngrok auth tokens, with sufficient quota for operation.

### Edge Case Handling
- **Invalid URLs**:
  - **Issue**: Users may enter malformed or non-HTTP/HTTPS URLs.
  - **Handling**: The `is_valid_url` function checks for valid schemes (`http`, `https`) and domains using `urlparse`.
  - **Feedback**: Invalid URLs are filtered out, and warnings are displayed in Streamlit (e.g., "Ignoring invalid URL(s): `<url>`").
- **Non-HTML Content**:
  - **Issue**: Crawled pages may return PDFs, images, or other non-HTML content.
  - **Handling**: Checks the `content-type` header; skips non-HTML content (e.g., `application/pdf`).
  - **Feedback**: Logs a warning (`Skipping non-HTML content (<type>) at: <url>`) and continues crawling.
- **Request Failures**:
  - **Issue**: Network issues, timeouts, or server errors may cause HTTP requests to fail.
  - **Handling**: Catches `requests.exceptions.RequestException` (e.g., `Timeout`, `ConnectionError`), marks the URL as visited to avoid retries, and continues with the next URL.
  - **Feedback**: Displays errors in Streamlit (e.g., "Error crawling `<url>`: `<error>`") and logs details.
- **Empty or Noisy Content**:
  - **Issue**: Some pages may lack meaningful content or include excessive UI clutter.
  - **Handling**:
    - Removes non-content elements (`<script>`, `<nav>`, etc.) during extraction.
    - Prioritizes semantic containers; falls back to `<body>` if none found.
    - If extraction yields little text (<100 characters), uses `get_text()` for broader extraction.
    - Stores a placeholder message (`[No meaningful content extracted]`) if extraction fails.
  - **Feedback**: Logs warnings (e.g., "No meaningful content extracted from `<url>`") and informs users if all pages yield no content.
- **Gemini API Failures**:
  - **Issue**: API calls may fail due to quota limits, authentication issues, or unexpected errors.
  - **Handling**:
    - Catches general exceptions during API calls, returning an error message.
    - Validates JSON output; checks for list structure, required keys (`module`, `Description`, `Submodules`), and object types.
  - **Feedback**: Displays detailed errors (e.g., "AI Processing Error: `<error>`") with a preview of the raw response if JSON is invalid.
- **Large Content Volumes**:
  - **Issue**: Crawled content may exceed Gemini's context length limit (~900,000 characters).
  - **Handling**: Truncates the context string to 900,000 characters before sending to Gemini.
  - **Feedback**: Logs a warning (`Context length (<length>) exceeds limit, truncating`).
- **Dynamic Websites**:
  - **Issue**: JavaScript-rendered content may not be accessible to `requests` and `BeautifulSoup`.
  - **Handling**: Limited support; relies on static HTML only (no headless browser).
  - **Feedback**: Informs users if extraction fails due to JavaScript reliance (e.g., "Could not extract any content... Website requires JavaScript").
- **Anti-Scraping Measures**:
  - **Issue**: Sites may block requests with CAPTCHAs, rate limits, or IP bans.
  - **Handling**: Marks blocked URLs as visited to avoid retries; continues with other URLs.
  - **Feedback**: Displays errors (e.g., "Error crawling `<url>`: `<error>`") and suggests possible reasons (e.g., "blocks scraping").
- **Streamlit/Ngrok Failures**:
  - **Issue**: Port conflicts, invalid Ngrok tokens, or Streamlit crashes may prevent the app from launching.
  - **Handling**:
    - Kills existing Ngrok tunnels (`ngrok.kill()`) before starting.
    - Checks for placeholder API keys and aborts if detected.
    - Redirects Streamlit logs to `streamlit_log.txt` for debugging.
  - **Feedback**: Displays errors in Streamlit (e.g., "Failed to start Ngrok tunnel or Streamlit: `<error>`").