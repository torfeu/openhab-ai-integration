# 🏠 OpenHAB AI Integration with OpenWebUI

> Transform your OpenHAB Smart Home into an intelligent, conversational system using Large Language Models

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![OpenHAB](https://img.shields.io/badge/OpenHAB-3.x%2F4.x-orange)](https://www.openhab.org/)
[![OpenWebUI](https://img.shields.io/badge/OpenWebUI-0.3.9%2B-blue)](https://github.com/open-webui/open-webui)

## 🎯 What This Does

This integration connects OpenHAB with AI models through OpenWebUI, enabling **natural language control and monitoring** of your smart home. Instead of remembering device names or writing automation rules, you can simply ask:

- *"Are there any anomalies in device power consumption?"*
- *"What's using the most energy right now?"*
- *"Turn off all lights in the bedroom"*

The AI **understands context**, **analyzes data**, and **provides intelligent insights** about your home automation system.

## ✨ Key Features

### 🧠 Intelligent Analysis
- **Anomaly Detection**: AI analyzes power consumption patterns and identifies unusual behavior
- **Contextual Understanding**: Recognizes that "4W at desk = monitors in standby"
- **Proactive Suggestions**: Recommends optimizations based on usage patterns

### 🔍 Advanced Search
- **Natural Language Queries**: Search devices using everyday language
- **Smart Filtering**: Automatically excludes group items and non-controllable devices
- **Vocabulary System**: 227+ recognized keywords extracted from your devices

### 🎛️ Device Control
- **Universal Control**: Supports switches, dimmers, rollershutters, thermostats, and more
- **Type-Aware Commands**: Knows which commands each device type accepts
- **Safety Checks**: Validates controllability before attempting changes

### 📊 Comprehensive Monitoring
- **327+ Devices**: Tested with large smart home installations
- **Real-time Status**: Instant access to current device states


## 🚀 Quick Start

**OpenHAB** 4.x or 5.x must be running and accessible. Then choose your preferred integration method:

---

## 🖥️ Option A — OpenWebUI (direct tool import)

Use the tool directly inside OpenWebUI as a function-calling tool.

### Prerequisites
- **OpenWebUI** 0.3.34 or higher
- **Ollama** with a compatible LLM (qwen3 or gpt-oss 20B)

### 1. Install OpenWebUI (if not already installed)

```bash
docker run -d -p 3000:8080 \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

### 2. Install Ollama and LLM

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama pull qwen3:14b
```

### 3. Import the OpenHAB Tool

1. In OpenWebUI → **Settings** → **Tools**
2. Click **"Import Tool"**
3. Upload `tool-openhab_smart_home.json` from this repository
4. Configure:
   - **OpenHAB Host**: `<your-openhab-ip>`
   - **OpenHAB Port**: `8080`
   - **API Token**: (optional) OpenHAB → Settings → API Security
   - **Enable Item Filter**: recommended

### 4. Test

Start a new chat and ask: `Show me all controllable devices in my smart home`

---

## 🔌 Option B — MCP Server (json-mcp-manager)

Run the OpenHAB tool as a standalone **MCP server** — accessible from Claude Code, Codex, Cursor, or any MCP client, without OpenWebUI.

Uses [owui-mcp-spawner](https://github.com/torfeu/owui-mcp-spawner).

### 1. Set up json-mcp-manager

```bash
git clone https://github.com/torfeu/owui-mcp-spawner.git
cd owui-mcp-spawner
python3 -m venv .venv
.venv/bin/pip install fastapi "uvicorn[standard]" pydantic python-multipart mcp packaging starlette httpx
.venv/bin/python app/manager.py
```

### 2. Install the OpenHAB tool

Open `http://127.0.0.1:7860` → upload `tool-openhab_smart_home.json` → **Install** → **Start**

### 3. Connect your MCP client

The tool is reachable at `http://<your-server-ip>:<port>/mcp`.

**Claude Code** — add to `~/.claude/mcp.json`:

```json
{
  "mcpServers": {
    "openhab": {
      "url": "http://<your-server-ip>:<port>/mcp"
    }
  }
}
```

---

## 📖 Usage Examples

### Device Discovery

```
User: What devices can I control in the bedroom?
AI: [Searches for "bedroom" + "controllable"]
    Found 5 devices:
    - Bedroom Light (Dimmer): 0-100%
    - Bedroom Ceiling (Color): color, brightness
    - Bedroom Rollershutter (Rollershutter): up, down, 0-100%
    ...
```

### Power Monitoring

```
User: Are there any anomalies in device power consumption?
AI: [Analyzes current consumption across all monitored devices]
    Current consumption overview:
    - Desk: 4W (likely monitors in standby)
    - 3D Printer (Living Room): 0W but 25kWh total
      ⚠️ High total consumption - check for standby losses
    
    Recommendation: Check if 3D printer is properly shut down after use
```

### Device Control

```
User: Turn off all lights in the office
AI: [Searches for "office" + "light"]
    Found 3 lights. Turning off...
    ✓ Office Desk Light → OFF
    ✓ Office Ceiling → OFF  
    ✓ Office Reading Lamp → OFF
```

### Vocabulary Discovery

```
User: What keywords can I use to search devices?
AI: Available keywords (227 total):
    bedroom, office, kitchen, living room, light, temperature,
    humidity, power, consumption, 3d printer, desk, ceiling, ...
```

## 🏗️ Architecture

```
┌─────────────┐      REST API      ┌──────────────┐
│  OpenHAB    │◄──────────────────►│  OpenWebUI   │
│  (Home      │                     │  Python Tool │
│  Automation)│                     └──────┬───────┘
└─────────────┘                            │
                                           │ Function Calls
                                           │
                                    ┌──────▼────────┐
                                    │  LLM (Ollama) │
                                    │  - qwen3      │
                                    │  - gpt-oss 20B│
                                    └───────────────┘
```

### How It Works

1. **User sends natural language query** to OpenWebUI chat
2. **LLM decides** which tools to call based on the query
3. **Python tool** makes REST API calls to OpenHAB
4. **OpenHAB responds** with device states/control confirmation
5. **LLM interprets** the data and responds in natural language
6. **User receives** intelligent, contextual answer

## 🛠️ Advanced Configuration

### Custom Filtered Types

By default, the tool filters out `Group` items. You can add custom types:

```python
# In OpenWebUI tool settings
Custom Filtered Types: "Group,Scene,DateTime"
```

### API Token (Recommended for Production)

Generate an API token in OpenHAB:
1. Go to **Settings** → **API Security**
2. Create new token with name "OpenWebUI"
3. Copy token and paste into tool configuration

### Cache TTL

Vocabulary is cached for performance. Default: 3600 seconds (1 hour)

```python
Cache TTL: 1800  # 30 minutes for more frequent updates
```

## 🧪 Tested Models

| Model | Version | Performance | Notes |
|-------|---------|-------------|-------|
| qwen3 | 14B/30B | ⭐⭐⭐⭐ | Excellent, fast, Function calling: native|
| gpt-oss | 20B | ⭐⭐⭐⭐ | Good, fast, Function calling: native|

## 📊 Real-World Example

**Scenario**: User asks about power consumption anomalies

**AI Response**:
```
🔍 Current Consumption Overview:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
• Desk Socket: 4W (0.146A)
  └─ Likely: Monitors in standby mode
  
• 3D Printer (Office): 0W
  └─ Yesterday: 0.33 kWh
  └─ Status: Properly shut down ✓
  
• 3D Printer (Living Room): 0W
  ⚠️ Total consumption: 25 kWh
  └─ Warning: High cumulative usage
  └─ Check for: Standby losses or incorrect sensor

• TV Socket: 0W (228V)
  └─ Total: 34.5 kWh
  └─ Status: Normal ✓

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Recommendations:
1. Desk monitors using 4W standby - consider smart power strip
2. Living room 3D printer shows 25kWh total - verify measurement accuracy
3. No critical anomalies detected
```

## 🤝 Contributing

This project was developed through collaboration between a smart home enthusiast and Claude AI. Contributions are welcome!

### Development Setup

```bash
git clone https://github.com/torfeu/openhab-ai-integration.git
cd openhab-ai-integration
```

## 📝 License

MIT License - see [LICENSE](LICENSE) file

## 🙏 Acknowledgments

- **OpenHAB Community** for the excellent home automation platform
- **OpenWebUI Team** for the flexible AI interface
- **Ollama** for making local LLMs accessible
- **Claude AI** for development assistance

## 🐛 Troubleshooting

### "Could not connect to OpenHAB"

- Check if OpenHAB is running: `http://your-openhab-ip:8080`
- Verify firewall allows port 8080
- Try without API token first (disable in settings)

### "No devices found"

- Check if filter is excluding too many items
- Disable filters temporarily: `Enable Item Filter: false`
- Verify OpenHAB has items configured

### LLM not calling tools

- Ensure model supports function calling (qwen3 recommended)
- Try simpler queries first: "List all devices"
- Check OpenWebUI logs for errors

## 📧 Support

- **Issues**: [GitHub Issues](https://github.com/torfeu/openhab-ai-integration/issues)

---

**Made with 🤖 by a human frustrated with traditional smart home UIs**

*Because talking to your home should be as natural as talking to a person*
