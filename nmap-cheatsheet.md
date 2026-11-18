# Nmap Cheatsheet — Week 1

## 1. Full TCP Port Scan (sabse pehla step)
```bash
nmap -sS -p- --min-rate=1000 <target-ip>
```
**Purpose:** Saare 65535 TCP ports fast speed me check karna ki kaunse open hain.
- `-sS` = stealth SYN scan (full handshake complete nahi karta, kam noisy)
- `-p-` = sabhi ports (1-65535)
- `--min-rate=1000` = scan ko fast force karna

**Example output:** 11 high ports (48xxx range) "open" mile, baaki sab filtered.

---

## 2. Service + Version Detection (open ports pe hi chalao)
```bash
nmap -sC -sV -p<port> <target-ip>
```
**Purpose:** Pata karna ki open port pe exactly kaunsi service/version chal rahi hai.
- `-sC` = default safe NSE scripts chalata hai
- `-sV` = service banner/version grab karta hai

**Note:** Agar result "filtered" aaye jabki pehle scan me "open" tha, firewall rate-limiting ya intermittent service ho sakti hai — dobara scan karo confirm karne ke liye.

---

## 3. UDP Scan (top 20 common ports)
```bash
sudo nmap -sU --top-ports 20 <target-ip>
```
**Purpose:** UDP services (DNS, SNMP, NetBIOS, etc.) check karna — ye TCP scan me nahi dikhte.
- `sudo` zaroori hai kyunki raw sockets access chahiye
- UDP scan slow hota hai, isliye `--top-ports` se sirf common ones check karo pehle

**Example output:** Saare 20 ports "closed" aaye — matlab target in services pe listen nahi kar raha (ya firewall block kar raha).

---

## 4. NSE Script Scan (targeted enumeration)
```bash
nmap --script=http-enum -p80 <target-ip>
```
**Purpose:** Specific service (yahan HTTP) ke baare me deeper info nikaalna — hidden directories, common files, etc.
- `--script=<name>` = ek specific NSE script run karo instead of full `-sC` set

**Note:** Port 80 bhi "filtered" aaya — matlab firewall HTTP traffic ko bhi block/limit kar raha hai is network pe.

---

## Key takeaway (80/20)
- `-sS -p-` → sabse pehle **kya khula hai** pata karo
- `-sC -sV` → open ports pe **kya chal raha hai** pata karo
- `-sU` → mat bhoolo, UDP bhi ek attack surface hai
- `--script=` → specific service ko deeper enumerate karo
- **Result mismatch (open → filtered) → ek baar dobara scan karke confirm karo, firewall/rate-limiting common hai**

