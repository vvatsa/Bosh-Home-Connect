# Bosh-Home-Connect

Local control of a Bosch Smart Home Controller (SHC gen 1) and its thermostats,
using Bosch's documented local REST API. No cloud, no Bosch app after setup.

## Step 1: Pair (do this first)

Pairing registers this computer as a client on the controller and produces a
certificate + key. Every later step needs them.

You need:
- The controller's IP address (check the router's device list).
- The **system password** set during the controller's initial setup in the Bosch app.
- A computer on the same network, with Python 3.10+.

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# 1. Press and HOLD the front button on the controller until the LEDs flash.
# 2. Within about a minute, run:
python -m boschshcpy.register_client -ip <CONTROLLER_IP> -n homectl -id homectl
```

It prompts for the system password and writes `oss_homectl_<host>_cert.pem` and
`oss_homectl_<host>_key.pem` (mode 0600). Keep them private; they are git-ignored.
The client is registered as `oss_...` with `ROLE_RESTRICTED_CLIENT`, as Bosch's terms require.

If pairing fails with an SSL error, the controller wasn't in pairing mode. Hold the button again and retry.

## Step 2: Verify

```bash
python -m boschshcpy.rawscan -ip <IP> -cert <cert.pem> -key <key.pem> devices
python -m boschshcpy.rawscan -ip <IP> -cert <cert.pem> -key <key.pem> rooms
```

## Step 3 (next): dump everything, including schedules

Heating schedules aren't in the official spec. A dump of `rooms`, `services` and
`automation/rules` from the real controller (saved under `dumps/`) will show how
they are stored before we build schedule control.

Bosch API terms: https://github.com/BoschSmartHome/bosch-shc-api-docs (private, non-profit use only).
