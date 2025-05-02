# 2. Man-in-the-middle Attack

## Attack Scenario 1: Intercept and Tamper The File Uploaded to OTA webpages

### Precondition of Attack Scenario
It is assumed that the attack has successfully changed the proxy server settings of the administrator’s PC (used to upload firmware) to route traffic through the attacker's proxy server.

### Attack Procdure
1. Assign the proxy server IP address of the administrator’s PC to that of the attacker.
2. Use tools like Burp Suite to intercept the HTTP packet sent when uploading the firmware file to the OTA server.
3. Tamper the captured file and send it to the OTA server
4. Vehicles without authentication mechanisms will download and install the malware.
---

## Attack Scenario 2: DNS/Proxy Spoofing for Fake Server Redirection 

### Precondition
Assume that an attacker can compromise a proxy server or DNS server and manipulate its responses to IP address queries.

### Attack Procedure
1. Assume that the attacker has forged the IP address responded from proxey or DNS,  connect the client directly to the fake MQTT broker or OTA server.
2. After comfirm the connection and request from targe vehicle, Generate correct packet for the vehicle.
3. Send the generated malware to the vehicle.
4.  Vehicles without authentication mechanisms will download and install the malware.

### Additional Scenarios with secure OTA

#### 1. On Server: Using HTTPS(SSL/TLS)
- We will use a fake CA file injectied on the vehicle.
- We will perform the activity by injecting the same CA file as the fake server to the client corresponding to the vehicle for the attack scenario

#### 2. On vehicle: Digital sign verification
- After injecting the vehicle to use the attacker's signature instead of the server's public key
- To perform the activity, the public key of attacker for the vehicle is performed.

---

## Attack Scenario 3: Packet Interception and Tamperation via Compromised Router

### Precondition
Assume the attacker has gained control over a router that relays wireless communication, allowing interception and modification of OTA update packets.

### Attack Procedure
1. Upload the update file via OTA.
2. Assuming that the router sends packets to the vehicle via the attacker IP, send the file to the attacker IP.
3. Disguise the attacker IP as the server IP and send it to the vehicle.
4. Vehicles without authentication mechanisms will download and install the malware.

---
