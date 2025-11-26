# Whisper Large V3

Cloud-based transcription using [Groq's Whisper Large V3 model](https://console.groq.com/docs/speech-to-text).

No local installation required - runs entirely via Groq's API. Groq delivers fast, low-cost inference with a generous free tier.

https://github.com/goodroot/hyprwhspr-backends/blob/main/backends/whisper-large-v3/config.json

## Setup

You'll need a Groq API key with access to the audio transcription endpoint.

### Configure hyprwhspr

Add these keys to your `~/.config/hyprwhspr/config.json`:

```json
{
    "transcription_backend": "remote",
    "rest_endpoint_url": "https://api.groq.com/openai/v1/audio/transcriptions",
    "rest_api_key": "gsk_YOUR-GROQ-API-KEY-HERE",
    "rest_body": {
        "model": "whisper-large-v3"
    }
}
```

Replace `gsk_YOUR-GROQ-API-KEY-HERE` with your actual Groq API key.

That's it! No backend server to run, no dependencies to install.