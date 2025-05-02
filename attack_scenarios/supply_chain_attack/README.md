# 1. Supply Chain Attack

## Attack Scenario 1: Malware Update via Official OTA File Upload Webpage [Level 1]

### Precondition
Assume that the attacker has obtained the IP address (or domain) of the website where the file is uploaded.

### Attack Procedure
1. The attacker creates a .bin file corresponding to the malicious firmware/software to be injected into the target ECU.
2. The .bin file is uploaded through the official website that uploads OTA files.
3. Vehicles without authentication install the malicious file.
---

## Attack Scenario 2: Publishing malicious code or URL to MQTT broker [Level 1]

### Precondition
Assume that an authenticated vehicle has been hacked and the IP of the MQTT Broker used by the OEM has been acquired.

### Attack Procedure
1. Publish a malicious file or firmware/software download URL to the MQTT broker.
2. The broker broadcasts the information to all vehicles with matching topics.
3. Vehicles without authentication install the malicious file.
---
