# hyprwhspr backends

This repository:

* documents the HTTP contract for remote [`hyprwhspr`](https://github.com/goodroot/hyprwhspr) backends

* collects example configurations and backend implementations 

---

## 1. Contract overview

When `transcription_backend` is set to `"remote"`, hyprwhspr will:

1. Capture audio locally.
2. Convert it to a 16-bit PCM WAV file in memory.
3. Send it to a configured HTTP endpoint as a `multipart/form-data` POST request.
4. Expect a JSON response containing the transcription.

This document defines that HTTP contract.

---

## 2. Hyprwhspr configuration

Remote mode is configured via `hyprwhspr`'s config:

```toml
transcription_backend = "remote"

# REQUIRED: the full URL to your transcription endpoint
rest_endpoint_url = "https://your-server.example.com/transcribe"

# OPTIONAL: sent as `Authorization: Bearer <rest_api_key>`
rest_api_key = "your-api-key-here"

# OPTIONAL: request timeout in seconds (default: 30)
rest_timeout = 30

# CURRENTLY: hyprwhspr always sends WAV audio.
# Reserved for future use if we add support for other formats.
rest_audio_format = "wav"

# OPTIONAL: language hint, sent as a form field (see below).
language = "en"
```

## 3. HTTP request (client contract)

### 3.1 Method & URL

Method: POST

URL: whatever is configured as rest_endpoint_url.

Example:

```
POST https://your-server.example.com/transcribe
```

### 3.2 Headers sent by hyprwhspr

Accept: application/json

Authorization: Bearer <rest_api_key> (only if configured)

### 3.3 Body encoding

Hyprwhspr always sends the request as multipart/form-data.

The multipart request will contain:

1. A file field containing the WAV audio

2. A language field (optional)

### 3.4 Form fields

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

### 3.5 Example raw request (conceptual)

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

## 4. HTTP response (server contract)

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
