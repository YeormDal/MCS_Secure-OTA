# 3. Replay Attack

## Attack Scenario 1: Reuse Previous Update Packet

### Precondition
Assume that the vehicle is connected to the fake MQTT broker. So the attacker can try to update malware at anytime
### Attack Prcedure
1. Capture and store normal packets from the another OTA target.
2. Assuming that vulnerabilities in the past version have been fixed by the new version, update the past version of the file through a fake MQTT broker.
3. The vehicle installs the packet due to insufficient message freshness verification.
---
