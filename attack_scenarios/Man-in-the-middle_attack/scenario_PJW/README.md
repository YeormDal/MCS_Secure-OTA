# Malware Distribution via a Spoofed OTA Server 

## 1. Scenario Overview and Initial Requirements

### 1.1 Scenario Overview
- In this scenario, the attacker impersonates the legitimate OTA (Over-the-Air) update server, intercepts the vehicle’s DNS requests, and redirects them to a malicious server. As a result, the vehicle installs a tampered OTA update containing malware.

### 1.2 Basic Requirements
- VMware : Used to run virtual machines for both the OTA server and client.
- dnsmasq : Redirects DNS requests to the attacker’s server (Flask-based OTA server).
- Bettercap : A tool used for DNS spoofing and man-in-the-middle (MITM) attacks.
- Flask(Python) : Serves as the spoofed OTA server backend.
- OpenSSL : Utilized to sign OTA packages and generate forged SSL certificates.

## 2. Hands-on Procedure
### Step 1: Building the OTA Server and DNS Environment
- Set up a Flask-based OTA server within a Kali Linux virtual machine.
- Configure dnsmasq to create a DNS spoofing environment.

### Step 2: Configuring DNS Spoofing
- Use Bettercap to redirect the vehicle’s DNS requests to the attacker’s OTA server IP.
- Forcefully map ota.realserver.com to the attacker’s IP address.

### Step 3: Spoofing the OTA Server and Setting Up HTTPS
- Deploy the Flask server over HTTPS to mimic a legitimate OTA server.
- Generate and apply a self-signed SSL certificate using OpenSSL.

### Step 4: Creating and Signing a Malicious OTA Package
- Generate an OTA update package (firmware.bin).
- Use OpenSSL to compute its hash and digitally sign it with the attacker’s private key.

### Step 5: Requesting OTA Update and Delivering Malicious Payload
- The client simulator connects to the OTA server and downloads firmware.bin, its signature, and hash.
- The client verifies the integrity and authenticity of the downloaded files before installation.
- Confirm whether the malicious OTA update is successfully installed.

## Step 1: Setting Up the OTA Server Environment
### 1.1 Objective of the Exercise
- In this step, we will build a mock OTA update server using Flask, simulating how a vehicle receives firmware updates from a trusted server.
- Flask : A lightweight web framework built on Python
  - It enables rapid development of web servers with minimal configuration, ideal for experimentation and prototyping.
  - While it can serve as a full web server like Apache, it is significantly more lightweight and easier to deploy.
 
### 1.2 Directory Structure
- We will begin by creating the folder structure required for the hands-on exercise.
```
mkdir flask_ota_lab
cd flask_ota_lab

mkdir certs updates server ota_client
```
![image](https://github.com/user-attachments/assets/384c0a0c-1fff-4b81-aa24-94f71b068488)

## Step 2 : Building the Flask-Based OTA Server
### 2.1 Installing Flask
```
pip install flask
```
- If you encounter installation errors on Kali Linux, try adding the --break-system-packages option to the command.
~~~
pip install flask --break-system-packages
~~~

![image](https://github.com/user-attachments/assets/0cfc97c0-dfd0-4bf6-a36c-10f071ef8068)

### 2.2 Creating the Flask OTA Server Code
- Use the nano editor to create the Python source file for the OTA server.
~~~
nano server/app.py
~~~
![image](https://github.com/user-attachments/assets/c5247dc6-4bc7-4ef7-9858-5dc6fc477eca)

- Write the server code in a file named app.py.
~~~
from flask import Flask, send_from_directory
import os

app = Flask(__name__)
BASE_DIR = os.path.dirname(os.path.abspath(__file__))
UPDATE_DIR = os.path.abspath(os.path.join(BASE_DIR, '..', 'updates'))

@app.route('/')
def index():
    return '<h1>Fake OTA Server - Flask</h1>'

@app.route('/<path:filename>')
def serve_file(filename):
    return send_from_directory(UPDATE_DIR, filename)

if __name__ == '__main__':
    cert_path = os.path.abspath(os.path.join(BASE_DIR, '..', 'certs', 'ota.crt'))
    key_path = os.path.abspath(os.path.join(BASE_DIR, '..', 'certs', 'ota.key'))
    app.run(host='0.0.0.0', port=443, ssl_context=(cert_path, key_path))
~~~
![image](https://github.com/user-attachments/assets/57df6953-9dd6-40be-a01d-b721a80773ec)

  - To save and exit the file in nano:
    Press Ctrl + X → Press Y (to confirm saving) → Press Enter (to finalize).
  - This sequence will be used frequently when creating or modifying files with nano.

## Step 3 : Creating an HTTPS Certificate and Integrating It with Flask
### 3.1 Objective of the Exercise
- Up to this point, the Flask server communicates over HTTP by default.
  - Since HTTP does not encrypt data, it is vulnerable to eavesdropping or tampering during transmission.
- We will now upgrade the server to use HTTPS (secure HTTP).
  - With HTTPS, communication between the server and client is encrypted and protected from interception.
- To enable HTTPS, a valid SSL/TLS certificate is required.
  - Without a certificate, browsers and clients may display warnings such as “This site is not secure.”
- Therefore, we will generate a self-signed certificate using OpenSSL.

- Goal: Apply this certificate to our Flask server to enable secure HTTPS communication.

### 3.2 Navigating to the cert Directory
- This is where we will store the certificate files.
  
~~~
cd ~/flask_ota_lab/certs
~~~

### 3.3 Generating a Certificate Using OpenSSL
~~~
openssl req -x509 -newkey rsa:2048 -keyout ota.key -out ota.crt -days 365 -nodes
~~~
![image](https://github.com/user-attachments/assets/1504f2e0-a62c-40ea-b59b-fb67bf8cb231)

- We will generate a self-signed HTTPS certificate (ota.crt) and a corresponding private key (ota.key) that:
  - Is valid for one year
  - Does not require a password for use
  
### You Will Be Prompted with the Following Input Fields During Generation
~~~
Country Name (2 letter code) [AU]: KR
State or Province Name (full name) [Some-State]: Seoul
Locality Name (eg, city) []: Gangnam
Organization Name (eg, company) []: OTA Labs
Organizational Unit Name (eg, section) []: Security Team
Common Name (e.g. server FQDN or YOUR name) []: ota.realserver.com
Email Address []: attacker@evil.com
~~~
- Common Name(CN) : Think of this as the “official identity” of the server embedded in the certificate.
  - If the CN does not match the domain the client is trying to connect to, the browser or client will not trust the certificate.
  - Therefore, enter ota.realserver.com as the Common Name.
- The remaining fields are optional metadata included in the certificate.
  - You may fill them in freely as they do not affect certificate validity in this context.

### 3.5 Verifying That the Certificate Was Successfully Created
~~~
ls -l
~~~
![image](https://github.com/user-attachments/assets/848fba6b-6c75-47af-90d3-0f39f28e4436)

- If you see both ota.crt and ota.key files, the certificate creation was successful!
- Additionally, you can see the full structure with the tree command!
~~~
tree ~/flask_ota_lab
~~~
![image](https://github.com/user-attachments/assets/860b50f2-6a3f-488a-882d-ebd17ff2674f)

### 3.6 Examine files with the cat command
~~~
cat ota.crt
~~~
![image](https://github.com/user-attachments/assets/be31a0fa-fae6-4909-9fdf-e61442819fca)

- View long public keys, issuance information, and more inside certificates
- Encrypt HTTPS communication with this

### Conclusion
- The FLask server now reads the "ssl_context=('certs/ota.crt', 'certs/ota.key')" option and can communicate securely over HTTPS!
  - The certificate has been successfully generated. ✔️
  - It has been securely stored in the certs/ directory. ✔️
  - The Flask server is now ready to be linked with the certificate. ✔️

## Step 4 : Creating and Signing a Malicious OTA Package
### 4.1 Objective of the Exercise
- When a vehicle receives an OTA update, it verifies the integrity of the file by checking its hash and digital signature.
- In this exercise, we will create a malicious OTA file (firmware.bin) and sign it using the attacker’s private key.
- This prepares us to deceive the vehicle into trusting and installing a tampered update.

### 4.2 Navigating to the updates Directory
- This directory will store all relevant OTA files, including firmware.bin, its hash, and the digital signature.
~~~
cd ~/flask_ota_lab/updates
~~~


### 4.3 Creating a Malicious OTA File
- Generate a firmware.bin file — its content can be simple, but we will use it to impersonate a legitimate OTA update!
~~~
echo "Malicious firmware simulation file" > firmware.bin
~~~

- Check the contents of the file.
~~~
cat firmware.bin
~~~
![image](https://github.com/user-attachments/assets/242a077f-e8a3-4095-9a8a-b9ac3d3ce2d4)

### 4.4 Generating the Hash Value of the Firmware (SHA-256)
~~~
sha256sum firmware.bin > firmware.hash
~~~

- The hash is a value used to verify that the file has not been tampered with during transmission.
- The hash value is saved in a file named firmware.hash.
- Check the contents of the file.
~~~
cat firmware.hash
~~~
![image](https://github.com/user-attachments/assets/122f4459-7047-4c74-a547-99168ff1634b)

### 4.5 Generating the Attacker’s Private Key
~~~
openssl genrsa -out attacker.key 2048
~~~

- Generate a private key that the attacker will use to sign the OTA package.
- This key allows the attacker to digitally “prove” — “I signed this file!” — by attaching a signature to the OTA file.
- Verify that the key was successfully created.
~~~
ls -l attacker.key
~~~
![image](https://github.com/user-attachments/assets/359e247f-abfb-4e79-ac60-1a6cf149f1c0)

### 4.6 Generating the Attacker’s Public Key
~~~
openssl rsa -in attacker.key -pubout -out attacker.pub
~~~

- The vehicle (client) uses only the public key to verify whether the signature is genuine!
- Check the contents of the public key file.
~~~
cat attacker.pub
~~~
![image](https://github.com/user-attachments/assets/aa2ce35d-933f-41d8-ad6e-ae0ebdb0f1a0)


### 4.7 Digitally Signing the Firmware File
~~~
openssl dgst -sha256 -sign attacker.key -out firmware.sig firmware.bin
~~~

- Sign the firmware.bin file using the attacker’s private key to generate the firmware.sig file.
- Verify that the signature file has been successfully created.
~~~
ls -l firmware.sig
~~~
![image](https://github.com/user-attachments/assets/b586b86c-8e5e-4852-9bec-51cf3caf2c36)

### 4.8 Verifying the Files Created So Far
~~~
ls -l
~~~
![image](https://github.com/user-attachments/assets/b7f5628e-5be2-4b4e-b4c5-59e6e7db806d)


### Conclusion
- All three files to be served by the OTA server are now ready:
  - firmware.bin
  - firmware.hash
  - firmware.sig
- The client will now:
  - Download firmware.bin
  - Verify it against the hash
  - Validate the signature
  - And mistakenly conclude, “This must be a legitimate OTA update.”

  - Malicious update is fully prepared ✔️
  - Attacker’s digital signature is in place ✔️

- Next step: Serve the files via the Flask OTA server, and create a client simulator to download and “install” the update.

## Step 5 : Running the Flask Server and Creating the Client Simulator
### 5.1 Launching the Flask OTA Server
- 1. Navigate to the server directory.
  - This is where the app.py file was previously created!
  ```
  cd ~/flask_ota_lab/server
  ```

- 2. Run the Flask server.
  ```
  sudo python3 app.py
  ```
  ![image](https://github.com/user-attachments/assets/94fd9717-6d28-44e9-beef-c8a0927ed369)

  - Why use sudo?
    - On Linux systems, port 443 (used for HTTPS) requires administrative privileges to bind.
  - At this point, your computer (the attacker’s laptop) is now functioning as an HTTPS-based OTA server!
- 3. Check the server status
  - Open a new terminal window while keeping the server running in the background.
  ```
  curl -k https://ota.realserver.com/firmware.bin --resolve ota.realserver.com:443:127.0.0.1
  ```
  ![image](https://github.com/user-attachments/assets/b0e5ebb4-a3ab-49b7-b059-cd932627b0ba)

  - Malicious firmware simulation file → Expected Output

### 5.2 Creating the Client Simulator
- 1. Navigate to the ota.client directory.
  ```
  cd ~/flask_ota_lab/ota_client
  ```

- 2. Create a file named client.simulator.py.
  ```
  nano client_simulator.py
  ```
  ![image](https://github.com/user-attachments/assets/e1b6f330-48fb-4ea5-9e10-314a25ddfa62)
  
  ```
  import requests
  import hashlib
  from cryptography.hazmat.primitives import hashes, serialization
  from cryptography.hazmat.primitives.asymmetric import padding
  
  # Server Address
  base_url = "https://ota.realserver.com"
  
  # File Download Function
  def download_file(filename):
      print(f"[+] Downloading {filename}...")
      r = requests.get(f"{base_url}/{filename}", verify=False)
      with open(filename, 'wb') as f:
          f.write(r.content)
      return r.content
  
  # 1. Download the OTA file
  firmware = download_file("firmware.bin")
  signature = download_file("firmware.sig")
  hash_expected = download_file("firmware.hash").decode().split()[0]
  
  # 2. Verify the hash
  hash_actual = hashlib.sha256(firmware).hexdigest()
  print(f"[✓] Expected hash: {hash_expected}")
  print(f"[✓] Actual hash:   {hash_actual}")
  
  if hash_expected != hash_actual:
      print("[✗] Hash mismatch! Aborting OTA installation.")
      exit()
  
  # 3. Verify the signature using the public key
  with open("attacker.pub", "rb") as f:
      public_key = serialization.load_pem_public_key(f.read())
  
  try:
      public_key.verify(
          signature,
          firmware,
          padding.PKCS1v15(),
          hashes.SHA256()
      )
      print("[✓] Signature is valid. OTA installation complete.")
  except Exception as e:
      print("[✗] Signature verification failed:", e)
  ```
  ![image](https://github.com/user-attachments/assets/a7059846-be30-48ed-898d-94e6f02e1a28)

- 3. Copy the public key
  ```
  cp ~/flask_ota_lab/updates/attacker.pub ~/flask_ota_lab/ota_client/
  ```

- 4. Run the simulator
  ```
  python3 client_simulator.py
  ```
  ![image](https://github.com/user-attachments/assets/4c319800-3063-4fae-b000-b092b4ea05cd)


### Conclusion
- We deployed a fake OTA server using Flask.
- We crafted a malicious OTA update package.
- We successfully tricked the client into installing it.

