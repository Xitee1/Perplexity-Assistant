<div align="center">
	<h1>Perplexity Assistant for Home Assistant</h1>
	<p>Your privacy-friendly AI helper powered by <a href="https://www.perplexity.ai/">Perplexity AI</a>, integrated directly into Home Assistant.</p>
	<p>
		<strong>Status:</strong> Experimental • <strong>Version:</strong> 1.0.0 • <strong>Integration Type:</strong> Service
	</p>
</div>

---

## ✨ Overview

Perplexity Assistant brings the Perplexity conversational AI experience into Home Assistant. It lets you:

* Ask natural language questions via the conversation system or a custom service (`perplexity_assistant.ask_perplexity`).
* Get concise smart-home–oriented responses (with optional persistent notifications).
* Provide a custom system prompt to tune the assistant's behavior.
* Allow controlled access to entity states for contextual answers (optional).
* Allow the assistant to emit Home Assistant ACTION directives you can parse to trigger services (optional; experimental).
* Track estimated API usage cost with two diagnostic sensors: Monthly cost and All‑time cost (placeholder until official billing endpoints become available).

> The assistant is designed to produce safe, short, clear responses suitable for voice TTS or dashboard display.

## 🚀 Features

| Feature | Description |
|---------|-------------|
| Conversation Agent | Registers as a Home Assistant conversation provider. |
| Service Call (`ask_perplexity`) | Manually send prompts from scripts/automations. |
| Custom System Prompt | Override or extend built-in behavioral instructions. |
| Entity Context (optional) | Provides a summary of your entities to the model. |
| Action Lines (experimental) | Model can output `ACTION: service.call - entity_id` lines. |
| Notification Option | Persist responses as notifications. |
| Cost Sensors | Track monthly and all-time (approx) usage cost (placeholder). |
| Options Flow | Modify API key, language, model, permissions post-install. |

## 🧩 Requirements

Before installing:

* A valid Perplexity API Key (create one at: https://www.perplexity.ai/account/api/keys )
* Home Assistant 2024.x or later (tested on recent versions supporting modern conversation API)
* Internet connectivity (cloud polling)

## 📦 Installation

1. Copy the `perplexity_assistant` directory into your Home Assistant `config/custom_components` folder.
2. Restart Home Assistant.
3. Go to: Settings → Devices & Services → Add Integration → Search for "Perplexity Assistant".
4. Enter your API key and desired options.
5. Finish the flow — the integration sets up the conversation agent and sensors (if enabled).

### HACS (Planned)
HACS support is not yet published. Once available, you will be able to add this repository as a custom integration.

## ⚙️ Configuration (Initial Flow)

During setup you can specify:

* API Key (required, must start with `pplx-` and length 53)
* Language (default: `en`)
* Model (default: `sonar` — other options include `sonar-pro`, `sonar-reasoning`, etc.)
* Notify Each Response (persistent notification of outputs)
* Custom System Prompt (short textual instruction override, up to 250 chars)
* Allow Entities Access (if enabled, entity states summary is sent to the model)
* Allow Actions On Entities (if enabled, `ACTION:` directives may be parsed/executed — implement parsing safely yourself)

## 🔁 Options Flow (Post-Install)

Navigate to the integration card → Configure to update the above fields. Changes take effect immediately after saving.

## 🗣️ Using the Assistant

### 1. Conversation UI / Voice
You can use voice assistants or the built-in conversation interface. When registered, Perplexity Assistant becomes an available conversation agent.

### 2. Service Calls
Call the service manually from Developer Tools → Services or from automations/scripts:

```yaml
service: perplexity_assistant.ask_perplexity
data:
	prompt: "What is the temperature in the living room?"
```

Optional fields (future extension) may include model overrides if added to `services.yaml`.

### 3. Parsing ACTION lines (Experimental)
If you allow actions and the model includes lines like:

```
ACTION: light.turn_on - light.living_room
ACTION: climate.set_temperature - climate.downstairs
```

You can write an automation that triggers on a response event and safely filters/validates allowed services/entities before execution.

## 📊 Sensors

Two diagnostic sensors are created if the flag `create_credit_sensor` was set during initial config:

| Sensor Name | Description |
|-------------|-------------|
| `sensor.perplexity_monthly_bill` | Aggregates cost for current month (resets monthly). |
| `sensor.perplexity_bill` | Aggregates total cost across all usage. |

> Cost values are based on the `usage.cost.total_cost` field in responses. If API cost data changes or is unavailable these may remain 0 or inaccurate.

## 🔐 Privacy & Safety

* No entity states are sent unless you explicitly enable “Allow access to Home Assistant entities”.
* Action execution is opt-in; by default responses are inert.
* You should audit any automated ACTION execution to avoid unintended device changes.

## 🛠 Developer Guide

### Project Layout

```
perplexity_assistant/
	__init__.py              # Entry setup/unload, service registration, platform forwarding
	config_flow.py           # Config + options flow definitions
	const.py                 # Constants (models, languages, system prompt)
	conversation.py          # Conversation agent implementation
	sensor.py                # Diagnostic cost sensors (monthly + all-time)
	services.yaml            # Service schema definition
	strings.json             # UI strings for config/options flow
	manifest.json            # Integration metadata
	README.md                # Documentation
```

### Key Components
* `async_setup_entry` registers the agent + service and forwards platforms.
* `conversation.py` implements `AbstractConversationAgent` with cost tracking and optional entity/context injection.
* `sensor.py` exposes cost aggregation; methods `increment_cost()` are invoked after successful API responses.

### Adding Real Billing / Credit Fetching
Replace the placeholder logic in `sensor.py` with a coordinator fetching real billing endpoints (if Perplexity exposes them). Recommended steps:
1. Create a `DataUpdateCoordinator` subclass to poll billing usage.
2. Store results in coordinator; sensors subscribe to updates.
3. Add throttling (`update_interval`) mindful of rate limits.

### Extending ACTION Handling
Currently response parsing is inline and partial. Improvements:
* Factor parsing into a helper (e.g., `action_parser.py`).
* Validate service names against an allowlist.
* Confirm entity existence and domain compatibility.
* Batch actions with `async_create_task` and error isolation.

### Testing Strategy (Suggested)
* Unit: Mock `aiohttp` responses for `conversation.py`.
* Integration: Simulate a config entry and verify sensors increment cost.
* Edge Cases:
	- Empty prompt
	- API error (non-200)
	- Invalid API key
	- Month rollover for monthly cost sensor
	- Large entity lists (performance)

### Contributing
1. Fork the repository.
2. Create a feature branch: `git checkout -b feat/your-feature`.
3. Make changes + add/update tests.
4. Run formatting (e.g., `ruff`, `black`) as desired.
5. Submit a PR with a clear description and screenshots/logs if UI/UX changes.

### Release & Versioning
Uses semantic versioning: `MAJOR.MINOR.PATCH`.
* Increment PATCH for bug fixes.
* Increment MINOR for backward-compatible features.
* Increment MAJOR for breaking changes.

## 🧪 Troubleshooting

| Issue | Possible Cause | Fix |
|-------|----------------|-----|
| Invalid API key error | Key format mismatch | Re-generate and ensure it starts with `pplx-` and is 53 chars. |
| No sensors created | `create_credit_sensor` missing | Remove and re-add integration (fresh config flow) or implement an options migration. |
| Empty responses | API transient error | Check logs; enable debug logging for `perplexity_assistant`. |
| Actions ignored | Actions disabled | Enable “Allow actions on entities” in options. |
| Costs remain 0 | API didn’t return cost usage | Confirm Perplexity response structure; update parsing. |

### Enable Debug Logging
Add to your `configuration.yaml`:

```yaml
logger:
	default: warning
	logs:
		perplexity_assistant: debug
```

## 🗺 Roadmap

* Real credit / token usage retrieval.
* HACS distribution.
* Improved ACTION execution sandbox.
* Optional streaming mode (real-time tokens).
* Translation + localization improvements.

## 📝 License

This project is distributed under the MIT License (add the LICENSE file if not present).

## 🙏 Acknowledgments

* Powered by Perplexity AI
* Inspired by Home Assistant’s openness and extensibility
* Created by Pekul

---

For questions or issues, open an issue on GitHub: https://github.com/Pekulll/perplexity-assistant/issues

