## Pulse Module Extractor: Technical Architecture Description

### Overview
The **Pulse Module Extractor** is a Streamlit-based application that crawls help documentation websites, extracts content, and uses Google's `Gemini` API to generate structured JSON output of functional modules and submodules. This document describes the technical architecture, including components, data flow, and a conceptual diagram.

### Components

#### Web Crawler
- **Library**: `requests` for HTTP requests, `BeautifulSoup` for HTML parsing.
- **Function**: `crawl_website(start_url, max_pages)`
- **Process**:
  - Starts at a given URL, follows links within the same domain and allowed prefixes (e.g., `/hc/`, `/support/`).
  - Extracts content from HTML pages, storing in a `content_map` dictionary.
  - Limits crawling to `max_pages` (default: 15) and respects a 0.5s delay between requests to avoid overwhelming servers.

#### Content Extractor
- **Function**: `extract_meaningful_content(soup, url)`
- **Process**:
  - Removes non-content elements (e.g., `<script>`, `<style>`, `<nav>`).
  - Prioritizes semantic containers (`<main>`, `<article>`, or IDs/classes like `content`, `main-content`).
  - Extracts text from block-level tags (`<h1>`, `<p>`, `<li>`, etc.) with formatting hints (e.g., Markdown headers for headings, list markers for `<li>`).
  - Falls back to `<body>` if no specific container is found, with additional cleaning to reduce noise.

#### AI Processor
- **Library**: `google-generativeai` (`Gemini` API).
- **Function**: `generate_modules_with_gemini(all_text_content)`
- **Process**:
  - Combines crawled content into a single context string, truncated at 900,000 characters to fit Gemini's limits.
  - Sends a structured prompt to Gemini to identify modules and submodules, requesting JSON output.
  - Parses and validates the JSON response, ensuring it is a list of objects with `module`, `Description`, and `Submodules` keys.

#### Streamlit Frontend
- **Library**: `streamlit`
- **Function**: `run_streamlit_app()`
- **Features**:
  - Text area for inputting one or more URLs (one per line).
  - Slider to set the maximum pages to crawl per URL (1–50, default: 15).
  - Progress bar and status messages for user feedback during crawling and AI processing.
  - Displays JSON output, raw text preview (first 5000 characters), and a list of visited URLs in expandable sections.

#### Ngrok Tunnel
- **Library**: `pyngrok`
- **Purpose**: Exposes the Streamlit server (running on port 8501) to the internet via a public URL.
- **Configuration**: Uses a hardcoded Ngrok auth token (insecure for production) to create a tunnel.

### Data Flow
1. **Input**:
   - User enters URLs in the Streamlit UI text area.
2. **Crawling**:
   - The `crawl_website` function processes each URL, following links and extracting content into a `content_map` (URL → formatted text).
3. **Aggregation**:
   - Content from all URLs is combined into a single context string, with page separators and titles.
4. **AI Processing**:
   - The context is sent to Gemini via `generate_modules_with_gemini`, which returns a JSON list of modules and submodules.
5. **Output**:
   - Streamlit renders the JSON output, a preview of the raw extracted text, and a list of visited URLs.

### Architecture Diagram (Conceptual)
```
[User]
   |
   v
[Streamlit UI] <--> [Ngrok Tunnel]
   |
   v
[Crawler: requests, BeautifulSoup]
   |
   v
[Content Extractor]
   |
   v
[AI Processor: Gemini API]
   |
   v
[Output: JSON, Raw Text, URLs]
```

### Key Technical Details
- **Crawling Constraints**:
  - Only follows links within the same domain and specific path prefixes (e.g., `/hc/`, `/support/` for help sites).
  - Skips non-HTML content (e.g., PDFs, images) and URLs with keywords like `/login`, `/pricing`.
  - Uses a `requests.Session` with a custom User-Agent for consistent HTTP requests.
- **Content Extraction**:
  - Prioritizes semantic HTML tags to reduce noise.
  - Applies regex-based text cleaning (`re.sub(r'\s+', ' ', text)`) to normalize whitespace.
  - Preserves structure with Markdown-like prefixes (e.g., `## H1:`, `*` for lists).
- **AI Interaction**:
  - Uses Gemini 1.5 Flash model with a low temperature (0.2) for deterministic output.
  - Validates JSON output to ensure it matches the expected structure.
- **Streamlit**:
  - Runs as a separate process (`streamlit_app.py`) to isolate the UI from the main script.
  - Passes the Gemini API key via an environment variable (`GOOGLE_API_KEY`).
- **Ngrok**:
  - Creates a tunnel to port 8501, enabling public access in Colab environments.
  - Logs are redirected to `streamlit_log.txt` for debugging.