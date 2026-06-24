# 🌐 Network Fundamentals & Systems Administration Documentation

Ushbu hujjat tarmoq asoslari, protokollar, amaliy masalalar (troubleshooting) va backend/tizim administratorlari intervyularida eng ko'p so'raladigan savollar to'plamini o'z ichiga oladi.

---

## 📌 Mundarija
1. [OSI va TCP/IP Modellari](#1-osi-va-tcpip-modellari)
2. [IP Manzillash va Subnetting](#2-ip-manzillash-va-subnetting)
3. [Tarmoq Protokollari (HTTP, DNS, SSH, TLS)](#3-tarmoq-protokollari)
4. [Port Forwarding va Tunneling (Ngrok, Cloudflare, JPRQ)](#4-port-forwarding-va-tunneling)
5. [Amaliy Masalalar va Yechimlar (Troubleshooting)](#5-amaliy-masalalar-va-yechimlar)
6. [Intervyuda Tushadigan Top Savollar](#6-intervyuda-tushadigan-top-savollar)

---

## 1. OSI va TCP/IP Modellari

Tarmoq orqali ma'lumot uzatishni tushunish uchun ushbu ikki model poydevor hisoblanadi.

### OSI Modeli (7 ta Qatlam)
*   **7. Application (Ilova):** Foydalanuvchi interfeysi (HTTP, FTP, SSH, DNS).
*   **6. Presentation (Taqdimot):** Shifrlash, siqish va ma'lumot formati (SSL/TLS, JSON, JPEG).
*   **5. Session (Seans):** Aloqani o'rnatish va boshqarish (RPC, NetBIOS).
*   **4. Transport (Transport):** Ma'lumotni yetkazib berish (TCP - ishonchli, UDP - tezkor).
*   **3. Network (Tarmoq):** Routing va IP manzillash (IP, ICMP, Routers).
*   **2. Data Link (Kanal):** MAC manzillar bilan ishlash, freymlar (Ethernet, Switches).
*   **1. Physical (Fizik):** Kabellar, signallar va bitlar (Bits, Cables, Hubs).

### TCP/IP Modeli (4 ta Qatlam)
OSI modelining soddalashtirilgan zamonaviy ko'rinishi:
1.  **Application** (OSI 5, 6, 7 ni birlashtiradi)
2.  **Transport** (OSI 4)
3.  **Internet** (OSI 3)
4.  **Network Access** (OSI 1, 2 ni birlashtiradi)

---

## 2. IP Manzillash va Subnetting

### IP Klaslari (IPv4)
*   **Class A:** `1.0.0.0` - `127.255.255.255` (Mask: `255.0.0.0` / `/8`)
*   **Class B:** `128.0.0.0` - `191.255.255.255` (Mask: `255.255.0.0` / `/16`)
*   **Class C:** `192.0.0.0` - `223.255.255.255` (Mask: `255.255.255.0` / `/24`)

### Private (Ichki) IP Manzillar
Lokal tarmoq (LAN) uchun ajratilgan, internetda routing bo'lmaydigan manzillar:
*   `10.0.0.0` – `10.255.255.255`
*   `172.16.0.0` – `172.31.255.255`
*   `192.168.0.0` – `192.168.255.255`

> **Eslatma:** `127.0.0.1` — Loopback (Localhost) manzili bo'lib, kompyuterning o'ziga murojaat qilish uchun ishlatiladi.

---

## 3. Tarmoq Protokollari

### HTTP va HTTPS
*   **HTTP (Port 80):** Shifrlanmagan, ochiq matn ko'rinishida ma'lumot uzatadi.
*   **HTTPS (Port 443):** HTTP + SSL/TLS. Ma'lumotlar shifrlangan holda uzatiladi.

### DNS (Domain Name System) — Port 53
Domen nomlarini (masalan, `google.com`) IP manzillarga (`142.250.190.46`) o'girib beruvchi tizim.
*   **A Record:** Domenni IPv4 manzilgan bog'laydi.
*   **AAAA Record:** Domenni IPv6 manzilgan bog'laydi.
*   **CNAME:** Bitta domenni ikkinchi domenga (alias) yo'naltiradi.
*   **MX Record:** Pochta serverlarini aniqlaydi.

### SSH (Secure Shell) — Port 22
Masofadagi Linux serverlarni xavfsiz boshqarish protokoli. Odatda xavfsizlik uchun port `22` dan boshqa ixtiyoriy portga o'zgartiriladi.

---

## 4. Port Forwarding va Tunneling

Mahalliy kompyuterda (localhost) ishlayotgan loyihalarni tashqi dunyoga (internetga) ochish yoki lokal serverni boshqarish usullari.

### 1. Port Forwarding (Yo'naltirish)
Ruter (Router) sozlamalariga kirib, tashqi IP'ga kelgan so'rovni lokal tarmoqdagi ma'lum bir ichki IP va portga yo'naltirish (Masalan: `Router:8080` -> `Lokal Server:80`). Oq (Public) IP talab qilinadi.

### 2. Tunneling Xizmatlari (Oq IP bo'lmagan holatda)
Agar provayderingiz sizga statik oq IP bermagan bo'lsa, quyidagi instrumentlar orqali lokal serverni internetga chiqarish mumkin:

*   **Ngrok:** `ngrok http 8000` — lokal 8000-portni xavfsiz URL orqali internetga ochadi.
*   **Cloudflare Tunnels:** Eng xavfsiz va professional usul. Kontentni Cloudflare proxy orqali o'tkazadi va wildcard subdomenlarni tekinga sozlash imkonini beradi.
*   **JPRQ / LocalTunnel:** Ngrok'ka muqobil, ochiq kodli tunneling yechimlari.

---

## 5. Amaliy Masalalar va Yechimlar

### 📝 Masala 1: Wildcard Subdomenlarni Lokal Serverga Yo'naltirish
**Muammo:** Sizda ko'p ijarali (Multi-tenant) Laravel loyiha bor. Har bir mijoz uchun `mijoz1.localhost`, `mijoz2.localhost` ko'rinishida subdomenlar kerak. Nginx buni qanday qabul qiladi?

**Yechim:**
1. Linux'da `/etc/hosts` fayliga faqat aniq domenlarni yozish mumkin, wildcard (`*.localhost`) ishlamaydi. Shuning uchun `dnsmasq` o'rnatiladi:
   ```bash
   sudo apt install dnsmasq
   echo "address=/.localhost/127.0.0.1" | sudo tee -a /etc/dnsmasq.conf
   sudo systemctl restart dnsmasq
   ```
2. Nginx Server Block (VirtualHost) sozlamasida `server_name` qismiga wildcard yoziladi:
   ```nginx
   server {
       listen 80;
       server_name *.localhost;
       root /var/www/my-laravel-project/public;
       index index.php;
       # ... qolgan Laravel sozlamalari
   }
   ```

### 📝 Masala 2: Serverga Ping Borayapti, Lekin Sayt Ochilmayapti
**Muammo:** Terminalda `ping my-server.com` qilganda javob kelyapti, lekin brauzerda `ERR_CONNECTION_REFUSED` xatoligi chiqyapti. Muammo nimada?

**Yechim qadamlari:**
1. Ping `ICMP` protokolida ishlaydi. Ping borayotgan bo'lsa, demak Tarmoq qatlami (Network layer) va IP to'g'ri ishlayapti.
2. Brauzer `HTTP (80)` yoki `HTTPS (443)` portlariga ulanishga urunadi. Xatolik ushbu portlar yopiqligini bildiradi.
3. Serverga SSH orqali kirib, Nginx/Apache ishlayotganini tekshirish kerak:
   ```bash
   sudo systemctl status nginx
   ```
4. Agar Nginx ishlayotgan bo'lsa, Linux Firewall (UFW) portlarni bloklagan bo'lishi mumkin. Portlarni ochish:
   ```bash
   sudo ufw allow 80/tcp
   sudo ufw allow 443/tcp
   ```

---

## 6. Intervyuda Tushadigan Top Savollar

### ❓ S1: Brauzerga `google.com` deb yozib Enter'ni bossak nima sodir bo'ladi?
**Javob:**
1. **DNS qidiruv:** Brauzer birinchi bo'lib lokal keshdan, keyin `/etc/hosts` faylidan, so'ng DNS serverdan (masalan, 8.8.8.8) `google.com` ning IP manzilini so'raydi.
2. **TCP Handshake:** IP topilgach, transport qatlamida server bilan 3 bosqichli aloqa (`SYN` -> `SYN-ACK` -> `ACK`) o'rnatiladi.
3. **TLS Handshake:** Agar ulanish HTTPS bo'lsa, xavfsizlik sertifikatlari almashiladi va shifrlangan kanal ochiladi.
4. **HTTP So'rov:** Brauzer serverga `GET / HTTP/1.1` so'rovini yuboradi.
5. **Server ishlovi:** Server (Nginx/Apache) so'rovni qabul qiladi, backend (PHP, Go) kodni ishlatadi va brauzerga `HTML/CSS/JS` qaytaradi.
6. **Rendering:** Brauzer kelgan kodni vizual ko'rinishga keltirib foydalanuvchiga ko'rsatadi.

### ❓ S2: TCP va UDP protokollarining farqi nimada va qachon qaysi biri ishlatiladi?
**Javob:**
*   **TCP (Transmission Control Protocol):** Ulanishga asoslangan (Connection-oriented). Ma'lumot paketlari yetib borganini tekshiradi, agar paket yo'qolsa qayta yuboradi. Tartibni saqlaydi. *Ishlatilishi:* HTTP, SSH, FTP, Database ulanishlari (ishonchlilik muhim bo'lgan joyda).
*   **UDP (User Datagram Protocol):** Ulanish o'rnatmaydi (Connectionless). Paket yetib bordimi, yo'qmi tekshirmaydi, shunchaki juda tez yuboradi. *Ishlatilishi:* Video aloqa (Zoom, Telegram calls), Onlayn o'yinlar, Live-striming (tezlik muhim, 1-2 ta kadr yo'qolsa fojia emas).

### ❓ S3: Reverse Proxy va Forward Proxy farqi nimada?
**Javob:**
*   **Forward Proxy (Klassik Proxy):** Mijoz (Client) tomonida turadi. Mijozning IP manzilini yashirib, internetga proxy orqali chiqishini ta'minlaydi (Masalan: VPN yoki kompaniya ichidagi cheklovchi proxy).
*   **Reverse Proxy:** Server tomonida turadi. Mijoz qaysi backend serverga murojaat qilayotganini bilmaydi. Nginx kelgan so'rovlarni qabul qilib, ichki tarmoqdagi turli portlarda ishlayotgan loyihalarga (masalan, PHP-FPM yoki Go-app'ga) tarqatadi. Load Balancing va SSL sertifikatlarni boshqarish uchun juda qulay.

### ❓ S4: Tarmoq trafigini va portlarni ko'rish uchun Linux'da qaysi komandalar ishlatiladi?
**Javob:**
*   `netstat -tuln` yoki `ss -tuln` — Serverda hozirda qaysi portlar tinglanyapti (Listen) va ishlayapti.
*   `curl -I http://localhost:8000` — Lokal portdagi saytning HTTP Header javobini tekshirish.
*   `tcpdump` — Tarmoq interfeysidan o'tayotgan real paketlarni tutib olish va tahlil qilish (Packet sniffer).
*   `traceroute google.com` — Paket Google serverigacha nechta router (hoplari) orqali o'tayotganini ko'rsatuvchi komanda.
