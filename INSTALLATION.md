# Installation Guide

Complete step-by-step installation instructions for OpenHAB AI Integration.

## Table of Contents

1. [System Requirements](#system-requirements)
2. [OpenHAB Setup](#openhab-setup)
3. [Ollama Installation](#ollama-installation)
4. [OpenWebUI Installation](#openwebui-installation)
5. [Tool Import and Configuration](#tool-import-and-configuration)
6. [First Steps](#first-steps)
7. [Troubleshooting](#troubleshooting)

---

## System Requirements

### Minimum Hardware
- **CPU**: 4 cores (8 recommended for larger models)
- **RAM**: 8GB (16GB+ recommended)
- **Storage**: 20GB free space
- **Network**: Local network access to OpenHAB instance

### Software Requirements
- **OpenHAB**: Version 4.x or 5.x
- **Docker**: Version 20.10+ (for OpenWebUI)
- **Ollama**: Latest version
- **OS**: Linux, macOS, or Windows with WSL2

---

## OpenHAB Setup

### 1. Verify OpenHAB Installation

Ensure OpenHAB is running and accessible:

```bash
# Check if OpenHAB is running
curl http://localhost:8080/rest/systeminfo

# Expected output: JSON with system information
```

### 2. Generate API Token (Recommended)

1. Open OpenHAB UI: `http://your-openhab-ip:8080`
2. Navigate to **Settings** → **API Security**
3. Click **"Create new API token"**
4. Name: `OpenWebUI-Integration`
5. Permissions: **Administrator** (or custom with read/write to items)
6. **Copy the token** - you'll need it later

> ⚠️ **Important**: Store the token securely. It cannot be retrieved again.

### 3. Test API Access

```bash
# Replace YOUR_TOKEN with your actual token
curl -H "Authorization: Bearer YOUR_TOKEN" \
  http://localhost:8080/rest/items

# Should return JSON array of all items
```

---

## Ollama Installation

### Linux / macOS

```bash
# Download and install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Verify installation
ollama --version
```

### Windows

Download from: https://ollama.com/download/windows

### Pull Required Model

```bash
# Recommended: Qwen 2.5 (7B or 14B)
ollama pull qwen3:14b

# Alternative: DeepSeek R1 
ollama pull qwen3:30b

# Lightweight option: Llama 3
ollama pull gpt-oss
```

### Verify Model Installation

```bash
# List installed models
ollama list

# Test the model
ollama run qwen3:14b "Hello, can you help me control my smart home?"
```

---

## OpenWebUI Installation

### Docker Installation (Recommended)

#### Option 1: Standalone OpenWebUI

```bash
docker run -d \
  --name open-webui \
  -p 3000:8080 \
  -v open-webui:/app/backend/data \
  --add-host=host.docker.internal:host-gateway \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

#### Option 2: OpenWebUI with Ollama in Docker

```bash
docker run -d \
  --name open-webui \
  -p 3000:8080 \
  -e OLLAMA_BASE_URL=http://host.docker.internal:11434 \
  -v open-webui:/app/backend/data \
  --add-host=host.docker.internal:host-gateway \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

### Initial Setup

1. Open browser: `http://localhost:3000`
2. **Create admin account**:
   - Email: your@email.com
   - Password: secure password
   - Name: Your Name
3. **Connect Ollama**:
   - Should auto-detect if on same machine
   - Manual: Settings → Connections → Ollama API: `http://localhost:11434`
4. **Select Model**:
   - Click dropdown in chat
   - Choose `qwen3:14b`

### Verify OpenWebUI Works

Start a chat and ask:
```
What can you help me with?
```

If you get a response, OpenWebUI is working correctly.

---

## Tool Import and Configuration

### 1. Download the Tool

```bash
# Clone this repository
git clone https://github.com/yourusername/openhab-ai-integration.git
cd openhab-ai-integration
```

Or download `tool-openhab_smart_home.json` directly from releases.

### 2. Import Tool into OpenWebUI

1. In OpenWebUI, click **your profile icon** (top right)
2. Go to **Settings** → **Tools**
3. Click **"Import Tool"** button
4. **Select file**: `tool-openhab_smart_home.json`
5. Click **"Import"**

### 3. Configure Tool Settings

After import, click the **⚙️ Settings icon** next to "OpenHAB Smart Home Tool":

#### Required Settings

- **OpenHAB Host**: 
  - If same machine: `localhost` or `127.0.0.1`
  - If different machine: `192.168.1.100` (your OpenHAB IP)
  - Docker: `host.docker.internal`
  
- **OpenHAB Port**: 
  - Default: `8080`

- **API Token**:
  - Paste token from earlier step
  - Leave empty if no authentication required

#### Optional Settings

- **Enable Item Filter**: `true` (recommended)
  - Filters out Group items for cleaner results
  
- **Filter Group Items**: `true` (recommended)
  
- **Custom Filtered Types**: 
  - Leave empty or add: `Scene,DateTime`
  
- **Cache TTL**: `3600` (1 hour)
  
- **Max Search Results**: `50`

Click **"Save"** to apply settings.

### 4. Enable the Tool for Chats

1. Start a **new chat**
2. Click the **🔧 Tools** button (bottom of chat input)
3. Enable **"OpenHAB Smart Home Tool"**
4. Tool is now active for this chat

---

## First Steps

### Test 1: Device Discovery

```
User: Show me all controllable devices
```

**Expected Response**:
```
Smart Home Overview

Total: 327 devices
Controllable: 118 devices
Read-only: 209 devices

Controllable devices by type:
- Switch: 37 devices (on, off, toggle)
- Dimmer: 22 devices (on, off, 0-100%)
- Color: 19 devices (on, off, color, brightness)
...
```

### Test 2: Search Functionality

```
User: Find all lights in the bedroom
```

**Expected Response**:
```
Search: bedroom, light (AND mode)
Time: 145ms

Found 3 controllable devices:

[💡] Bedroom Ceiling Light (Dimmer)
   Status: 75% | Controllable: on, off, 0-100%
   
[💡] Bedroom Desk Lamp (Switch)
   Status: ON | Controllable: on, off, toggle
...
```

### Test 3: Device Control

```
User: Turn off the bedroom desk lamp
```

**Expected Response**:
```
[SUCCESS] Bedroom Desk Lamp successfully set to 'OFF'!
```

### Test 4: Power Monitoring

```
User: What's using power right now?
```

**Expected Response**:
```
Current power consumption:
- Office Desk: 4W (monitors in standby)
- Kitchen Refrigerator: 145W (normal operation)
- Living Room TV: 0W (off)
...
```

---

## Troubleshooting

### Issue: "Could not connect to OpenHAB"

**Symptoms**: Tool cannot reach OpenHAB API

**Solutions**:

1. **Verify OpenHAB is running**:
   ```bash
   curl http://localhost:8080/rest/systeminfo
   ```

2. **Check firewall**:
   ```bash
   # Linux
   sudo ufw status
   sudo ufw allow 8080
   
   # Check if port is listening
   netstat -tuln | grep 8080
   ```

3. **Docker network issue**:
   - If OpenWebUI in Docker, use `host.docker.internal` instead of `localhost`
   - Or use host networking: `docker run --network=host ...`

4. **Wrong IP address**:
   ```bash
   # Find OpenHAB machine IP
   ip addr show  # Linux
   ifconfig      # macOS
   ```

### Issue: "No devices found"

**Symptoms**: Query returns 0 devices

**Solutions**:

1. **Disable filters temporarily**:
   - Tool Settings → `Enable Item Filter: false`
   - Save and test again

2. **Check OpenHAB has items**:
   ```bash
   curl http://localhost:8080/rest/items | jq '. | length'
   # Should return number > 0
   ```

3. **API token permissions**:
   - Regenerate token with Administrator role
   - Or test without token (remove from settings)

### Issue: "LLM not using tools"

**Symptoms**: AI responds but doesn't call OpenHAB functions

**Solutions**:

1. **Model doesn't support function calling**:
   - Switch to `qwen3:14b or qwen3:30b` (best support)
   - Avoid older/smaller models

2. **Tool not enabled for chat**:
   - Click 🔧 Tools button
   - Verify "OpenHAB Smart Home Tool" is checked

3. **Query too vague**:
   - Try explicit queries: "Use the OpenHAB tool to list devices"
   - Avoid: "What's in my home?" (too general)

### Issue: "Timeout connecting to OpenHAB"

**Symptoms**: Requests take too long

**Solutions**:

1. **Large item count** (>1000 items):
   - Reduce timeout in tool code: `timeout=30`
   - Enable filters to reduce load

2. **Network latency**:
   - OpenHAB on different network segment
   - Use wired connection instead of WiFi

3. **OpenHAB overloaded**:
   - Check OpenHAB logs: `/var/log/openhab/`
   - Reduce polling frequency in bindings

### Issue: "Tool import failed"

**Symptoms**: Error when importing JSON

**Solutions**:

1. **File corrupted**:
   - Re-download from releases
   - Verify JSON is valid: `jq . tool-openhab_smart_home.json`

2. **OpenWebUI version too old**:
   - Requires 0.3.9+
   - Update: `docker pull ghcr.io/open-webui/open-webui:main`

3. **Browser cache**:
   - Hard refresh: Ctrl+Shift+R
   - Clear browser cache

---

## Advanced Configuration

### Using SSL/HTTPS

If OpenHAB uses HTTPS:

```python
# Modify tool code:
def get_base_url(self) -> str:
    return f"https://{self.host}:{self.port}/rest"
```

### Custom Port

```python
# Tool Settings:
OpenHAB Port: 8443  # Or your custom port
```

### Multiple OpenHAB Instances

Create separate tool imports:
1. Import tool
2. Rename: "OpenHAB Smart Home Tool - House"
3. Configure with first instance
4. Import again
5. Rename: "OpenHAB Smart Home Tool - Cabin"
6. Configure with second instance

---

## Next Steps

Once installed:

1. **Explore vocabulary**: 
   ```
   What keywords can I search for?
   ```

2. **Test automation ideas**:
   ```
   When I say "movie mode", turn off living room lights and close blinds
   ```

3. **Create custom queries**:
   ```
   Show me all devices that consumed power yesterday
   ```

4. **Share feedback**: Open GitHub issue with suggestions!

---

**Installation complete! 🎉**

You now have an AI-powered smart home assistant. Enjoy natural conversations with your home automation!
