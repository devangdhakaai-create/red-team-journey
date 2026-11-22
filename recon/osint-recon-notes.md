# OSINT Recon Notes — Week 1, Day 3

## Kya kiya (What we did)

Target domain: `tryhackme.com`

### Step 1: Sublist3r try kiya — fail ho gaya
Sublist3r ek purana tool hai jo Kali ke naye Python (3.14) version ke saath compatible nahi hai — isme multiprocessing wala internal bug hai (`TypeError: cannot pickle '_thread.lock' object`). Ye tool ki khud ki problem hai, humari command galat nahi thi.

**Lesson:** Purane/unmaintained tools kabhi-kabhi naye system pe break ho jaate hain. Time waste karke fix karne ke bajaye, turant **alternative dhoondo** — yahi real-world red teaming skill hai (tool pe depend mat raho, concept pe depend raho).

---

### Step 2: crt.sh use kiya (Certificate Transparency Logs)

```bash
curl -s "https://crt.sh/?q=%.tryhackme.com&output=json" | jq -r '.[].name_value' | sort -u > subdomains.txt
```

**Kya karta hai:** `crt.sh` public SSL certificate records search karta hai jahan bhi `tryhackme.com` likha hai — har certificate me domain names hote hain, isliye ye subdomains nikaalne ka ek bahut reliable free tareeka hai.

**Output:** ~100 subdomains mile, including kuch wildcard entries (jaise `*.vm.tryhackme.com` — ye koi real subdomain nahi hai, balki ek pattern hai jo bolta hai "is pattern ke andar koi bhi subdomain valid certificate rakhta hai").

---

### Step 3: Live subdomain check kiya

```bash
cat subdomains.txt | while read sub; do
  curl -s -o /dev/null -w "%{http_code} $sub\n" "https://$sub" --max-time 3
done
```

**Kya karta hai:** Har subdomain pe ek quick HTTPS request bhejta hai aur sirf status code print karta hai (bina pura page download kiye) — isse pata chalta hai kaunsa subdomain **live/active** hai aur kaunsa dead ya unreachable.

---

## Results — Status Code Samajhna (80/20 reference)

| Status Code | Matlab | Humara Example |
|---|---|---|
| **200** | Live, direct response milta hai | `auth.tryhackme.com`, `careers.tryhackme.com`, `tryhackme.com` |
| **301/302/307** | Redirect ho raha hai kahi aur | `business.tryhackme.com` (301), `blog.tryhackme.com` (302) |
| **401** | Live hai but authentication chahiye | `insights-proxy-worker.tryhackme.com` |
| **403** | Live hai but access forbidden | `assets.tryhackme.com`, `cdn-images.tryhackme.com` |
| **404** | Server live hai but ye specific path nahi milta | `click.tryhackme.com`, `resources.tryhackme.com` |
| **429** | Live hai but "too many requests" (rate-limited) | `desc.tryhackme.com`, `www.tryhackme.com` |
| **000** | Koi response nahi — dead, ya wildcard entry, ya firewall block | Saare `*.cell-prod-...` wildcard entries |

---

## Key Takeaways (80/20)

1. **Wildcard entries (`*.xxx.com`) real subdomains nahi hain** — inhe seedha curl se check nahi kar sakte, ye sirf certificate pattern hai. Isliye inpe "000" aana normal hai, iska matlab "dead" nahi hai.

2. **`403` aur `401` wale subdomains interesting hote hain** — matlab wahan kuch chal raha hai jo public access ke liye nahi hai (jaise internal tools, staging environments). Real engagement me ye further investigate karne layak targets hote hain.

3. **`429` ka matlab hai rate-limiting active hai** — agar tum bahut fast scan karoge, target block kar dega. Isliye real recon me scan speed control karna important hota hai.

4. **Subdomains jaise `cdn-images-staging.tryhackme.com`** bahut valuable hote hain — "staging" wale subdomains aksar less-secured/test environments hote hain jo production se kam protected hote hain.

5. **Tool fail hone pe panic mat karo** — Sublist3r fail hua, lekin crt.sh se same (balki better) result mil gaya. Red teaming me hamesha backup method pata hona chahiye.

---

## Conclusion
`tryhackme.com` ka subdomain footprint kaafi bada hai (100+ entries), jisme kai infrastructure subdomains (`vm.tryhackme.com` cells, reverse-proxies) company ke internal lab/VM infrastructure ko reveal karte hain — ye batata hai ki OSINT sirf "emails dhoondhna" nahi hai, balki **company ka pura attack surface map karna** hai.