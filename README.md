# hyprwhspr backends

This repository:

* collects example configurations and backend implementations 

* documents the HTTP contract for remote [`hyprwhspr`](https://github.com/goodroot/hyprwhspr) backends

---

## Backends

* [Parakeet-tdt-0.6b-v3](https://github.com/goodroot/hyprwhspr-backends/tree/main/backends/parakeet-tdt-0.6b-v3)

> To request a new backend, [open an issue](https://github.com/goodroot/hyprwhspr-backends/issues/new?template=new_backend.md). Alternatively, please contribute!

---

## Hyprwhspr configuration

Remote mode is configured via `hyprwhspr`'s config:

```toml
transcription_backend = "remote"

# REQUIRED: the full URL to your transcription endpoint
rest_endpoint_url = "https://your-server.example.com/transcribe"

# OPTIONAL: sent as `Authorization: Bearer <rest_api_key>`
rest_api_key = "your-api-key-here"

# OPTIONAL: request timeout in seconds (default: 30)
rest_timeout = 30

# OPTIONAL: Reserved for future use if we add support for other formats.
rest_audio_format = "wav"

# OPTIONAL: language hint, sent as a form field (see below).
language = "en"
```
    
---

## Contract overview

When `transcription_backend` is set to `"remote"`, hyprwhspr will:

1. Capture audio locally.
2. Convert it to a 16-bit PCM WAV file in memory.
3. Send it to a configured HTTP endpoint as a `multipart/form-data` POST request.
4. Expect a JSON response containing the transcription.

This document defines that HTTP contract.

## HTTP request (client contract)

### Method & URL

Method: POST

URL: whatever is configured as rest_endpoint_url.

Example:

```
POST https://your-server.example.com/transcribe
```

### Headers sent by hyprwhspr

Accept: application/json

Authorization: Bearer <rest_api_key> (only if configured)

### Body encoding

Hyprwhspr always sends the request as multipart/form-data.

The multipart request will contain:

1. A file field containing the WAV audio

2. A language field (optional)

### Form fields

**file (required)**

Field name: file
Filename: audio.wav
MIME type: audio/wav
Format: 16-bit PCM
Channels: mono
Sample rate: typically 16 kHz

Servers must read this field as the raw audio input.

**language (optional)**

Field name: language
Type: text
Only included if configured in hyprwhspr.

Servers may use this as a language hint.

### Example raw request (conceptual)

```
POST /transcribe HTTP/1.1
Host: your-server.example.com
Accept: application/json
Authorization: Bearer some-token
Content-Type: multipart/form-data; boundary=----XYZ

------XYZ
Content-Disposition: form-data; name="file"; filename="audio.wav"
Content-Type: audio/wav

(binary WAV bytes...)

------XYZ
Content-Disposition: form-data; name="language"

en

------XYZ--
```

## HTTP response (server contract)

On success, the server must return:

* Status: 200

* Body: JSON with a transcription string under one of these keys:

```
1. { "text": "hello world" }

2. { "transcription": "hello world" }

3. { "result": "hello world" }
```

Hyprwhspr checks these keys in this order:

text → transcription → result.
