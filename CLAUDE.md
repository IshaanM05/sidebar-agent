# Sidebar — Voice Agent

AssemblyAI Voice Agent Hackathon project (Sep 2026). Two-mode voice agent: co-pilot (diarized listening + on-demand agent) and coach (adaptive 1:1).

## Stack
- Python 3.10+
- AssemblyAI SDK (`assemblyai`) for Universal-Streaming v3
- Raw WebSockets (`websockets`) for Voice Agent API
- `sounddevice` + `numpy` for local audio I/O
- `python-dotenv` for env loading

## Project structure
```
src/sidebar/
├── config.py       — constants, env loading, wake phrase
├── audio.py        — shared mic capture (16kHz), resampling (→24kHz), playback, echo suppression
├── streaming.py    — Universal-Streaming listener, diarized turns, rolling transcript
├── agent.py        — Voice Agent API client, tool dispatch, context updates
├── tools.py        — tool definitions (flat schema) and execution (calculate, get_time, define_word)
└── engine.py       — orchestrator: wake detection, audio gate, context handoff
run.py              — entry point
hello_streaming.py  — standalone streaming test
hello_agent.py      — standalone agent test
```

## Architecture
- **Universal-Streaming** (`wss://streaming.assemblyai.com/v3/ws`): diarized ears, always listening
- **Voice Agent API** (`wss://agents.assemblyai.com/v1/ws`): mouth+brain, activated on wake-phrase "hey sidebar"
- Audio captured at 16kHz (streaming), resampled to 24kHz for agent
- Echo suppression: mic forwarding to agent muted during playback + 300ms guard

## Key API gotchas
- Voice Agent API auth requires `Bearer` prefix; streaming does not
- Voice Agent API audio is PCM16 24kHz **base64-encoded in JSON**, not raw binary frames
- Tool schema is flat `{type, name, description, parameters}` — not OpenAI nested format
- Input audio field: `input.audio.audio` / Output audio field: `reply.audio.data`
- `output.voice` is immutable after `session.ready` — commit before connecting
- Send `{"type": "session.end"}` for clean hangup (stops billing immediately)
- Always send `{"type":"Terminate"}` when closing streaming — abandoned sessions bill up to 3hrs
- Fetch voices live from `GET /v1/voices` — don't hardcode voice names

## Running
```bash
cp .env.example .env  # add your API key
pip install -r requirements.txt

# Full engine (Mode A co-pilot)
python run.py

# Standalone tests
python hello_streaming.py   # diarized streaming
python hello_agent.py       # voice agent round-trip
```
