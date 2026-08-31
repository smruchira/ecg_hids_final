# Raspberry Pi deployment

This is the minimal deployment when another Raspberry Pi service creates
`full_log.jsonl` automatically. Copy only the following files/directories to
`/home/chega/HIDS/`:

```text
edge/config.py
edge/orchestrator.py
edge/watchdog.py
edge/privacy_strip.py
edge/render_client.py
edge/sdn_gateway_client.py
edge/requirements.txt
rpi/ecg-hids-orchestrator.service
```

The external producer must append JSONL records to:

```text
/home/chega/HIDS/full_log.jsonl
```

The orchestrator waits for that file and can create an empty placeholder if
the producer has not created it yet. Do not copy `.venv`, `__pycache__`,
`demo_stream.py`, `training/`, or `cloud/` to the runtime deployment.

Install and start the orchestrator:

```bash
cd /home/chega/HIDS
python3 -m venv edge/.venv
edge/.venv/bin/pip install -r edge/requirements.txt
sudo cp rpi/ecg-hids-orchestrator.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now ecg-hids-orchestrator.service
sudo journalctl -u ecg-hids-orchestrator -f
```

Only install the listener if this HIDS instance also receives ESP32 samples
directly. It is not needed when another service creates `full_log.jsonl`:

```bash
sudo cp rpi/ecg-hids-listener.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now ecg-hids-listener.service
```

Do not run `demo_stream.py` in production. It generates synthetic entries.
