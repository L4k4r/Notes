**Link-Local Multicast Name Resolution** (LLMNR) and **NetBIOS Name Service** (NBT-NS) are Microsoft Windows components that serve as alternate methods of host identification that can be used when DNS fails. If a machine attempts to resolve a host but DNS resolution fails, typically, the machine will try to ask all other machines on the local network for the correct host address via LLMNR.

The kicker here is that when LLMNR/NBT-NS are used for name resolution, ANY host on the network can reply. This is where we come in with `Responder` to poison these requests.

---
### On Linux

Start Responder
```bash
sudo responder -I ens224 
```

Hashes are saved in `/usr/share/responder/logs`

Crack the hash with Hashcat (`-m 5600 = NTLMv2`)
```bash
hashcat -m 5600 <hash> /usr/share/wordlists/rockyou.txt 
```

---
### On Windows

We can use a tool called Inveigh

Start the tool
```powershell
.\inveigh.exe
```

We can press `esc` to enter/exit interactive console.
We can quickly view unique captured hashes by typing `GET NTLMV2UNIQUE`.
We can type in `GET NTLMV2USERNAMES` and see which usernames we have collected.

---
### **Key Limitation:**

- This attack **does not work if the user does not attempt to access a non-existent resource**.
- If the network is properly configured with **DNS, WPAD security, and SMB signing**, the effectiveness of this attack is reduced.

To enhance the attack, **you can actively trick users** into making requests by:

- Sending **phishing emails** with links to nonexistent SMB shares.
- Deploying **Rogue WPAD (Web Proxy Auto-Discovery)** responses to force authentication.
- Performing **MITM attacks** to redirect traffic.