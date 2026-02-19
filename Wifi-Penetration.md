# Wifi penetration Theeury and in-depth

----

## wifi encryption standards


| Protocol                                | Status      | Security Level | Authentication Method                      | Encryption             | Integrity             | Extra                                          |
| :-------------------------------------- | :---------- | :------------- | :----------------------------------------- | :--------------------- | :-------------------- | :--------------------------------------------- |
| 802.11 (Legacy – Pre-RSN)               | Deprecated  | Very Weak      | Open System or Shared Key                  | None or WEP (RC4)      | None or CRC-32 (weak) | Pre-WPA era security; fully broken today.      |
| WEP                                     | Deprecated  | Very Weak      | Open System or Shared Key                  | RC4 (40/104-bit)       | CRC-32                | Easily cracked in minutes.                     |
| WPA                                     | Deprecated  | Weak           | WPA-PSK or 802.1X                          | TKIP (uses RC4)        | MIC (Michael)         | TKIP vulnerable. Replaced by WPA2              |
| WPA2-Personal                           | Active and Widely Used | Strong         | Pre-Shared Key (PSK)                       | AES-CCMP               | CCMP                  | Vulnerable to weak passwords & KRACK (patched) |
| WPA3-Personal                           | Current     | Very Strong    | SAE (Simultaneous Authentication of Equals)| AES-GCMP (128-bit)     | GCMP                  | Protects against brute-force; forward secrecy  |
| WPA2-Enterprise                         | Active      | Very Strong           | 802.1X with RADIUS (EAP methods)           | AES-CCMP               | CCMP                  | Per-user credentials; used in enterprises      |
| WPA3-Enterprise                         | Current     | Extremely Strong     | 802.1X with RADIUS (192-bit mode optional) | AES-GCMP-256           | GCMP-256              | Designed for high-security environments        |
| Open Networks                           | Active      | None           | None                                       | None                   | None                  | Traffic fully visible to others                |
| Opportunistic Wireless Encryption (OWE) | Active      | Moderate (No authentication)| None (no authentication)                   | AES-GCMP               | GCMP                  | Encrypts traffic but no identity verification "MITM attacks" |

Notes: The 802.11 letters (b/g/n/ac/ax) refer to Wi-Fi generations/speeds, not security.

----

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

> airmon-ng -> airodump-ng -> aireplay-ng -> airodump-ng -> generate Hash -> Hashcat/cowpatty/john/aircrack-ng -> **Great Success**

----


# Aircrack-ng Toolkit

----

the toolbox of multiple network tools 
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
sudo airodump-ng wlan0mon -c 1 -w captured_file
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
sudo aireplay-ng -0 5 -a BSSID -c ESSID wlan0mon
```
Use the Deauth method : `-0`
Send 5 packets : `5`
The Access point Targeted's BSSID `-a BSSID`
The Client which is deauthenticated: `-c ESSID`
The network interface used: `wlan0mon`

Now the handshake will take place and airodump-ng will capture it and store it.

## Aircrack-ng

## Airograph-ng



----


# hashcat

----

## Before cracking
Before cracking the airodump-ng capture the file, we must make it into a format hashcat can work with and understand.
Use hcxpcapngtool to create the hash which can be used by hashcat.
```
hcxpcapngtool -o hash_of_password captured_file.cap
```
The captured file : `captured_file.cap`
the output is saved in hash_of_password:`-o hash_of_password`
The generated new file is now in a hash format hashcat can work with

## Hashcat Commands
Hashcat is a videly used tool for many types of cracking not only for wifi.

### Basics
Command
```
hashcat -m 22000 HASH WORDLIST
```

| Fucntions  | Functionality | example |
| :--------- | :------------ | :------ |
| `-m X`     | Specify Hash type X, can be left blank and hashcat will attempt to guess scheme | -m 22000 |
| `-a X`     | Specifies Attack Mode ( See Attack Modes section) | -a 3 |
| `HASH`   |Specifies the hash which is being worked on to carck | |


### Attack Modes
#### Attack Mode 1 Combinator 
Command Example
```
hashcat -a 1 -m 22000 HASH WORDLIST1 WORDLIST2
```

Attack Mode 1, Combinator, will take two wordlists as input and combine each item on the lists
the first given wordlist will be the prefix and second provided will be the surfix 

**Example:**

| Wordlist 1 word | Wordlist 2 word |
|:---|:---|
|hello1 | super1 
|world2 | man2 |

This will result in: 
- hello1super1 
- hello1man2
- world2super1
- world2man2 
**Total words used**
If 2 words are in wordlist 1 and 2 words are in wordlist 2 it would be:
`2 x 2 = 4` 
Creating 4 words in total, which is used.

If wordlist 1 has 3 words and wordlist 2 has 2 it would be:
`3 x 2 = 6` 
Creating 6 words in total, which is used.
#### Attack Mode 2 

#### Attack Mode 3 Mask 
Example:
```
hashcat -a 3 -m 22000 HASH 'MASK'
```
Example with Boundaries (the mask provided must be max amount long)
```
hashcat -a 3 -m 22000 HASH --increment --increment-min 8 --increment-max=14 'MAXIMUMMASKS'
```

A mask attack is bruteforcing each valid chacter on each spot, setting a minimum and maximum can be usefull when unsure of exact length.
The valid characters is specified by the mask layout, the first command specifies first charcters valid characters, the second command speficies the second characeters valid characters and etc.
See mask examples if confused.
**Masks Are:**
|  Command   | Description  |
|   :--      |    :--                |
|    `?l`    |Lowercase ASCII letters ( a b c d ... )|
|    `?u`    |Uppercase ASCII letters ( A B C D ... )|
|    `?d`    |Digits ( 0-9 ) |
|    `?h`    |Digits and lowercase ASCII letters ( ` ?l + ?d = ?h ` )|
|    `?H`    |Digits and Uppercase ASCII letters ( ` ?u + ?d = ?H ` )|
|    `?s`    |Special Characters ( Space , . ? # ... ) |
|    `?a`    |Digits,Special characters, Lower- & Uppercase ASCII letters ( ` ?l + ?u + ?d + ?s = ?a ` )|
|    `?b`    |All posible byte values |

The options `?a` or `?l`, `?u`, `?d`, & `?s`, make up the full range of 95 printable ASCII chacters.

**Mask Examples**
Example 1 : ` '?l?l?l?l?l' ` this will bruteforce lowercase letters on each possition, allways creating a 1-5 charcter long input used for cracking. A valid input would be **"abcde"**
Example 2 : ` '?l?l?l?l?l' --incement-min 3 ` will bruteforce lowercase letters on each possition and create 3-5 character long inputs used for cracking A valid input would be **"abc"**, **"abcd"**, and **"abcde"**
Example 3 : ` '?d?l?l?l?l' ` This will create possibilities up to a 5 chacter long string, every combination will start with a digit (?d) and then placement 2-5 will be lowercase ascii letters. A valid input would be **"1abcd"** and **"2abcd"**




#### Attack Mode 4 

#### Attack Mode 5 

#### Attack Mode 6 Hybrid: Wordlist + Mask
Example:
```
hashcat -a 6 -m 22000 HASH WORDLIST 'MASK'
```
The hybdri mode will use the word from the worlist and combine it with a mask.
It looks like this
**Word + Mask = Result**
The resault will then be used to crack the hash.
Each word will go through every combination of the mask, meaning the word will be combined with a mask combination 1 and a mask combination 2.
See example of combination
**Example of combination**
Example:
The first word from the wordlist is `hello`
The mask is ` '?d?d?l' `
One of the results would be `hello12a` another would be `hello12b` a third would be `hello12c`, a fourth `hello99x` and so on.
when the mask has reached all combinations which is 10x10x26=2600. 2600 combinations with word 1, then there will be 2600 combinations with word 2 and so fourth.

To calculate just times each masks valid characters. etc 
?d (0-9)       = 10 valid characters
?l (a-z)       = 26 valid characters
?u (A-Z)       = 26 valid characters
?h (0-9 + a-z) = 39 valid characters
?H (0-9 + A-Z) = 36 valid characters
?a             = 95 valid characters.
?s             = 33 valid characters.

TBH i dont know ?s :D.
But should be 33 by explaination of ?à, ?a is ?l + ?u + ?d + ?s.
so by that logic the logically Calculation of ?s should be: 33
**Logic of reasoning**
  ?l + ?u + ?d + ?s = ?a     -> -?s on both sides 
  ?l + ?u + ?d = ?a - ?s     -> -?a on both sides  
  ?l + ?u + ?d - ?a = - ?s   -> *-1 on both sides to get positive ?s
  ?l - ?u - ?d + ?a = ?s     -> Change order for overview
  ?a - ?l - ?u - ?d = ?s     -> Lets get the total that is subtracted from ?a
  ?a - (?l + ?u + ?d) = ?s   -> Put in the numbers for each variable known
  95 - (26 + 26 + 10) = ?s   -> Add together to find what needs to be subtracted to ?a
  95 - 62 = ?s               -> the amount of ?l + ?u + ?d is 62 valid options. subtract it from total 95 valid characters of ?a
  33 = ?s                    == Left with 33 possible valid combinations left in ?a which means ?s must be 33 valid characters.



#### Attack Mode 7 Hybrid: Mask + Wordlist
Example:
```
hashcat -a 6 -m 22000 HASH WORDLIST 'MASK'
```
The hybdri mode 7 will use the mask and combine it with the words from the worlist.
Example if the word from wordlist was "hello", and the mask was "`'?d?d?l' `"
One of the results would be `12ahello`


### Advanced and Tailored

| Fucntions  | Functionality | example |
| `-I`       | Give a list of available CPU and GPU devices | -I |
| `-D X`     | Speifies what type of device used either 1 GPU or 2 CPU| -D 1 |
| `-d X`     | Specifies specific device ( Used -I to find devices ) | -d 8 |








----


# The Cheatsheet

----

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
sudo aireplay-ng -0 5 -a BSSID -c ESSID wlan0mon
```

**Check for handshake**
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
**Hashcat w/ wordlist on WPA Encryption**
```
hashcat -m 22000 HASH WORDLIST   
```
### **Hashcat Attack Modes**
**Hashcat -a 1 w/ Combinator: 2 wordlists on WPA Encryption**
```
hashcat -a 1 -m 22000 HASH WORDLIST WORDLIST2
```

**Hashcat -a 2 w/ on WPA Encryption**

**Hashcat -a 3 w/ Mask on WPA Encryption**
```
hashcat -a 3 -m 22000 HASH 'MASK'
```

**Masks:**
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


**Hashcat -a 3 w/ Mask W/ Boundary on WPA Encryption**
```
hashcat -a 3 -m 22000 HASH --increment --increment-min 8 --increment-max=14 'MAXIMUMMASKS'
```
**Hashcat -a 4 w/ on WPA Encryption**

**Hashcat -a 5 w/ on WPA Encryption**


**Hashcat -a 6 w/ Hybrid: Wordlist + Mask on WPA Encryption**
```
hashcat -a 6 -m 22000 HASH WORDLIST 'MASK'
```
**Hashcat -a 7 w/ Hybrid: Mask + Wordlist on WPA Encryption**
```
hashcat -a 7 -m 22000 HASH WORDLIST 'MASK'
```















