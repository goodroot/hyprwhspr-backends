# GPT-4o-mini-transcribe

Cloud-based transcription using [OpenAI's GPT-4o-mini model](https://platform.openai.com/docs/models/gpt-4o-mini-transcribe).

No local installation required - runs entirely via OpenAI's API.

https://github.com/goodroot/hyprwhspr-backends/blob/main/backends/gpt-4o-mini-transcribe/config.json

## Setup

You'll need an OpenAI API key with access to the transcription endpoint.

### Configure hyprwhspr

Add these keys to your `~/.config/hyprwhspr/config.json`:

```json
{
    "transcription_backend": "remote",
    "rest_endpoint_url": "https://api.openai.com/v1/audio/transcriptions",
    "rest_api_key": "sk-proj-YOUR-API-KEY-HERE",
    "rest_body": {
        "model": "gpt-4o-mini-transcribe"
    }
}
```

Replace `sk-proj-YOUR-API-KEY-HERE` with your actual OpenAI API key.

That's it! No backend server to run, no dependencies to install.
