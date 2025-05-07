### [Prerequisite: Attacker's perspective]
- The vehicle communication module requests the server address based on the domain name.
- The proxy server is already under the control of the attacker, so it can monitor and modify traffic.
- The response that informs the IP address when the OTA update file is transmitted to the vehicle's communication module can be intercepted and reassigned.
- The OTA server is not secured.

### Step 1 (Assumption): File update to the OTA server
- Upload the firmware file to a normal OTA server.
- The OTA server registers it in a downloadable path.
- It notifies that the update has been made.

### Step 2 (Assumption): The vehicle communication module sends an OTA request
- The vehicle communication module sends an OTA request to the [ota.com](http://ota.com) domain.
- At this time, the server's IP address must be confirmed to receive the OTA file, so a process of confirming the server address (IP) occurs.
- The attacker intercepts the **response that informs the IP address of the server when requesting OTA** from the proxy server and manipulates the response to send his server IP address to the vehicle.
- The proxy monitors the traffic transmitted in the network layer, and when a request to confirm the server address is detected, it forges the response to respond with the attacker's IP address.

### Step 3: Attacker operates a fake OTA server
- The attacker provides a fake OTA file from his web server.
- The vehicle communication module receives the malicious file by the attacker.
- It copies the format (size, header, version structure) similar to the actual OTA file and inserts malicious commands/functions in the middle.
- In other words, the vehicle communication module recognizes the OTA update file as a normal file.

### Step 4: Vehicle communication module downloads the fake OTA file
- The vehicle communication module downloads the OTA file from the server connected to the forged IP.
- It receives the file with a status similar to a normal response.
- The communication module in the vehicle distributes the fake file to the ECU, and the ECU applies the file without separate integrity verification.
- Malicious commands are executed and abnormal behavior is triggered, allowing remote control and subsequent attacks.

In an environment where OTA server authentication, file signing, and encryption are not applied due to low security level where even integrity verification does not exist, an attacker can easily manipulate the request path and deliver a fake OTA file.


```python
# Used to create a web server using a Python tool called Flask.
from flask import Flask, send_file

# Starting the Flask web server
app = Flask(__name__)

# When a vehicle makes a request to the '/ota' address, the function below is executed.
@app.route("/ota")
def send_fake_file():
    # 'fake_ota_update.bin'이라는 파일을 다운로드하게 함
     return send_file("fake_ota_update.bin", as_attachment=True)

# Run the server on port 8000 (so any vehicle can access it)
if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)
```
