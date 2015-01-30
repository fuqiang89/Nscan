# Nscan
Nscan is a fast Network scanner optimized for internet-wide scanning purposes inspired by Masscan and Zmap. It has it's own tiny TCP/IP stack and uses Raw sockets for sending TCP SYN probes. It doesn't need to set SYN Cookies so it doesn't wastes time checking if a received packet is a result of it's own scan, that makes Nscan faster than other similar scanners.

Nscan has a cool feature which allow you to extend your scan by chaining found ip:port to another scripts where it might checks for vulnerabilities, exploit targets, check for Proxies/VPNs... 

# Getting Nscan to work

Installing Nscan on Debian/Ubuntu boxes:
```
$ git clone https://github.com/OffensivePython/Nscan
$ cd Nscan/nscan
$ chmod +x nscan.py
```

Check if Nscan executes
```
$ ./nscan.py
Usage: 
nscan.py x.x.x.x/x [options]
nscan.py iface load/unload : Load/Unload Nscan alias interface
nscan.py resume filename.conf: resume previous scan


Options:
  -h, --help            show this help message and exit
  -p PORTS, --port=PORTS
                        Port(s) number (e.g. -p21-25,80)
  -t THREADS, --threads=THREADS
                        Threads used to send packets (default=1)
  --import=IMPORTS      Nscan scripts to import (e.g.
                        --import=ssh_key:22+check_proxy:80-85,8080)
  -b, --banner          Fetch banners
  -n COUNT              Number of results to get
  -o FILE, --output=FILE
                        Output file
  -c N,T, --cooldown=N,T
                        Every N (int) packets sent sleep P (float)
                        (Default=1000,1)
```

# Usage
Nscan is simple to use, it works just the way you expect it
First thing you need to do is to load nscan alias interface
```
$ ./nscan.py iface load
Press enter key to load nscan alias interface

[....] Running /etc/init.d/networking restart is deprecated because it may not [warnable some interfaces ... (warning).
[ ok ] Reconfiguring network interfaces...done.
Nscan alias interface loaded: 10.0.2.16
```
# Simple Scan
To scan your local network for port 22,80:
```
$ ./nscan.py 192.168.0.0/16 -p22,80

    _   __                    
   / | / /_____________ _____ 
  /  |/ / ___/ ___/ __ `/ __ \
 / /|  (__  ) /__/ /_/ / / / /
/_/ |_/____/\___/\__,_/_/ /_/ 
@OffensivePython             1.0
URL: https://github.com/OffensivePython/Nscan

Scanning [192.168.0.0 -> 192.169.0.0] (65536 hosts/1 ports)
[MAIN] Starting the scan (Fri Jan 30 07:11:02 2015)
...
```
This scans your 65535 hosts in your local network

Multithreading the scan:
-----------------------
use '-t' to specify how many sending thread you want to use, this decreases the elapsed time of the scan by n times:
```
$ ./nscan.py 192.168.0.0/16 -p3389,5900-5910 -t3 
```
This splits the 65535 hosts in 3 ranges (3 threads), every thread is going to scan 21845 host

Grabbing banners and save logs in a file:
----------------------------------------
use '-b' to grab banners and '-o' to save logs in a file
```
$ ./nscan.py 192.168.0.0/16 -p3389,5900-5910 -t3 -b -o nscan.log
```

Scanning to find N results:
----------------------------
In order to stop the scan after receiving 10 results:
```
$ ./nscan.py 192.168.0.0/16 -p443 -b -n10
```

Importing Nscripts:
-------------------
To import Nscripts, use '--import' with filename (without extension '.py') and specifiying the port or range of ports
```
$ ./nscan.py xxx.xxx.161.152/24 -p1080 --import=proxy:1080

    _   __                    
   / | / /_____________ _____ 
  /  |/ / ___/ ___/ __ `/ __ \
 / /|  (__  ) /__/ /_/ / / / /
/_/ |_/____/\___/\__,_/_/ /_/ 
@OffensivePython             1.0
URL: https://github.com/OffensivePython/Nscan

Scanning [xxx.xxx.161.152 -> xxx.xxx.162.0] (104 hosts/1 ports)
[MAIN] Starting the scan (Fri Jan 30 09:14:14 2015)
[SEND] Sent: 104 packets
[RECV] Received: 7 packets
[MAIN] xxx.xxx.161.152:1080
[MAIN] xxx.xxx.161.173:1080
[MAIN] xxx.xxx.161.195:1080
[MAIN] xxx.xxx.161.196:1080
[MAIN] xxx.xxx.161.194:1080
[MAIN] xxx.xxx.161.239:1080
[MAIN] xxx.xxx.161.193:1080
[PROXY] xxx.xxx.161.152:1080 | SOCKS4
[PROXY] xxx.xxx.161.195:1080 | SOCKS4
[PROXY] xxx.xxx.161.196:1080 | SOCKS4
[PROXY] xxx.xxx.161.194:1080 | SOCKS4
[PROXY] xxx.xxx.161.193:1080 | SOCKS4
[MAIN] Packets sent in 0.0 minutes
[MAIN] Total elapsed time: 0.7 minutes
[MAIN] Done (Fri Jan 30 09:14:58 2015)
```
Every ip has the port 1080 open, will be chained to the Nscript proxy, which checks if a SOCKS service is running behind it.
This will chains every ip:port that has the port 1080, and the range of ports [3127,3128,3129] to proxy script
```
$ ./nscan.py xxx.xxx.xxx.xxx/xx -p8080,1080,3127-3129 --import=proxy:1080,3127-3129
```
P.S: Port 8080 will not be chained to the script, since it's not specified


