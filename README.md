🕵️ nodriver-mcp-custom
Stealth browser automation MCP server, optimized for local LLMs with built-in HAR export.
A lightweight fork of Saber-CC/nodriver-mcp
 that strips 40+ diagnostic tools and adds network traffic capture — designed specifically for local models like Qwen 3 32B/35B, Llama 3, and other tool-calling LLMs running on consumer hardware.
✨ What's Different from the Original
Feature
	
Original
	
This Fork
Tools exposed
	
56–75
	
15 (scraping mode by default)
HAR export
	
❌
	
✅ save_har tool
Default mode
	
Full diagnostic
	
Scraping-optimized
Token overhead
	
High (large tool descriptions)
	
~80% smaller
Best for
	
Debugging, performance traces
	
Web scraping, API capture
🎯 Key Features

    🔇 Scraping Mode by Default — Only 15 essential tools exposed. No noise, no hallucinated tool calls.
    📡 HAR Export — Capture all network requests (XHR, fetch, API calls) during browser interaction and save as standard .har file.
    🛡️ Stealth — Inherits nodriver's anti-detection: no CDP fingerprint, no WebDriver flags.
    ⚡ Local LLM Friendly — Tested with Qwen 3 35B (3-bit quantized) via Ollama, LM Studio, Cherry Studio.
    🔌 Universal MCP Client — Works with any MCP-compatible client (Cherry Studio, LM Studio, Cline, Continue, Open WebUI).

📦 Installation
From GitHub (recommended)

bash
uv tool install "nodriver-mcp @ git+https://github.com/Penchecker/nodriver-mcp-custom.git@main"

From source

bash
git clone https://github.com/Penchecker/nodriver-mcp-custom.git
cd nodriver-mcp-custom
uv sync
uv run nodriver-mcp

Requirements

    Python 3.12+
    Chrome or Chromium installed
    (Optional) Node.js 22+ for performance trace analysis

⚡ Quick Start
Cherry Studio

    Open Settings → MCP
    Click Add Server
    Fill in:

Field
	
Value
Name
	
nodriver
Type
	
stdio
Command
	
/home/YOU/.local/bin/nodriver-mcp
Arguments
	
(leave empty)
Environment
	
See below
Timeout
	
180

    Enable the server in your chat (🔌 icon)
    Send your prompt

LM Studio
Add to ~/.config/lm-studio/mcp_servers.json:

json
{
  "mcpServers": {
    "nodriver": {
      "command": "nodriver-mcp"
    }
  }
}

🛠️ Configuration
All configuration via environment variables:
Variable
	
Default
	
Description
NODRIVER_SCRAPING_MODE
	
true
	
Expose only 15 scraping tools
NODRIVER_FULL_MODE
	
false
	
Set to true to expose all 56+ tools
NODRIVER_BROWSER_PATH
	
auto-detect
	
Path to Chrome/Chromium binary
NODRIVER_HEADLESS
	
false
	
Run browser without visible window
NODRIVER_PROXY
	
system
	
Proxy URL (HTTP/SOCKS)
NODRIVER_USER_DATA_DIR
	
~/.nodriver-mcp/chrome-profile
	
Persistent profile directory
Example: Headless + Custom Chrome Path

NODRIVER_BROWSER_PATH=/usr/bin/chromium
NODRIVER_HEADLESS=true
NODRIVER_SCRAPING_MODE=true

🎯 Scraping Mode: 15 Tools
When NODRIVER_SCRAPING_MODE=true (default), only these tools are exposed:
Tool
	
Purpose
start_browser
	
Launch browser with URL, proxy, persistence
close_browser
	
Shutdown browser and cleanup
new_page
	
Open new tab/page
navigate_page
	
Navigate, reload, back, forward
list_pages
	
List all open tabs
select_page
	
Switch active tab
take_snapshot
	
Get page content as accessibility tree (text)
click
	
Click element by UID
click_at
	
Click at coordinates
fill
	
Fill input field by UID
fill_form
	
Batch fill multiple fields
type_text
	
Type text into focused element
press_key
	
Press keyboard shortcut
wait_for
	
Wait for text to appear
save_har
	
Export network requests to HAR file ✨
📊 HAR Export
The save_har tool captures all network traffic (requests, responses, headers, timing) during browser interaction and saves it in HTTP Archive
 format.
Usage

python
save_har(file_path="/home/user/output.har")

Typical Workflow

1. start_browser(url="https://example.com")
2. take_snapshot()                    # Read page, find elements
3. click(uid="uid=5")                 # Interact (triggers API calls)
4. wait_for(text=["Loaded"])          # Wait for response
5. take_snapshot()                    # Verify new content
6. save_har(file_path="./traffic.har") # Export all network requests
7. close_browser()

Analyzing HAR Files

    Google HAR Analyzer
     — web-based viewer
    HAR Viewer
     — alternative viewer
    Import into Postman, Charles Proxy, or Fiddler

🤖 Example Prompt for Local LLM

You are a web scraping agent. Use MCP tools to:

1. Go to https://github.com/trending/python
2. Take a snapshot and find the first repository
3. Click on it
4. Wait for the page to load
5. Click on "Issues" tab
6. Save HAR to /home/user/github.har
7. Close browser

Call one tool at a time. Wait for each result before proceeding.

Expected Behavior
The model will autonomously:

    Launch browser
    Navigate to GitHub trending
    Extract element UIDs from snapshot
    Click through the site
    Capture all XHR/API calls in HAR
    Save and close

🔄 Switching to Full Mode
If you need all 56+ tools (performance traces, console collection, diagnostics):

bash
NODRIVER_FULL_MODE=true nodriver-mcp

Or in your MCP client config:

NODRIVER_FULL_MODE=true

🐛 Troubleshooting
"Could not find Chrome binary"
Set NODRIVER_BROWSER_PATH:

NODRIVER_BROWSER_PATH=/usr/bin/chromium

Find your browser path:

bash
which google-chrome
which chromium
ls /opt/google/chrome/chrome

Model describes actions instead of calling tools
Add to system prompt:

ALWAYS call tools via function call. Never describe actions in text.
Call one tool at a time and wait for the result.

HAR file is empty
The model didn't trigger network requests. Ensure it:

    Actually clicks/interacts with the page
    Waits for content to load before saving
    Doesn't close browser before save_har

Timeout errors
Increase timeout in your MCP client config to 180 seconds.
📝 License
MIT License — see LICENSE
 file.
This is a fork of Saber-CC/nodriver-mcp
, also MIT licensed.
🙏 Credits

    Original project: Saber-CC/nodriver-mcp
    Browser backend: nodriver
    MCP protocol: Model Context Protocol

📬 Contributing
Issues and PRs welcome! This fork focuses on:

    Scraping optimization
    HAR/network capture
    Local LLM compatibility

For advanced debugging features, use the original repository
.