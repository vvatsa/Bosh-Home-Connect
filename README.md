# Bosh-Home-Connect

Local control of a Bosch Smart Home Controller (SHC gen 1) and its thermostats,
using Bosch's documented local REST API. No cloud, no Bosch app after setup.

## Step 0: Taking over a controller you didn't set up

The system password is created during the initial setup in the Bosch app, and the
controller may still be tied to the previous owner's account. Options, easiest first:

1. **Ask the previous owner** for the system password, or to remove the controller
   from their account. If they give you the password, skip to Step 1.
2. **Factory-reset the controller** (physical access needed). Follow Bosch's official
   reset instructions for the Smart Home Controller I; check the exact button timing
   on Bosch's support site or the device manual rather than guessing. A reset wipes
   the controller, so every thermostat must be re-paired to it, and a new system
   password is then set.
3. **Run setup once in the Bosch Smart Home app** (needs a Bosch account). This is the
   one step that can't avoid the app. After it, the app is no longer needed day to day.

Write the system password down. Everything below uses it.

## Step 0a: Get the Bosch Smart Home app (only needed once)

The app is needed once, to set the system password (Step 0). Its Play Store listing
is UK-only, so it may not install from Ireland (or elsewhere) normally.

- Package name: `com.bosch.sh.ui.android` (developer: Bosch / Robert Bosch Smart Home GmbH)
- Play link: https://play.google.com/store/apps/details?id=com.bosch.sh.ui.android

Try in this order, on an **Android** device (much easier than iOS):

1. **Open the Play link in the Play Store app.** If it shows Install, just install it.
   If it says "not available in your country", continue. If it says the device isn't
   compatible, that is an Android version/hardware limit and the options below may not help.
2. **Aurora Store** (open-source Play client): install it from `auroraoss.com` or F-Droid,
   choose anonymous login, set the spoofed locale to the UK in its settings, then search for
   the package name above and install. No UK Google account needed.
3. **APK sideload** from a reputable mirror such as APKMirror, as a fallback. Third-party files
   can be tampered with, so check the package name matches exactly and the developer is Bosch.
   Allow "install unknown apps" for the one app you download with, and turn it off again afterwards.

iPhone/iPad: the App Store listing is UK-only, so you would need a separate Apple ID set to the
UK store. Use an Android device if you can.

Unverified: whether account sign-up and controller setup work from outside the UK once the app
is installed. Expected to work (setup happens on the home network), but not confirmed.

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
