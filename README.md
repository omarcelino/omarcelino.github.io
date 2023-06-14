# omarcelino.github.io
Test
### **Cheat Sheet**

The cheat sheet is a useful command reference for this module.

## **Scanning Options**

| Nmap Option | Description |
| --- | --- |
| 10.10.10.0/24 | Target network range. |
| -sn | Disables port scanning. |
| -Pn | Disables ICMP Echo Requests |
| -n | Disables DNS Resolution. |
| -PE | Performs the ping scan by using ICMP Echo Requests against the target. |
| --packet-trace | Shows all packets sent and received. |
| --reason | Displays the reason for a specific result. |
| --disable-arp-ping | Disables ARP Ping Requests. |
| --top-ports=<num> | Scans the specified top ports that have been defined as most frequent. |
| -p- | Scan all ports. |
| -p22-110 | Scan all ports between 22 and 110. |
| -p22,25 | Scans only the specified ports 22 and 25. |
| -F | Scans top 100 ports. |
| -sS | Performs an TCP SYN-Scan. |
| -sA | Performs an TCP ACK-Scan. |
| -sU | Performs an UDP Scan. |
| -sV | Scans the discovered services for their versions. |
| -sC | Perform a Script Scan with scripts that are categorized as "default". |
| --script <script> | Performs a Script Scan by using the specified scripts. |
| -O | Performs an OS Detection Scan to determine the OS of the target. |
| -A | Performs OS Detection, Service Detection, and traceroute scans. |
| -D RND:5 | Sets the number of random Decoys that will be used to scan the target. |
| -e | Specifies the network interface that is used for the scan. |
| -S 10.10.10.200 | Specifies the source IP address for the scan. |
| -g | Specifies the source port for the scan. |
| --dns-server <ns> | DNS resolution is performed by using a specified name server. |

## **Output Options**

| Nmap Option | Description |
| --- | --- |
| -oA filename | Stores the results in all available formats starting with the name of "filename". |
| -oN filename | Stores the results in normal format with the name "filename". |
| -oG filename | Stores the results in "grepable" format with the name of "filename". |
| -oX filename | Stores the results in XML format with the name of "filename". |

## **Performance Options**

| Nmap Option | Description |
| --- | --- |
| --max-retries <num> | Sets the number of retries for scans of specific ports. |
| --stats-every=5s | Displays scan's status every 5 seconds. |
| -v/-vv | Displays verbose output during the scan. |
| --initial-rtt-timeout 50ms | Sets the specified time value as initial RTT timeout. |
| --max-rtt-timeout 100ms | Sets the specified time value as maximum RTT timeout. |
| --min-rate 300 | Sets the number of packets that will be sent simultaneously. |
| -T <0-5> | Specifies the specific timing template. |

# ****Host and Port Scanning****

## ****Scanning Top 10 TCP Ports****

sudo nmap 10.129.91.32 --top-ports=10 -Pn
Starting Nmap 7.93 ( [https://nmap.org](https://nmap.org/) ) at 2023-03-25 19:18 CET
Nmap scan report for 10.129.91.32
Host is up (0.084s latency).

PORT     STATE  SERVICE
21/tcp   closed ftp
22/tcp   open   ssh
23/tcp   closed telnet
25/tcp   closed smtp
80/tcp   open   http
110/tcp  open   pop3
139/tcp  open   netbios-ssn
443/tcp  closed https
445/tcp  open   microsoft-ds
3389/tcp closed ms-wbt-server

We see that we only scanned the top 10 TCP ports of our target, and `Nmap` displays their state accordingly. If we trace the packets `Nmap` sends, we will see the `RST` flag on `TCP port 21` that our target sends back to us. To have a clear view of the SYN scan, we disable the ICMP echo requests (`-Pn`), DNS resolution (`-n`), and ARP ping scan (`--disable-arp-ping`).

## ****Nmap - Trace the Packets****

sudo nmap 10.129.91.32 -p 21 --packet-trace -Pn -n --disable-arp-ping
Starting Nmap 7.93 ( [https://nmap.org](https://nmap.org/) ) at 2023-03-25 19:24 CET
SENT (0.0594s) TCP 10.10.16.72:40598 > 10.129.91.32:21 S ttl=46 id=32790 iplen=44  seq=955433470 win=1024 <mss 1460>
RCVD (0.0887s) TCP 10.129.91.32:21 > 10.10.16.72:40598 RA ttl=63 id=6215 iplen=40  seq=0 win=0
Nmap scan report for 10.129.91.32
Host is up (0.030s latency).

PORT   STATE  SERVICE
21/tcp closed ftp

****Connect Scan****

The `Connect` scan is useful because it is the most accurate way to determine the state of a port, and it is also the most stealthy

```
sudo nmap 10.129.91.32 -p 443 --packet-trace --disable-arp-ping -Pn -n --reason -sT
Starting Nmap 7.93 ( [https://nmap.org](https://nmap.org/) ) at 2023-03-25 19:28 CET
CONN (0.0240s) TCP localhost > 10.129.91.32:443 => Operation now in progress
CONN (0.0537s) TCP localhost > 10.129.91.32:443 => Connection refused
Nmap scan report for 10.129.91.32
Host is up, received user-set (0.030s latency).

PORT    STATE  SERVICE REASON
443/tcp closed https   conn-refused
```

****Filtered Ports****

When a port is shown as filtered, it can have several reasons. In most cases, firewalls have certain rules set to handle specific connections. The packets can either be `dropped`, or `rejected`.herefore we scan the TCP port **139**, which was already shown as filtered. To be able to track how our sent packets are handled, we deactivate the ICMP echo requests (`-Pn`), DNS resolution (`-n`), and ARP ping scan (`--disable-arp-ping`) again.

```
sudo nmap 10.129.2.28 -p 139 --packet-trace -n --disable-arp-ping -PnStarting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:45 CEST
SENT (0.0381s) TCP 10.10.14.2:60277 > 10.129.2.28:139 S ttl=47 id=14523 iplen=44  seq=4175236769 win=1024 <mss 1460>
SENT (1.0411s) TCP 10.10.14.2:60278 > 10.129.2.28:139 S ttl=45 id=7372 iplen=44  seq=4175171232 win=1024 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up.

PORT    STATE    SERVICE
139/tcp filtered netbios-ssn
```

## ****Discovering Open UDP Ports****

we cannot determine if the UDP packet has arrived at all or not. If the UDP port is `open`, we only get a response if the application is configured to do so.

```
sudo nmap 10.129.91.32 -F -sU
Starting Nmap 7.93 ( [https://nmap.org](https://nmap.org/) ) at 2023-03-25 19:38 CET
Nmap scan report for 10.129.91.32
Host is up (0.031s latency).
Not shown: 96 closed udp ports (port-unreach)
PORT    STATE         SERVICE
68/udp  open|filtered dhcpc
137/udp open          netbios-ns
138/udp open|filtered netbios-dgm
999/udp open|filtered applix
```

```
sudo nmap 10.129.91.32 -sU -Pn -n --disable-arp-ping --packet-trace -p 137 --reason
Starting Nmap 7.93 ( [https://nmap.org](https://nmap.org/) ) at 2023-03-25 19:44 CET
SENT (0.0651s) UDP 10.10.16.72:46910 > 10.129.91.32:137 ttl=38 id=3405 iplen=78
SENT (0.0652s) UDP 10.10.16.72:46910 > 10.129.91.32:137 ttl=37 id=3405 iplen=78
SENT (0.0652s) UDP 10.10.16.72:46910 > 10.129.91.32:137 ttl=40 id=3405 iplen=78
SENT (1.0675s) UDP 10.10.16.72:46912 > 10.129.91.32:137 ttl=53 id=35904 iplen=78
SENT (1.0676s) UDP 10.10.16.72:46912 > 10.129.91.32:137 ttl=40 id=35904 iplen=78
SENT (1.0676s) UDP 10.10.16.72:46912 > 10.129.91.32:137 ttl=37 id=35904 iplen=78
RCVD (1.0977s) UDP 10.129.91.32:137 > 10.10.16.72:46912 ttl=63 id=38478 iplen=221
Nmap scan report for 10.129.91.32
Host is up, received user-set (0.030s latency).
PORT    STATE SERVICE    REASON
137/udp open  netbios-ns udp-response ttl 63
Nmap done: 1 IP address (1 host up) scanned in 1.13 seconds
```

If we get an ICMP response with `error code 3` (port unreachable), we know that the port is indeed `closed`

```
sudo nmap 10.129.91.32 -sU -Pn -n --disable-arp-ping --packet-trace -p 100 --reason 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-25 19:47 CET
SENT (0.0572s) UDP 10.10.16.72:55077 > 10.129.91.32:100 ttl=44 id=38311 iplen=28 
SENT (1.0597s) UDP 10.10.16.72:55079 > 10.129.91.32:100 ttl=55 id=19812 iplen=28 
RCVD (1.0888s) ICMP [10.129.91.32 > 10.10.16.72 Port unreachable (type=3/code=3) ] IP [ttl=63 id=60519 iplen=56 ]
Nmap scan report for 10.129.91.32
Host is up, received user-set (0.029s latency).

PORT    STATE  SERVICE REASON
100/udp closed unknown port-unreach ttl 63
```

```
sudo nmap 10.129.91.32 -Pn -n --disable-arp-ping --packet-trace -p 445 -sV --reason 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-25 19:49 CET
SENT (0.4323s) TCP 10.10.16.72:55978 > 10.129.91.32:445 S ttl=55 id=58924 iplen=44  seq=3893306529 win=1024 <mss 1460>
RCVD (0.4629s) TCP 10.129.91.32:445 > 10.10.16.72:55978 SA ttl=63 id=0 iplen=44  seq=925850261 win=29200 <mss 1338>
NSOCK INFO [0.5930s] nsock_iod_new2(): nsock_iod_new (IOD #1)
NSOCK INFO [0.5930s] nsock_connect_tcp(): TCP connection requested to 10.129.91.32:445 (IOD #1) EID 8
NSOCK INFO [0.6250s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 8 [10.129.91.32:445]
Service scan sending probe NULL to 10.129.91.32:445 (tcp)
NSOCK INFO [0.6250s] nsock_read(): Read request from IOD #1 [10.129.91.32:445] (timeout: 6000ms) EID 18
NSOCK INFO [6.6320s] nsock_trace_handler_callback(): Callback: READ TIMEOUT for EID 18 [10.129.91.32:445]
Service scan sending probe SMBProgNeg to 10.129.91.32:445 (tcp)
NSOCK INFO [6.6320s] nsock_write(): Write request for 168 bytes to IOD #1 EID 27 [10.129.91.32:445]
NSOCK INFO [6.6320s] nsock_read(): Read request from IOD #1 [10.129.91.32:445] (timeout: 5000ms) EID 34
NSOCK INFO [6.6320s] nsock_trace_handler_callback(): Callback: WRITE SUCCESS for EID 27 [10.129.91.32:445]
NSOCK INFO [6.7370s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 34 [10.129.91.32:445] (135 bytes)
Service scan match (Probe SMBProgNeg matched with SMBProgNeg line 13836): 10.129.91.32:445 is netbios-ssn.  Version: |Samba smbd|3.X - 4.X|workgroup: WORKGROUP|
NSOCK INFO [6.7370s] nsock_iod_delete(): nsock_iod_delete (IOD #1)
Nmap scan report for 10.129.91.32
Host is up, received user-set (0.031s latency).

PORT    STATE SERVICE     REASON         VERSION
445/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
Service Info: Host: NIX-NMAP-DEFAULT
```

# ****Questions****

1. Find all TCP ports on your target. Submit the total number of found TCP ports as the answer.
    
    *7*
    

```
sudo nmap 10.129.91.32 -Pn -n --disable-arp-ping
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-25 20:03 CET
Nmap scan report for 10.129.91.32
Host is up (0.041s latency).
Not shown: 993 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
110/tcp   open  pop3
139/tcp   open  netbios-ssn
143/tcp   open  imap
445/tcp   open  microsoft-ds
31337/tcp open  Elite
```

1. Enumerate the hostname of your target and submit it as the answer. (case-sensitive)
    
    *NIX-NMAP-DEFAULT*
    
    ```
    sudo nmap 10.129.91.32 -Pn -n --disable-arp-ping -p 445 -sV --reason 
    Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-25 19:59 CET
    Nmap scan report for 10.129.91.32
    Host is up, received user-set (0.030s latency).
    
    PORT    STATE SERVICE     REASON         VERSION
    445/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
    Service Info: Host: **NIX-NMAP-DEFAULT**
    
    Service detection performed. Please report any incorrec
    ```
    
    # ****Nmap Scripting Engine****
    
    | Category | Description |
    | --- | --- |
    | auth | Determination of authentication credentials. |
    | broadcast | Scripts, which are used for host discovery by broadcasting and the discovered hosts, can be automatically added to the remaining scans. |
    | brute | Executes scripts that try to log in to the respective service by brute-forcing with credentials. |
    | default | Default scripts executed by using the -sC option. |
    | discovery | Evaluation of accessible services. |
    | dos | These scripts are used to check services for denial of service vulnerabilities and are used less as it harms the services. |
    | exploit | This category of scripts tries to exploit known vulnerabilities for the scanned port. |
    | external | Scripts that use external services for further processing. |
    | fuzzer | This uses scripts to identify vulnerabilities and unexpected packet handling by sending different fields, which can take much time. |
    | intrusive | Intrusive scripts that could negatively affect the target system. |
    | malware | Checks if some malware infects the target system. |
    | safe | Defensive scripts that do not perform intrusive and destructive access. |
    | version | Extension for service detection. |
    | vuln | Identification of specific vulnerabilities. |
    
    # ****Firewall and IDS/IPS Evasion****
    
    ```
    sudo nmap 10.129.57.246 -p 21,22,25 -sS -Pn -n --disable-arp-ping --packet-trace
    Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-25 21:37 CET
    SENT (0.0782s) TCP 10.10.16.72:49339 > 10.129.57.246:25 S ttl=44 id=1761 iplen=44  seq=1900857882 win=1024 <mss 1460>
    SENT (0.0785s) TCP 10.10.16.72:49339 > 10.129.57.246:22 S ttl=49 id=36076 iplen=44  seq=1900857882 win=1024 <mss 1460>
    SENT (0.0786s) TCP 10.10.16.72:49339 > 10.129.57.246:21 S ttl=59 id=65231 iplen=44  seq=1900857882 win=1024 <mss 1460>
    RCVD (0.1087s) TCP 10.129.57.246:25 > 10.10.16.72:49339 RA ttl=63 id=53125 iplen=40  seq=0 win=0 
    RCVD (0.1823s) TCP 10.129.57.246:22 > 10.10.16.72:49339 SA ttl=63 id=0 iplen=44  seq=3554978192 win=29200 <mss 1338>
    RCVD (0.1824s) TCP 10.129.57.246:21 > 10.10.16.72:49339 RA ttl=63 id=53132 iplen=40  seq=0 win=0 
    Nmap scan report for 10.129.57.246
    Host is up (0.048s latency).
    
    PORT   STATE  SERVICE
    21/tcp closed ftp
    22/tcp open   ssh
    25/tcp closed smtp
    ```
    
    # **Decoys**
    
    There are cases in which administrators block specific subnets from different regions in principle. This prevents any access to the target network. Another example is when IPS should block us. For this reason, the Decoy scanning method (`-D`) is the right choice. With this method, Nmap generates various random IP addresses inserted into the IP header to disguise the origin of the packet sent.
    
    # ****Testing Firewall Rule****
    
    ```
    sudo nmap 10.129.2.28 -n -Pn -p445 -O
    ```
    
    ## ****Scan by Using Different Source IP****
    
    ```
    sudo nmap 10.129.2.28 -n -Pn -p 445 -O -S 10.129.2.200 -e tun0
    ```
    
    # ****DNS Proxying****
    
    ## ****SYN-Scan of a Filtered Port****
    
    ```
    sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace
    
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 22:50 CEST
    SENT (0.0417s) TCP 10.10.14.2:33436 > 10.129.2.28:50000 S ttl=41 id=21939 iplen=44  seq=736533153 win=1024 <mss 1460>
    SENT (1.0481s) TCP 10.10.14.2:33437 > 10.129.2.28:50000 S ttl=46 id=6446 iplen=44  seq=736598688 win=1024 <mss 1460>
    Nmap scan report for 10.129.2.28
    Host is up.
    
    PORT      STATE    SERVICE
    50000/tcp filtered ibm-db2
    
    Nmap done: 1 IP address (1 host up) scanned in 2.06 seconds
    ```
    
    ## ****SYN-Scan From DNS Port****
    
    ```
    sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53
    
    SENT (0.0482s) TCP 10.10.14.2:53 > 10.129.2.28:50000 S ttl=58 id=27470 iplen=44  seq=4003923435 win=1024 <mss 1460>
    RCVD (0.0608s) TCP 10.129.2.28:50000 > 10.10.14.2:53 SA ttl=64 id=0 iplen=44  seq=540635485 win=64240 <mss 1460>
    Nmap scan report for 10.129.2.28
    Host is up (0.013s latency).
    
    PORT      STATE SERVICE
    50000/tcp open  ibm-db2
    MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
    
    Nmap done: 1 IP address (1 host up) scanned in 0.08 seconds
    ```
    
    Now that we have found out that the firewall accepts `TCP port 53`, it is very likely that IDS/IPS filters might also be configured much weaker than others. We can test this by trying to connect to this port by using `Netcat`
    .
    
    ****Connect To The Filtered Port****
    
    ```
    ncat -nv --source-port 53 10.129.2.28 50000Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Connected to 10.129.2.28:50000.
    220 ProFTP
    ```
    
    # **Firewall and IDS/IPS Evasion - Easy Lab**
    
    ---
    
    Now let's get practical. A company hired us to test their IT security defenses, including their `IDS` and `IPS` systems. Our client wants to increase their IT security and will, therefore, make specific improvements to their `IDS/IPS` systems after each successful test. We do not know, however, according to which guidelines these changes will be made. Our goal is to find out specific information from the given situations.
    
    We are only ever provided with a machine protected by `IDS/IPS` systems and can be tested. For learning purposes and to get a feel for how `IDS/IPS` can behave, we have access to a status web page at:
    
    # ****Questions****
    
    1. Our client wants to know if we can identify which operating system their provided machine is running on. Submit the OS name as the answer.
        
        Ubuntu
        
    
    ```
    sudo nmap 10.129.2.80 -p 80 -sS -Pn -sV -n --disable-arp-ping --packet-trace -D RND:5
    Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-25 22:41 CET
    PORT   STATE SERVICE VERSION
    80/tcp open  http    Apache httpd 2.4.18 ((**Ubuntu**))
    ```
    
    # **Firewall and IDS/IPS Evasion - Medium Lab**
    
    ---
    
    After we conducted the first test and submitted our results to our client, the administrators made some changes and improvements to the `IDS/IPS` and firewall. We could hear that the administrators were not satisfied with their previous configurations during the meeting, and they could see that the network traffic could be filtered more strictly.
    
    ****Questions****
    
    1. After the configurations are transferred to the system, our client wants to know if it is possible to find out our target's DNS server version. Submit the DNS server version of the target as the answer.
    
    |  |  |
    | --- | --- |
    |  |  |
    |  |  |
