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

# OPTIONAL: arbitrary HTTP headers to send with each request
rest_headers = { authorization = "Bearer your-api-key-here" }

# OPTIONAL: body fields merged with defaults (file, language, etc.)
rest_body = { model = "custom-model" }

# OPTIONAL: convenience for Authorization header
# Equivalent to: rest_headers = { authorization = "Bearer your-api-key-here" }
rest_api_key = "your-api-key-here"

# OPTIONAL: request timeout in seconds (default: 30)
rest_timeout = 30

# OPTIONAL: language hint, sent as a form field (see below)
# Can be overridden per-request by setting language in rest_body
language = "en"
```

Note: `rest_body` merges with auto-generated fields (like `language`). 

Set `language` inside `rest_body` if you need to override the configured language per request.
    
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

Additional headers can be configured via `rest_headers`. 

The `rest_api_key` option is a convenience that sets `Authorization: Bearer <rest_api_key>`.

### Body encoding

Hyprwhspr always sends the request as multipart/form-data.

The multipart request will contain:

1. A file field containing the WAV audio (always included)

2. Fields from `rest_body` configuration (optional, merged with defaults)

3. A language field (included if configured, unless overridden in `rest_body`)

### Form fields

**file (required, auto-generated)**

Field name: file
Filename: audio.wav
MIME type: audio/wav
Format: 16-bit PCM
Channels: mono
Sample rate: typically 16 kHz

Servers must read this field as the raw audio input. This field is always included automatically.

**language (optional, auto-generated)**

Field name: language
Type: text
Included if configured in hyprwhspr's top-level `language` setting, unless overridden by setting `language` in `rest_body`.

Servers may use this as a language hint.

**Custom fields (optional, from `rest_body`)**

Any fields specified in `rest_body` will be merged with the default fields (file, language). 

Fields in `rest_body` take precedence over auto-generated fields with the same name.

### Example raw request (conceptual)

With configuration:

```toml
rest_headers = { authorization = "Bearer some-token", "x-model" = "parakeet-tdt-0.6b-v3" }
rest_body = { temperature = "0.0" }
language = "en"
```

The request would be:

```
POST /transcribe HTTP/1.1
Host: your-server.example.com
Accept: application/json
Authorization: Bearer some-token
X-Model: parakeet-tdt-0.6b-v3
Content-Type: multipart/form-data; boundary=----XYZ

------XYZ
Content-Disposition: form-data; name="file"; filename="audio.wav"
Content-Type: audio/wav

(binary WAV bytes...)

------XYZ
Content-Disposition: form-data; name="language"

en

------XYZ
Content-Disposition: form-data; name="temperature"

0.0

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
