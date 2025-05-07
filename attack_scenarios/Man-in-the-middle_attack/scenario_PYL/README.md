# OTA MITM Attack Simulation

This project simulates a Man-In-The-Middle (MITM) attack on an insecure OTA (Over-The-Air) update system. The attacker intercepts and redirects OTA requests, delivering a malicious update file instead of the legitimate one.

## Assumptions

- The attacker has already compromised the proxy server.
- The vehicle’s communication module makes OTA requests using domain names (e.g., `ota.com`).
- The proxy server, under the attacker's control, can inspect and modify traffic.
- The OTA server lacks authentication, integrity checks, and encryption.
- The vehicle does not verify signatures or hashes on received files.

---

## Attack Scenario (Attacker’s Perspective)

### Step 1: Firmware Update Uploaded to OTA Server (Baseline Assumption)
- The legitimate OTA server hosts a new firmware file.
- It is published at a downloadable URL.
- A notice is sent out that an update is available.

### Step 2: Vehicle Makes OTA Request
- The vehicle's communication module sends an OTA request to `http://ota.com`.
- To fetch the file, it needs to resolve the domain to an IP.
- The attacker intercepts the IP resolution response and replaces it with the attacker’s server IP.
- From the vehicle’s perspective, `ota.com` now points to the attacker-controlled server.

### Step 3: Attacker Hosts a Fake OTA Server
- The attacker operates a malicious OTA server that mimics the real one.
- The fake server responds with a crafted `fake_ota_update.bin`.
- The file mimics the format of the real update (headers, version, etc.) but contains malicious code.
- Optionally, the attacker recomputes the hash and signature fields to match the file content.
- Alternatively, the attacker may deliver an old vulnerable version (rollback attack).

### Step 4: Vehicle Downloads Malicious OTA File
- The vehicle downloads the file, assuming it’s legitimate.
- If integrity verification is skipped or bypassed, the malicious update is accepted.
- The update is passed to the ECU, which applies it without further verification.
- The attacker gains the ability to remotely control or disrupt the vehicle.

---

## Additional Attack Variants

- **Rollback attack**: Serving outdated vulnerable firmware.
- **Hash collision attack**: Reusing the same hash in the OTA package.
- **Fake version metadata**: Tricking version checks by spoofing metadata.
- **Signed-but-malicious file**: When code signing is not enforced.

---

## Step-by-Step Simulation

### Step 1: Create a Fake OTA Server

```python
# fake_ota_server.py

from flask import Flask, send_file

app = Flask(__name__)

@app.route("/ota")
def send_fake_file():
    return send_file("fake_ota_update.bin", as_attachment=True)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)
```

#### Explanation

- A simple Flask server listens on port 8000.
- When /ota is accessed, it returns the fake binary file.
- The .bin file imitates a real OTA update.

### Step 2: Simulate the Vehicle's Communication Module

```python
# vehicle_module.py

import requests

url = "http://ota.com:8000/ota"

print("[Vehicle] Sending OTA update request...")

response = requests.get(url)

if response.status_code == 200:
    with open("downloaded_ota.bin", "wb") as f:
        f.write(response.content)
    print("[Vehicle] OTA file received! Saved as downloaded_ota.bin")
else:
    print("[Vehicle] Failed to download OTA file. Status code:", response.status_code)
```

#### Explanation

- Sends an HTTP GET request to `http://ota.com:8000/ota`.
- If successful, saves the file as `downloaded_ota.bin`.
- Simulates a vehicle blindly trusting the update file.

## Configuration (for local testing)
### 1. Modify `hosts` file on your machine (Windows only)
To redirect `ota.com` to your local attacker server:
1. Run Notepad as Administrator
2. Open: `C:\Windows\System32\drivers\etc\hosts`
3. Add the following line:
```
127.0.0.1 ota.com
```
4. Save and close

## Disclaimer
This simulation is for educational and research purposes only. Never use these techniques against real vehicles or systems. Always get permission before performing security testing.

## File Structure
```graphql
.
├── fake_ota_server.py        # Fake OTA server (attacker)
├── vehicle_module.py         # Simulated vehicle requesting the OTA file
├── fake_ota_update.bin       # Malicious binary (crafted by attacker)
└── README.md                 # This file
```
