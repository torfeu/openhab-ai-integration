# Quick Start Guide

## Prerequisites Check

Before starting, verify:

1. **OpenHAB is running and accessible**:
```bash
curl -X GET http://your-openhab-ip:8080/rest/items
```
Should return JSON with your items.

2. **OpenHAB version**: 4.x or 5.x

3. **Network access**: OpenWebUI must be able to reach OpenHAB

---

## Installation (5 Minutes)

### Step 1: Install Ollama + Model

```bash
# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Pull recommended model
ollama pull qwen3:14b

# Or alternative large model
ollama pull gpt-oss:20b
```

### Step 2: Install OpenWebUI (Docker)

```bash
docker run -d \
  --name open-webui \
  -p 3000:8080 \
  -v open-webui:/app/backend/data \
  --add-host=host.docker.internal:host-gateway \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

Open: `http://localhost:3000` and create account.

### Step 3: Import Tool

1. Download `tool-openhab_smart_home.json` from this repo
2. In OpenWebUI: **Workspace** (sidebar) → **Tools** (Werkzeuge)
3. Click **"+ New Tool"** or **"Import Tool"**
4. Upload JSON file
5. Configure settings:
   - **OpenHAB Host**: Your OpenHAB IP (e.g., `192.168.1.100`)
   - **OpenHAB Port**: `8080`
   - **API Token**: (optional, from OpenHAB Settings → API Security)
   - **Enable Item Filter**: `true` (recommended)

### Step 4: Test

Start new chat, enable the tool (🔧 button), and ask:

```
Show me all controllable devices
```

**Success!** ✅ You should see a list of your devices.

---

## Troubleshooting

### "Could not connect to OpenHAB"

```bash
# Test manually
curl -X GET http://your-openhab-ip:8080/rest/items

# If Docker: use host.docker.internal instead of localhost
# In tool settings: OpenHAB Host = host.docker.internal
```

### "No devices found"

- Disable filters: Tool Settings → `Enable Item Filter: false`
- Check OpenHAB has items: Should have devices configured

### "LLM not calling tool"

- Use qwen3 (best function calling support)
- Enable tool for chat: Click 🔧 button
- Try explicit: "Use the OpenHAB tool to list devices"

---

## Next Steps

- Read [EXAMPLES.md](EXAMPLES.md) for use cases
- Check [INSTALLATION.md](INSTALLATION.md) for detailed setup
- Star the repo if it helped you! ⭐
