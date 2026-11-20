# Service Enumeration Notes — Week 1, Day 2

## 1. SMB (Port 445/139)

**Commands:**
```bash
nmap -sC -sV -p445 <target-ip>
smbclient -L //<target-ip>/ -N
enum4linux -a <target-ip>
```

**Generally kya milta hai:**
- `smbclient -L` → available shares ki list (e.g., `ADMIN$`, `C$`, `IPC$`, ya koi custom share)
- `enum4linux -a` → domain/workgroup name, OS version, user list (agar null session allowed hai), share permissions

**80/20 Takeaway:**
> Agar `-N` (null/anonymous) se shares list ho jaate hain ya koi share **bina password ke access** ho jaata hai, ye ek critical misconfiguration hai — SMB anonymous access real engagements me bahut common finding hai aur aksar sensitive files/credentials leak karta hai.

---

## 2. FTP (Port 21)

**Commands:**
```bash
nmap -sC -sV -p21 <target-ip>
ftp <target-ip>
# Username: anonymous | Password: blank or anything
```

**Generally kya milta hai:**
- Agar anonymous login allowed hai → FTP server ke andar files/folders dikhte hain
- `ls` se directory listing, `get <file>` se download

**80/20 Takeaway:**
> **Anonymous FTP login = instant red flag.** Agar ye allowed hai, pehla kaam hota hai directory listing dekhna — config files, backup files, ya credentials files aksar yahi pade milte hain.

---

## 3. SSH (Port 22)

**Commands:**
```bash
nmap -sC -sV -p22 <target-ip>
```

**Generally kya milta hai:**
- SSH version banner (e.g., `OpenSSH 7.2p2`)
- Supported auth methods (password, publickey)

**80/20 Takeaway:**
> SSH khud directly exploit karne layak kam hi milta hai (jab tak koi known CVE na ho purane version me). Isliye SSH recon ka main kaam hai **version note karna** — baad me `searchsploit <version>` se check karoge ki koi known vulnerability hai ya nahi. SSH ka real use aksar baad me hota hai — jab tumhe already credentials mil chuke hon (via other vulns) toh SSH se login karke access lena.

---

## 4. HTTP (Port 80/8080)

**Commands:**
```bash
nmap -sC -sV -p80 <target-ip>
gobuster dir -u http://<target-ip> -w /usr/share/wordlists/dirb/common.txt
```

**Generally kya milta hai:**
- `nmap -sC -sV` → web server type/version (Apache, nginx, IIS), kabhi-kabhi page title
- `gobuster` → hidden directories/files (e.g., `/admin`, `/backup`, `/uploads`, `/.git`)

**80/20 Takeaway:**
> HTTP sabse bada attack surface hota hai kyunki yahi se web app vulnerabilities (Week 2 ka topic) shuru hote hain. `gobuster`/directory brute-force se mile hidden paths aksar **login panels, backup files, ya admin interfaces** reveal karte hain — jo aage exploitation ka entry point bante hain.

---

## Overall Day 2 — 80/20 Summary

| Service | #1 Thing to Check | Why It Matters |
|---|---|---|
| SMB | Anonymous/null session shares | Files/creds leak hona common hai |
| FTP | Anonymous login | Direct file access mil sakta hai |
| SSH | Version banner | Known CVE check ke liye, exploitation baad me |
| HTTP | Hidden directories | Web app attack surface yahi se khulta hai |

**Core lesson:** Enumeration ka goal "service chal rahi hai" confirm karna nahi hai — goal hai **misconfiguration ya weak spot dhoondna** jo exploitation ka raasta khole. Har service ka apna ek "default weak spot" hota hai, aur wahi pehle check karo.
