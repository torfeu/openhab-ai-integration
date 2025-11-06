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

### Prerequisites

- **OpenHAB** 4.x or 5.x running and accessible
- **OpenWebUI** 0.3.34 or higher installed
- **Ollama** with a compatible LLM (tested with qwen3 and gpt-oss 20B)
- Python 3.8+ (included with OpenWebUI)

### Installation

#### 1. Install OpenWebUI (if not already installed)

```bash
# Using Docker (recommended)
docker run -d -p 3000:8080 \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

Visit `http://localhost:3000` to complete setup.

#### 2. Install Ollama and LLM

```bash
# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Pull a compatible model (qwen2.5 recommended)
ollama pull qwen3:14b

# Alternative: GPT-OSS
ollama pull gpt-oss:latest

#### 3. Import the OpenHAB Tool

1. In OpenWebUI, go to **Settings** → **Tools**
2. Click **"Import Tool"**
3. Upload the `tool-openhab_smart_home.json` file from this repository
4. Configure the tool settings:
   - **OpenHAB Host**: Your OpenHAB IP (e.g., `192.168.1.100`)
   - **OpenHAB Port**: Usually `8080`
   - **API Token**: (Optional) Generate in OpenHAB under Settings → API Security
   - **Enable Item Filter**: Recommended (excludes Groups for cleaner results)

#### 4. Test the Integration

Start a new chat in OpenWebUI and ask:

```
Show me all controllable devices in my smart home
```

If successful, you'll see a categorized list of your devices!

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
| qwen3 | 14B/30B | ⭐⭐⭐⭐ | Excellent, fast |
| gpt-oss | 20B | ⭐⭐⭐⭐ | Good, fast |

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
git clone https://github.com/yourusername/openhab-ai-integration.git
cd openhab-ai-integration

# Test the tool locally
python test_tool.py
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

- **Issues**: [GitHub Issues](https://github.com/yourusername/openhab-ai-integration/issues)
- **Discussions**: [GitHub Discussions](https://github.com/yourusername/openhab-ai-integration/discussions)

---

**Made with 🤖 by a human frustrated with traditional smart home UIs**

*Because talking to your home should be as natural as talking to a person*
