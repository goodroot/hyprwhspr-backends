# Parakeet-tdt-0.6b-v3

Requires at least 2GB RAM / VRAM.

GPU recommended but not required - see the GPU flag in the Python file.

https://github.com/goodroot/hyprwhspr-backends/blob/main/backends/parakeet-tdt-0.6b-v3/parakeet-tdt-0.6b-v3.py

This backend uses Python:

```bash
# Create directory and download the backend file
# Place it somewhere permanent, for example:
mkdir -p ~/hyprwhspr-backends/parakeet-tdt-0.6b-v3
cd ~/hyprwhspr-backends/parakeet-tdt-0.6b-v3
curl -O https://raw.githubusercontent.com/goodroot/hyprwhspr-backends/main/backends/parakeet-tdt-0.6b-v3/parakeet-tdt-0.6b-v3.py

# Make executable
chmod +x parakeet-tdt-0.6b-v3.py

# Run backend (uv handles dependencies automatically!)
./parakeet-tdt-0.6b-v3.py
```

Alternatively, you can use `uv run`:

```bash
uv run parakeet-tdt-0.6b-v3.py
```

## Running with systemd (optional)

To start the backend on boot and restart on failure, run it as a systemd service.

This example is generic so you can adapt it to your own paths and usernames.

### Directory Layout

Place your backend somewhere permanent, for example:

```
/home/USER/hyprwhspr-backends/parakeet-tdt-0.6b-v3/
```

Inside that directory you should have:

- `parakeet-tdt-0.6b-v3.py`

Make sure you can run it manually before adding systemd:

```bash
cd /home/USER/hyprwhspr-backends/parakeet-tdt-0.6b-v3
uv run parakeet-tdt-0.6b-v3.py
```

Or if the script is executable:

```bash
./parakeet-tdt-0.6b-v3.py
```

### Example systemd service

Create:

```bash
sudo nano /etc/systemd/system/parakeet-tdt-0.6b-v3.service
```

Contents:

```ini
[Unit]
Description=Parakeet-tdt-0.6B-v3 backend for hyprwhspr
After=network.target hyprwhspr.service ydotool.service
Wants=hyprwhspr.service

[Service]
Type=simple
WorkingDirectory=/home/USER/hyprwhspr-backends/parakeet-tdt-0.6b-v3
ExecStart=/usr/bin/uv run /home/USER/hyprwhspr-backends/parakeet-tdt-0.6b-v3/parakeet-tdt-0.6b-v3.py

Environment=NEMO_CACHE_DIR=/home/USER/hyprwhspr-backends/parakeet-tdt-0.6b-v3/.nemo_cache
Environment=CUDA_VISIBLE_DEVICES=0

User=USER
Group=USER

Restart=on-failure
RestartSec=5

StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

Replace `USER` with the username that owns the backend directory.

### Enable and Start

```bash
sudo systemctl daemon-reload
sudo systemctl enable parakeet-tdt-0.6b-v3.service
sudo systemctl start parakeet-tdt-0.6b-v3.service
```

Check:

```bash
systemctl status parakeet-tdt-0.6b-v3.service
```

### Configure hyprwhspr

In your hyprwhspr config:

```json
{
    "transcription_backend": "remote",
    "rest_endpoint_url": "http://127.0.0.1:8080/transcribe",
    "rest_headers": {
        "x-model": "parakeet-tdt-0.6b-v3"
    },
    "rest_body": {
        "temperature": "0.0"
    },
    "rest_api_key": "your-api-key-here",
    "rest_timeout": 60,
    "language": "en"
}
```

Systemd integration is optional! 

The backend will work fine without it.

You just have to start it on your own. 🙂
