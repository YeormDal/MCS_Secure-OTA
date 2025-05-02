# Attack Scenario 1: Capture OTA webpage to leak and tamper with files

## Attack scenario prerequisites
Assume that the designated proxy server of the computer (device) used by the administrator to upload firmware has been changed to the attacker's proxy server through hacking.

### How to perform the attack scenario
1. Assign the IP address of the proxy server of the computer that will perform the update file upload to the attacker's IP address.
![Screenshot 2025-05-01 123313](https://github.com/user-attachments/assets/d3703274-8741-4da6-a8a5-2a7a4606fa31)

2. Capture the packets transmitted to the server when performing a file upload on the web using a tool such as burf. ![Screenshot 2025-05-01 123507](https://github.com/user-attachments/assets/930ba93e-8924-4cd4-8327-fa45cc5df02c)

3. Transmit the unencrypted file contents from the captured packets by modifying them.
![Screenshot 2025-05-01 123546](https://github.com/user-attachments/assets/e5ada10a-4d60-4428-8153-cecf15f13a24)

4. Install the malicious file on vehicles without authentication. ![Screenshot 2025-05-01 123714](https://github.com/user-attachments/assets/894f1c63-f3e4-4071-a484-b5f54897da9c)
