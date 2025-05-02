# Attack Scenarios

---
## Attack Proposol
- The attacker aims to **cause a vehicle accident or gain control of it** by injecting malicious code
- The attacker aims to **keep the vehicle in a secure or functionally vulnerable state** by interfering with the official OTA

  
## 1. Supply Chain Attack

### Attack Scenario 1: Malware Update via Official OTA File Upload Webpage [Level 1]

#### Precondition
Assume that the attacker has obtained the IP address (or domain) of the website where the file is uploaded.

#### Attack Procedure
1. The attacker creates a .bin file corresponding to the malicious firmware/software to be injected into the target ECU.
2. The .bin file is uploaded through the official website that uploads OTA files.
3. Vehicles without authentication install the malicious file.
---

### Attack Scenario 2: Publishing malicious code or URL to MQTT broker [Level 1]

#### Precondition
Assume that an authenticated vehicle has been hacked and the IP of the MQTT Broker used by the OEM has been acquired.

#### Attack Procedure
1. Publish a malicious file or firmware/software download URL to the MQTT broker.
2. The broker broadcasts the information to all vehicles with matching topics.
3. Vehicles without authentication install the malicious file.
---

## 2. Man-in-the-middle Attack

### Attack Scenario 1: Intercept and Tamper The File Uploaded to OTA webpages

#### Precondition of Attack Scenario
It is assumed that the attack has successfully changed the proxy server settings of the administrator’s PC (used to upload firmware) to route traffic through the attacker's proxy server.

#### Attack Procdure
1. Assign the proxy server IP address of the administrator’s PC to that of the attacker.
2. Use tools like Burp Suite to intercept the HTTP packet sent when uploading the firmware file to the OTA server.
3. Tamper the captured file and send it to the OTA server
4. Vehicles without authentication mechanisms will download and install the malware.
---

### Attack Scenario 2: DNS/Proxy Spoofing for Fake Server Redirection 

#### Precondition
Assume that an attacker can compromise a proxy server or DNS server and manipulate its responses to IP address queries.

#### Attack Procedure
1. Assume that the attacker has forged the IP address responded from proxey or DNS,  connect the client directly to the fake MQTT broker or OTA server.
2. After comfirm the connection and request from targe vehicle, Generate correct packet for the vehicle.
3. Send the generated malware to the vehicle.
4.  Vehicles without authentication mechanisms will download and install the malware.

#### Additional Scenarios with secure OTA

##### 1. On Server: Using HTTPS(SSL/TLS)
- We will use a fake CA file injectied on the vehicle.
- We will perform the activity by injecting the same CA file as the fake server to the client corresponding to the vehicle for the attack scenario

##### 2. On vehicle: Digital sign verification
- After injecting the vehicle to use the attacker's signature instead of the server's public key
- To perform the activity, the public key of attacker for the vehicle is performed.

---

### Attack Scenario 3: Packet Interception and Tamperation via Compromised Router

#### Precondition
Assume the attacker has gained control over a router that relays wireless communication, allowing interception and modification of OTA update packets.

#### Attack Procedure
1. Upload the update file via OTA.
2. Assuming that the router sends packets to the vehicle via the attacker IP, send the file to the attacker IP.
3. Disguise the attacker IP as the server IP and send it to the vehicle.
4. Vehicles without authentication mechanisms will download and install the malware.

---

## 3. Replay Attack

### Attack Scenario 1: Reuse Previous Update Packet

#### Precondition
Assume that the vehicle is connected to the fake MQTT broker. So the attacker can try to update malware at anytime
#### Attack Prcedure
1. Capture and store normal packets from the another OTA target.
2. Assuming that vulnerabilities in the past version have been fixed by the new version, update the past version of the file through a fake MQTT broker.
3. The vehicle installs the packet due to insufficient message freshness verification.
---
