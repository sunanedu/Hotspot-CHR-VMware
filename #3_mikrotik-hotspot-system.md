# กลุ่มที่ 3 — ระบบ Hotspot โดยตรง
### สำหรับผู้เรียนที่เริ่มต้นจากศูนย์ | ต่อยอดจากกลุ่มที่ 1 และ 2

---

> **📌 ข้อกำหนดก่อนเรียน**
> ก่อนเรียนกลุ่มนี้ต้องผ่านการตั้งค่าพื้นฐานของกลุ่มที่ 2 มาแล้ว โดยเฉพาะ:
> - Interface WAN/LAN ตั้งค่าเรียบร้อย
> - DHCP Server ทำงานบน LAN
> - NAT Masquerade ใช้งานได้
> - PC/มือถือเชื่อมต่อผ่าน LAN แล้วออกอินเทอร์เน็ตได้
>
> ถ้ายังไม่ผ่านขั้นตอนด้านบน ให้กลับไปทำกลุ่มที่ 2 ก่อน เพราะ Hotspot ต้องการพื้นฐานเหล่านั้นทั้งหมด

---

## บทที่ 17 — Hotspot ทำงานอย่างไร?

---

### 17.1 Hotspot คืออะไร?

**Hotspot** (อ่านว่า ฮ็อต-สปอต) ในบริบทของ MikroTik คือ **ระบบควบคุมการใช้งานอินเทอร์เน็ต**
ที่บังคับให้ผู้ใช้ต้อง **Login ก่อนถึงจะใช้งานได้**

สิ่งที่ Hotspot ทำได้:
- บังคับ Login ด้วย Username / Password
- จำกัดเวลาใช้งาน (เช่น ซื้อ 1 ชั่วโมง หมดเวลาแล้วหลุด)
- จำกัดปริมาณ Data (เช่น ซื้อ 1 GB หมดแล้วหลุด)
- จำกัดความเร็ว (เช่น 10 Mbps ต่อคน)
- บันทึก Log การใช้งานตามกฎหมาย

---

### 17.2 Hotspot vs Router ธรรมดา

| คุณสมบัติ | Router ธรรมดา | MikroTik Hotspot |
|----------|--------------|-----------------|
| การเข้าใช้งาน | เชื่อมต่อ Wi-Fi แล้วใช้งานได้เลย | ต้อง Login ก่อน |
| ควบคุมผู้ใช้ | ไม่ได้ | ได้ ทุกคน |
| จำกัดเวลา | ไม่ได้ | ได้ ต่อ User |
| จำกัด Speed | ยาก | ง่าย ต่อ User |
| บันทึก Log | ไม่ได้ | ได้ ละเอียด |
| เหมาะกับ | บ้านพักอาศัย | ร้านค้า, โรงแรม, ISP |

---

### 17.3 ภาพรวมการทำงานของ Hotspot

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ลูกค้าเชื่อมต่อ Wi-Fi (หรือสาย LAN)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
           │
           ↓
DHCP แจก IP ให้ลูกค้า (192.168.88.x)
           │
           ↓
ลูกค้าเปิดเบราว์เซอร์ พิมพ์ www.google.com
           │
           ↓
MikroTik Hotspot ดักจับ Request
  "คนนี้ยังไม่ได้ Login!"
           │
           ↓
Redirect (เปลี่ยนเส้นทาง) ไปหน้า Login
  http://192.168.88.1/login
           │
           ↓
ลูกค้ากรอก Username / Password
           │
           ↓ ถูก ✓          ↓ ผิด ✗
      อนุญาต           แสดงข้อผิดพลาด
      ออกเน็ตได้        ให้กรอกใหม่
           │
           ↓
MikroTik เปิด "ประตู" ให้ IP นั้น
ลูกค้าออกอินเทอร์เน็ตได้ตามสิทธิ์ที่กำหนด
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### 17.4 User Manager vs Built-in Hotspot

MikroTik มีระบบ Hotspot 2 แบบ:

**แบบที่ 1 — Built-in Hotspot ← แนะนำสำหรับมือใหม่**
- ระบบ Hotspot ที่ติดมากับ RouterOS
- ตั้งค่าง่าย จัดการ User ได้โดยตรงจาก Winbox
- เหมาะกับร้านค้า, Hotspot ทั่วไป, ขนาดเล็ก-กลาง

**แบบที่ 2 — User Manager (ยู-เซอร์ แมน-เน-เจอร์)**
- ระบบ RADIUS Server (เรเดียส เซิร์ฟ-เวอร์) ที่ซับซ้อนกว่า
- จัดการ User หลายพัน, หลายหมื่น User ได้ง่าย
- มี Web Interface สำหรับขายบัตรเน็ต
- เหมาะกับ ISP, โรงแรม, อาคารขนาดใหญ่

> 📌 **เอกสารนี้จะเน้นที่ Built-in Hotspot** ซึ่งเหมาะกับผู้เริ่มต้น
> User Manager จะอธิบายในระดับกลาง-สูงต่อไป

---

## บทที่ 18 — การสร้าง Hotspot Profile

---

### 18.1 Hotspot Profile คืออะไร?

**Profile** (อ่านว่า โพร-ไฟล์) แปลว่า "โปรไฟล์" หรือ "ชุดการตั้งค่า"

ใน MikroTik Hotspot มี Profile 2 ระดับ:

```
1. Hotspot Server Profile (โปรไฟล์ระดับ Server)
   ── ตั้งค่าภาพรวมของระบบ Hotspot ทั้งหมด
   ── เช่น หน้า Login, DNS, Walled Garden

2. Hotspot User Profile (โปรไฟล์ระดับ User)
   ── ตั้งค่าสำหรับกลุ่ม User
   ── เช่น ความเร็ว, เวลาใช้งาน, Data Limit
```

---

### 18.2 การสร้าง Hotspot ด้วย Wizard (แนะนำสำหรับมือใหม่)

MikroTik มี Hotspot Setup Wizard ที่ทำให้ตั้งค่าได้ง่ายมาก:

```
Winbox → IP → Hotspot → กด "Hotspot Setup"
```

**ขั้นตอนการ Setup:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ขั้น 1: Hotspot Interface
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
เลือก Interface ที่ลูกค้าเชื่อมต่อ
→ เลือก ether2 หรือ bridge1 (LAN Interface)
→ ห้ามเลือก WAN Interface (ether1)!
→ กด "Next"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ขั้น 2: Local Address of Network
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
IP ของ MikroTik ในฝั่ง LAN
→ ปกติจะกรอกมาให้แล้ว เช่น 192.168.88.1/24
→ ติ๊ก "Masquerade Network" ✓
→ กด "Next"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ขั้น 3: Address Pool of Network
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ช่วง IP ที่แจกให้ลูกค้า
→ เช่น 192.168.88.10-192.168.88.254
→ กด "Next"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ขั้น 4: Select Certificate
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SSL Certificate สำหรับ HTTPS Login
→ เลือก "none" สำหรับมือใหม่
→ (ตั้ง HTTPS ทีหลังได้)
→ กด "Next"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ขั้น 5: IP Address of SMTP Server
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SMTP = เซิร์ฟเวอร์ส่งอีเมล
→ ปล่อยว่างไว้ก่อน (0.0.0.0)
→ กด "Next"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ขั้น 6: DNS Configuration
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
→ ปกติจะกรอกมาให้แล้ว
→ ตรวจสอบว่ามี DNS เช่น 8.8.8.8
→ กด "Next"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ขั้น 7: DNS Name
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ชื่อ Domain ของ Hotspot Login
→ ตัวอย่าง: hotspot.myshop.com
→ หรือปล่อยว่างเพื่อใช้ IP แทน
→ กด "Next"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ขั้น 8: Create Local Hotspot User
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
สร้าง User แรกสำหรับทดสอบ
→ Name: admin (หรือชื่อใดก็ได้)
→ Password: ตั้งรหัส
→ กด "Next"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ขั้น 9: Hotspot Setup has completed ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

> ⚠️ **Note:** หลังจาก Setup เสร็จ MikroTik จะเพิ่ม Firewall Rules อัตโนมัติ
> ห้าม Delete Rules เหล่านั้น เพราะ Hotspot จำเป็นต้องใช้

---

### 18.3 Hotspot Server Profile — ตั้งค่าละเอียด

หลัง Setup เสร็จสามารถแก้ไข Profile ได้:

```
IP → Hotspot → แท็บ "Server Profiles" → ดับเบิ้ลคลิกที่ Profile
```

**แท็บ General:**
```
Name: hsprof1 (ชื่อ Profile)
HTML Directory: hotspot (โฟลเดอร์หน้า Login)
HTTP Cookie Validity: 3d (จำ Login ไว้ 3 วัน)
Login By:
    ✓ HTTP PAP (กรอก Username/Password)
    □ MAC Address (Login ด้วย MAC อัตโนมัติ)
    □ HTTP CHAP (เข้ารหัสรหัสผ่าน)
```

**Login By — อธิบายเพิ่มเติม:**

| วิธี Login | ความหมาย | ใช้เมื่อไหร่ |
|-----------|---------|------------|
| HTTP PAP | กรอก User/Pass แบบธรรมดา | ทั่วไป (มือใหม่) |
| HTTP CHAP | กรอก User/Pass แบบเข้ารหัส | ปลอดภัยกว่า PAP |
| MAC Address | Login อัตโนมัติด้วย MAC | อุปกรณ์ประจำ ไม่ต้องกรอก |
| Trial | ใช้ฟรีได้เองโดยไม่ต้อง Login | เปิดให้ทดลองใช้ฟรีระยะสั้น |

> 💡 **Tip:** สำหรับร้านค้าทั่วไปให้ใช้ **HTTP CHAP** เพราะเข้ารหัสรหัสผ่านระหว่างส่ง ปลอดภัยกว่า PAP แต่ยังกรอก Username/Password เหมือนกัน

---

### 18.4 Hotspot User Profile — สร้างแพ็กเกจบริการ

**User Profile** คือ "แพ็กเกจ" ที่กำหนดสิทธิ์และข้อจำกัดของ User

```
IP → Hotspot → แท็บ "User Profiles" → กด "+"
```

ตัวอย่างการสร้าง Profile สำหรับแพ็กเกจต่าง ๆ:

**แพ็กเกจ "Free" — ฟรี ความเร็วต่ำ:**
```
Name: free
Rate Limit (rx/tx): 1M/1M
Session Timeout: 00:30:00 (30 นาที)
Shared Users: 1 (ใช้ได้ 1 เครื่องพร้อมกัน)
```

**แพ็กเกจ "Basic" — เสียเงิน ความเร็วปานกลาง:**
```
Name: basic
Rate Limit (rx/tx): 5M/5M
Session Timeout: 02:00:00 (2 ชั่วโมง)
Shared Users: 1
```

**แพ็กเกจ "Premium" — ความเร็วสูง:**
```
Name: premium
Rate Limit (rx/tx): 20M/20M
Session Timeout: 08:00:00 (8 ชั่วโมง)
Shared Users: 3 (ใช้ได้ 3 เครื่องพร้อมกัน)
```

> 💡 **Tip เรื่อง Rate Limit:** รูปแบบการใส่คือ `rx/tx` หมายถึง:
> - **rx** (อ่านว่า อาร์-เอ็กซ์) = Receive = ความเร็ว **ดาวน์โหลด** (ลูกค้าได้รับข้อมูล)
> - **tx** (อ่านว่า ที-เอ็กซ์) = Transmit = ความเร็ว **อัปโหลด** (ลูกค้าส่งข้อมูล)
> - `5M/5M` = ดาวน์โหลด 5 Mbps, อัปโหลด 5 Mbps

---

## บทที่ 19 — การสร้าง User / Password สำหรับลูกค้า

---

### 19.1 Hotspot User คืออะไร?

**Hotspot User** คือบัญชีผู้ใช้ที่ลูกค้าจะนำไป Login เข้าระบบ

ข้อมูลของ User ประกอบด้วย:
- **Username** (ยู-เซอร์-เนม) = ชื่อผู้ใช้
- **Password** (พาส-เวิร์ด) = รหัสผ่าน
- **Profile** = แพ็กเกจที่ใช้ (ความเร็ว, เวลา, Data)
- **Limit Uptime** = จำกัดเวลาใช้งานรวม
- **Limit Bytes** = จำกัดปริมาณ Data

---

### 19.2 สร้าง User แบบ Manual (ทีละคน)

```
IP → Hotspot → แท็บ "Users" → กด "+"
```

กรอกข้อมูล:
```
Name: user001          ← Username ที่ลูกค้าใช้ Login
Password: pass1234     ← รหัสผ่าน
Profile: basic         ← เลือก Profile ที่สร้างไว้

[ถ้าต้องการจำกัดเพิ่มเติม]
Limit Uptime: 02:00:00         ← ใช้ได้รวม 2 ชั่วโมง
Limit Bytes Total: 1073741824  ← ใช้ได้ 1 GB (คำนวณเป็น Bytes)
```

**การคำนวณ Bytes:**
```
1 KB = 1,024 Bytes
1 MB = 1,048,576 Bytes
1 GB = 1,073,741,824 Bytes

ตัวอย่าง:
500 MB = 524,288,000 Bytes
1 GB   = 1,073,741,824 Bytes
2 GB   = 2,147,483,648 Bytes
```

> 💡 **Tip:** ถ้าไม่ต้องการจำกัด Data ให้ปล่อยช่อง Limit Bytes ว่างไว้ ระบบจะไม่จำกัด

---

### 19.3 สร้าง User หลายคนพร้อมกัน (Generate)

ถ้าต้องการสร้าง User เยอะ ๆ เช่น บัตรเน็ต 50 ใบพร้อมกัน:

```
IP → Hotspot → แท็บ "Users" → กด "Generate"
```

ตั้งค่า:
```
Count: 50                  ← จำนวน User ที่ต้องการ
Name Prefix: user          ← Prefix ชื่อ (เช่น user001, user002...)
Name Length: 6             ← ความยาว Username
Password Length: 6         ← ความยาว Password
Profile: basic             ← Profile ที่ใช้
Limit Uptime: 02:00:00     ← เวลา
```

กด "Generate" → MikroTik จะสร้าง User user001 ถึง user050 พร้อม Password สุ่มให้อัตโนมัติ

> 💡 **Tip:** หลัง Generate แล้วสามารถ Print หรือ Export เป็น CSV ได้ เพื่อนำไปพิมพ์บัตรเน็ตให้ลูกค้า

---

### 19.4 การตั้งค่า User แบบ MAC Address Binding

**MAC Address Binding** (แมค แอด-เดรส ไบน์-ดิง) คือการผูก User กับอุปกรณ์
ทำให้ลูกค้า Login แล้ว อุปกรณ์เครื่องนั้นจะจำไว้โดยไม่ต้อง Login ซ้ำ

```
IP → Hotspot → แท็บ "Users" → ดับเบิ้ลคลิกที่ User
    → ช่อง "MAC Address" → ใส่ MAC ของอุปกรณ์ลูกค้า
```

> ⚠️ **Note:** ถ้าใส่ MAC Address ไว้ User นั้นจะ Login ได้เฉพาะจากอุปกรณ์ที่กำหนดเท่านั้น ถ้าลูกค้าเปลี่ยนมือถือหรือเปลี่ยนเครื่อง จะ Login ไม่ได้

---

### 19.5 การดู User ทั้งหมดและการจัดการ

**ดู User ทั้งหมด:**
```
IP → Hotspot → แท็บ "Users"
```

**ฟิลด์สำคัญในตาราง:**
| ฟิลด์ | ความหมาย |
|------|---------|
| Name | Username |
| Profile | แพ็กเกจ |
| Uptime | เวลาใช้งานสะสม |
| Bytes In | Data ที่ดาวน์โหลด |
| Bytes Out | Data ที่อัปโหลด |
| Last Logged Out | ออกจากระบบครั้งล่าสุด |

**ลบ User:**
```
คลิกที่ User → กด "-" (ลบ)
หรือ คลิกขวา → Remove
```

**Reset เวลาและ Data ของ User:**
```
คลิกขวาที่ User → Reset Counters
```

> ❓ **คำถามทิ้งไว้:** ถ้าลูกค้า Login ด้วย User เดิมจากมือถือ 2 เครื่องพร้อมกัน จะเกิดอะไรขึ้น? ค่า Shared Users ที่กำหนดใน Profile มีผลอย่างไร?

---

## บทที่ 20 — Walled Garden คืออะไร?

---

### 20.1 Walled Garden คืออะไร?

**Walled Garden** (อ่านว่า วอลล์ด การ์-เด้น) แปลตรงตัวว่า "สวนที่มีกำแพง"

ในบริบท Hotspot หมายถึง: **รายชื่อเว็บไซต์หรือ IP ที่ลูกค้าเข้าได้โดยไม่ต้อง Login**

**ทำไมต้องมี Walled Garden?**

ลองนึกสถานการณ์นี้:
- ลูกค้าเชื่อมต่อ Wi-Fi ร้านคุณ
- ยังไม่ได้ Login
- ลูกค้าต้องการเช็คราคาสินค้าบนเว็บร้านคุณ
- แต่ Hotspot บล็อกเว็บทุกอย่างก่อน Login!

ด้วย Walled Garden คุณสามารถกำหนดให้ **เว็บของร้าน** เข้าได้ก่อน Login

**ตัวอย่างการใช้งาน Walled Garden:**
- เว็บไซต์ร้านค้าของคุณ (ลูกค้าดูสินค้าได้ก่อน Login)
- หน้า Facebook Fanpage (เพื่อการตลาด)
- ระบบชำระเงิน (ลูกค้าจ่ายเงินเพื่อซื้อ Session)
- เว็บ Terms of Service (ข้อตกลงการใช้งาน)

---

### 20.2 ประเภทของ Walled Garden

**แบบที่ 1 — Walled Garden (IP-based)**

กำหนดด้วย IP Address หรือ Subnet ของเว็บที่ต้องการ:
```
IP → Hotspot → แท็บ "Walled Garden IP List" → กด "+"
    → Action: accept
    → Dst. Address: 203.150.x.x/24  ← IP ของเว็บปลายทาง
```

**แบบที่ 2 — Walled Garden (Domain-based)**

กำหนดด้วยชื่อ Domain (ง่ายกว่าแบบแรก):
```
IP → Hotspot → แท็บ "Walled Garden" → กด "+"
    → Action: allow
    → Server: * (ทุก Hotspot Server)
    → Dst. Host: *.facebook.com    ← Domain ที่ต้องการเปิด
```

ตัวอย่าง Domain ที่ควรเพิ่มใน Walled Garden:
```
*.facebook.com          ← Facebook ทุก Subdomain
*.line.me               ← LINE
*.google.com            ← Google
*.googleapis.com        ← Google APIs (จำเป็นสำหรับ Android)
*.gstatic.com           ← Google Static Files
*.apple.com             ← Apple (iOS)
*.icloud.com            ← iCloud
```

> 💡 **Tip:** ใส่ `*` (เครื่องหมาย Asterisk / แอส-เทอ-ริสค์) นำหน้า Domain เพื่อครอบคลุม Subdomain ทั้งหมด เช่น `*.google.com` จะครอบคลุม `www.google.com`, `mail.google.com`, `maps.google.com` ทั้งหมด

> ⚠️ **Note:** ถ้าใส่ Walled Garden มากเกินไป ลูกค้าอาจเข้าถึงอินเทอร์เน็ตได้บางส่วนโดยไม่ต้อง Login ทำให้รายได้หาย ใส่เฉพาะที่จำเป็นจริง ๆ

> ❓ **คำถามทิ้งไว้:** ถ้าเว็บ Google เปลี่ยน IP Address Walled Garden แบบ IP-based จะยังทำงานได้ไหม? แบบ Domain-based ล่ะ? อันไหนดีกว่ากัน?

---

## บทที่ 21 — การปรับแต่งหน้า Login Page

---

### 21.1 หน้า Login Page คืออะไร?

**Login Page** (อ่านว่า โล-กิน เพจ) คือหน้าเว็บที่ลูกค้าเห็นเมื่อ MikroTik Redirect มา
เป็นหน้า HTML ธรรมดาที่เก็บอยู่ใน Flash Memory ของ MikroTik

ไฟล์ Login Page อยู่ที่:
```
Files → โฟลเดอร์ hotspot/
    ├── login.html        ← หน้า Login หลัก
    ├── logout.html       ← หน้าหลัง Logout
    ├── status.html       ← หน้าแสดงสถานะ
    ├── error.html        ← หน้าแสดงข้อผิดพลาด
    ├── alogin.html       ← หน้า Login สำเร็จ
    └── radvert.html      ← หน้าโฆษณา (ถ้ามี)
```

---

### 21.2 การดาวน์โหลดและแก้ไข Login Page

**ขั้นตอน:**

```
ขั้น 1: Download ไฟล์ Login Page จาก MikroTik
    Files → hotspot/ → คลิกขวาที่ login.html → Download

ขั้น 2: แก้ไขด้วย Text Editor
    ใช้ Notepad++, VS Code, หรือ Sublime Text
    (ห้ามใช้ Microsoft Word หรือ WordPad)

ขั้น 3: Upload กลับไปยัง MikroTik
    Files → hotspot/ → ลากไฟล์เข้าไปใน Files
    หรือกด "Upload" แล้วเลือกไฟล์
```

---

### 21.3 ตัวแปรพิเศษใน Login Page

MikroTik มีตัวแปร (Variable) พิเศษที่ใช้ใน HTML ได้:

| ตัวแปร | ความหมาย | ตัวอย่างที่แสดง |
|--------|---------|--------------|
| `$(username)` | Username ที่กรอก | user001 |
| `$(ip)` | IP ของลูกค้า | 192.168.88.10 |
| `$(mac)` | MAC Address ของลูกค้า | AA:BB:CC:DD:EE:FF |
| `$(session-time-left)` | เวลาที่เหลือ | 01:30:00 |
| `$(bytes-in-nice)` | Data ดาวน์โหลด | 150.5 MB |
| `$(bytes-out-nice)` | Data อัปโหลด | 20.3 MB |
| `$(link-login)` | URL สำหรับ Login | http://... |
| `$(link-logout)` | URL สำหรับ Logout | http://... |
| `$(error)` | ข้อความ Error | Wrong username or password |

---

### 21.4 ตัวอย่าง Login Page อย่างง่าย

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>เข้าสู่ระบบ Wi-Fi</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f0f0f0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        .login-box {
            background: white;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            width: 320px;
            text-align: center;
        }
        h2 { color: #333; }
        input {
            width: 100%;
            padding: 10px;
            margin: 8px 0;
            border: 1px solid #ddd;
            border-radius: 5px;
            box-sizing: border-box;
        }
        button {
            width: 100%;
            padding: 12px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
        }
        .error { color: red; margin-top: 10px; }
    </style>
</head>
<body>
    <div class="login-box">
        <h2>🌐 ยินดีต้อนรับ</h2>
        <p>ร้าน My Coffee Shop</p>

        <!-- ฟอร์ม Login — ห้ามเปลี่ยน action และ method -->
        <form action="$(link-login-only)" method="post">
            <input type="hidden" name="dst" value="$(link-orig)">
            <input type="text"
                   name="username"
                   placeholder="Username"
                   value="$(username)">
            <input type="password"
                   name="password"
                   placeholder="Password">
            <button type="submit">เข้าสู่ระบบ</button>
        </form>

        <!-- แสดง Error ถ้า Login ผิด -->
        $(if error)
        <p class="error">⚠️ $(error)</p>
        $(endif)

        <p style="font-size:12px; color:#999; margin-top:20px;">
            IP ของคุณ: $(ip)
        </p>
    </div>
</body>
</html>
```

> ⚠️ **Note สำคัญ!** ในฟอร์ม Login ต้องมีครบ:
> - `action="$(link-login-only)"` — ห้ามเปลี่ยน
> - `method="post"` — ห้ามเปลี่ยน
> - `name="username"` — ห้ามเปลี่ยน
> - `name="password"` — ห้ามเปลี่ยน
> - `name="dst"` — สำหรับ Redirect หลัง Login
>
> ถ้าเปลี่ยน attribute เหล่านี้ ระบบ Login จะพัง

> 💡 **Tip:** ก่อนแก้ไข login.html ควร Backup ไฟล์ต้นฉบับไว้ก่อนเสมอ เพราะถ้าแก้ผิดพลาด ลูกค้าจะ Login ไม่ได้เลย

> 📌 **ไปศึกษาต่อ:** หน้า Login ที่สวยงามและมีฟีเจอร์ครบ เช่น Social Login (Login ด้วย Facebook), โฆษณา, ระบบนับถอยหลัง ต้องเรียนรู้ HTML/CSS/JavaScript เพิ่มเติม หรือดาวน์โหลด Template สำเร็จรูปจากเว็บไซต์ต่าง ๆ

---

## บทที่ 22 — การตั้ง Session Limit เวลา / ปริมาณ Data

---

### 22.1 Session คืออะไร?

**Session** (อ่านว่า เซส-ชั่น) คือ "ช่วงเวลาการใช้งาน" ของลูกค้าหนึ่งครั้ง
ตั้งแต่ Login จนถึง Logout หรือหมดเวลา

**Session จะสิ้นสุดเมื่อ:**
- ลูกค้า Logout เอง
- หมดเวลาที่กำหนด (Session Timeout)
- ใช้ Data ครบที่กำหนด (Byte Limit)
- ลูกค้าปิด Wi-Fi หรือออกจากพื้นที่ (Idle Timeout)
- Admin Kick ออก

---

### 22.2 Session Timeout — จำกัดเวลา

**Session Timeout** (เซส-ชั่น ไทม์-เอ้าท์) คือ เวลาสูงสุดที่ลูกค้าใช้งานได้ต่อครั้ง

ตั้งได้ 2 ที่:
1. ใน User Profile (ใช้กับทุก User ใน Profile นั้น)
2. ใน User แต่ละคน (Override Profile)

**ตั้งใน User Profile:**
```
IP → Hotspot → User Profiles → ดับเบิ้ลคลิก Profile
    → Session Timeout: 02:00:00  ← 2 ชั่วโมง
```

**รูปแบบเวลา:**
```
00:30:00 = 30 นาที
01:00:00 = 1 ชั่วโมง
02:00:00 = 2 ชั่วโมง
08:00:00 = 8 ชั่วโมง
1d       = 1 วัน
```

---

### 22.3 Idle Timeout — หมดเวลาเมื่อไม่ได้ใช้งาน

**Idle Timeout** (ไอ-เดิล ไทม์-เอ้าท์) คือ เวลาที่ถ้าลูกค้าไม่มีการส่งข้อมูลเลย ระบบจะ Kick ออก

ใช้ป้องกัน:
- คนจ่ายเงินซื้อ Session แต่ไม่ใช้งาน (นั่งดูรายการโทรทัศน์แทน)
- มือถือที่ล็อคหน้าจอแต่ยังเชื่อมต่ออยู่ครอง IP และ Session

```
IP → Hotspot → User Profiles → ดับเบิ้ลคลิก Profile
    → Idle Timeout: 00:10:00  ← ถ้าไม่ใช้งาน 10 นาที จะ Kick ออก
```

> 💡 **Tip:** ตั้ง Idle Timeout ไม่สั้นเกินไป (ไม่ต่ำกว่า 5 นาที) เพราะ YouTube, Line, Spotify มีการส่ง Background Data เล็กน้อยอยู่เสมอ ถ้าสั้นเกินไปลูกค้าจะหลุดออกบ่อย

---

### 22.4 Keepalive Timeout — ตรวจสอบว่าลูกค้ายังอยู่

**Keepalive Timeout** (คีพ-อะ-ไลฟ์ ไทม์-เอ้าท์) คือ เวลาที่ MikroTik รอตอบสนองจากลูกค้า
ถ้าลูกค้าไม่ตอบสนองภายในเวลานี้ (เช่น ปิด Wi-Fi) จะ Kick ออก

```
IP → Hotspot → User Profiles → ดับเบิ้ลคลิก Profile
    → Keepalive Timeout: 00:02:00  ← รอ 2 นาที
```

**ความแตกต่างระหว่าง Timeout ทั้งสาม:**
```
Session Timeout:  เวลาใช้งานสูงสุดรวม (นับแม้กำลังใช้งานอยู่)
Idle Timeout:     เวลาที่ไม่มีการส่งข้อมูลเลย
Keepalive Timeout: เวลารอ Ping ตอบกลับจากลูกค้า
```

---

### 22.5 Byte Limit — จำกัดปริมาณ Data

ตั้งได้ใน User Profile หรือใน User แต่ละคน:

**ใน User Profile (ใช้กับทุกคนใน Profile):**
```
IP → Hotspot → User Profiles → แท็บ "Limits"
    → Limit Bytes In: 1073741824   ← Download 1 GB
    → Limit Bytes Out: 524288000   ← Upload 500 MB
    → Limit Bytes Total: 1610612736 ← รวม 1.5 GB
```

**ใน User แต่ละคน (กำหนดเฉพาะ):**
```
IP → Hotspot → Users → ดับเบิ้ลคลิกที่ User
    → Limit Bytes Total: 1073741824  ← 1 GB
```

> ⚠️ **Note:** ค่า Limit ใน User Profile กับใน User ตัว ค่าที่เฉพาะเจาะจงกว่า (ใน User) จะ Override ค่าใน Profile เสมอ

---

## บทที่ 23 — การตั้ง Speed Limit (Rate Limit) ต่อ User

---

### 23.1 Rate Limit คืออะไร?

**Rate Limit** (อ่านว่า เรต ลิ-มิต) คือการจำกัดความเร็วอินเทอร์เน็ตต่อ User หรือต่อ Profile

ทำไมต้องจำกัดความเร็ว?
- ป้องกันไม่ให้ลูกค้าคนหนึ่งกินแบนด์วิดท์ทั้งหมด
- สร้างแพ็กเกจบริการหลายระดับราคา
- ให้บริการที่ยุติธรรมกับทุกคน

---

### 23.2 การตั้ง Rate Limit ใน User Profile

```
IP → Hotspot → User Profiles → ดับเบิ้ลคลิก Profile
    → Rate Limit (rx/tx): 5M/5M
```

**รูปแบบการใส่ค่า:**
```
5M/5M        = Download 5 Mbps / Upload 5 Mbps
10M/5M       = Download 10 Mbps / Upload 5 Mbps
512k/256k    = Download 512 Kbps / Upload 256 Kbps
```

**หน่วยที่ใช้:**
- `k` หรือ `K` = Kilobits per second (กิโลบิตต่อวินาที)
- `M` = Megabits per second (เมกะบิตต่อวินาที)
- `G` = Gigabits per second (กิกะบิตต่อวินาที)

> ⚠️ **Note:** MikroTik ใช้หน่วยเป็น **Bits** ไม่ใช่ Bytes
> ถ้าต้องการ Download 1 MB/s จริง ต้องตั้ง `8M` (เพราะ 1 Byte = 8 Bits)
> ผู้ใช้มักสับสนตรงนี้!

---

### 23.3 Rate Limit แบบ Burst

**Burst** (อ่านว่า เบิร์สต์) แปลว่า "ระเบิด" หรือ "พุ่งสูงชั่วคราว"

Burst ทำให้ลูกค้าได้ความเร็วสูงกว่าปกติชั่วคราว
เช่น ปกติ 5M แต่ในช่วง 10 วินาทีแรกได้ 10M เพื่อให้โหลดหน้าเว็บเร็วขึ้น

รูปแบบ Rate Limit แบบ Burst:
```
Rate Limit (rx/tx): 10M/10M 20M/20M 5M/5M 00:00:05 5M/5M 5M/5M
                    ────────  ───────  ─────  ─────────  ─────  ─────
                    ปกติ      Burst    Thres  เวลา Burst  MIR    MIR
```

อธิบายแต่ละส่วน:
| ส่วน | ค่าตัวอย่าง | ความหมาย |
|-----|-----------|---------|
| Rate Limit | 10M/10M | ความเร็วปกติ |
| Burst Limit | 20M/20M | ความเร็วสูงสุดตอน Burst |
| Burst Threshold | 5M/5M | ถ้าต่ำกว่านี้ถึงจะ Burst ได้ |
| Burst Time | 00:00:05 | Burst ได้นาน 5 วินาที |

> 💡 **Tip:** สำหรับมือใหม่ ให้ตั้งแค่ `Rate Limit (rx/tx): 5M/5M` แบบง่าย ๆ ก่อน
> Burst ตั้งทีหลังเมื่อเข้าใจระบบดีแล้ว

---

### 23.4 ตั้ง Rate Limit ใน User แต่ละคน

ถ้าต้องการความเร็วพิเศษสำหรับ User คนใดคนหนึ่ง:
```
IP → Hotspot → Users → ดับเบิ้ลคลิกที่ User
    → Rate Limit (rx/tx): 20M/20M
```

> 💡 **Tip:** Rate Limit ใน User จะ Override Rate Limit ใน Profile
> ใช้ประโยชน์โดยสร้าง User พิเศษสำหรับตัวเองหรือ VIP ที่ต้องการความเร็วสูงพิเศษ

---

## บทที่ 24 — การ Kick / Disconnect User

---

### 24.1 ทำไมต้อง Kick User?

บางครั้งต้องตัดการเชื่อมต่อของลูกค้า เช่น:
- ลูกค้าใช้งานผิดกฎ
- Session ค้าง ไม่ยอม Disconnect เอง
- ต้องการ Reset การใช้งาน
- ทดสอบระบบ

---

### 24.2 วิธี Kick User

**วิธีที่ 1 — ผ่าน Active Hotspot:**
```
IP → Hotspot → แท็บ "Active" → คลิกที่ User ที่ต้องการ
    → กด "-" (ลบ) หรือคลิกขวา → Remove
```

**วิธีที่ 2 — ผ่าน Terminal:**
```bash
# ดู Active User ทั้งหมด
/ip hotspot active print

# Kick User ด้วยชื่อ
/ip hotspot active remove [find user="user001"]

# Kick ทุกคนพร้อมกัน (ระวัง!)
/ip hotspot active remove [find]
```

> ⚠️ **Note:** การ Kick User จะทำให้ลูกค้าหลุดออกทันที แต่ **ไม่ได้ลบ Account** ลูกค้ายังสามารถ Login กลับมาใหม่ได้ ถ้าต้องการบล็อกถาวรต้องลบ User Account ออก หรือเพิ่มเข้า Blacklist

---

### 24.3 การ Block User ชั่วคราวหรือถาวร

**Disable User (ปิดการใช้งานชั่วคราว):**
```
IP → Hotspot → Users → คลิกที่ User
    → คลิกปุ่ม "X" สีแดงด้านซ้ายของรายการ
    (หรือคลิกขวา → Disable)
```

User ที่ Disable จะ Login ไม่ได้จนกว่าจะ Enable กลับ

**Enable User กลับมา:**
```
คลิกขวาที่ User → Enable
```

---

## บทที่ 25 — การดู Active User และ Log

---

### 25.1 ดู Active User (คนที่กำลังใช้งานอยู่)

```
IP → Hotspot → แท็บ "Active"
```

ข้อมูลที่เห็น:
```
Server    User      Address        MAC              Login Time  Idle  Session  Bytes In   Bytes Out
hs1       user001   192.168.88.10  AA:BB:CC:DD:EE   00:30:00   00:00 01:30:00  150.5 MiB  20.3 MiB
hs1       user002   192.168.88.11  11:22:33:44:55   00:15:00   00:02 00:45:00  50.2 MiB   5.1 MiB
```

**ฟิลด์สำคัญ:**
| ฟิลด์ | ความหมาย |
|------|---------|
| User | Username ที่ Login |
| Address | IP ของลูกค้า |
| MAC | MAC Address ของอุปกรณ์ |
| Login Time | Login มานานแค่ไหนแล้ว |
| Idle | ไม่มี Traffic มานานแค่ไหน |
| Session Time Left | เวลาที่เหลือ |
| Bytes In | Data ที่ดาวน์โหลดทั้งหมด |
| Bytes Out | Data ที่อัปโหลดทั้งหมด |

---

### 25.2 ดู Hotspot Host (อุปกรณ์ทุกชิ้นใน LAN)

```
IP → Hotspot → แท็บ "Hosts"
```

เห็นอุปกรณ์ทุกชิ้นที่เชื่อมต่อ ไม่ว่าจะ Login แล้วหรือยัง:
- **Status "authorized"** = Login แล้ว ใช้งานได้
- **Status "!authorized"** = ยังไม่ได้ Login

---

### 25.3 ดู Log ของ Hotspot

**Log** (อ่านว่า ล็อก) คือบันทึกเหตุการณ์ต่าง ๆ ในระบบ

**ดู Log ผ่าน Winbox:**
```
Winbox → Log → เลือก Topic: hotspot
```

**ดู Log ผ่าน Terminal:**
```bash
# ดู Log ทั้งหมด
/log print

# ดู Log เฉพาะ Hotspot
/log print where topics~"hotspot"

# ดู Log แบบ Real-time (อัปเดตสด)
/log print follow where topics~"hotspot"
```

**ตัวอย่าง Log ที่เห็นบ่อย:**
```
12:30:01 hotspot,info user001 logged in from 192.168.88.10
12:30:01 hotspot,info user001 session started 192.168.88.10
13:30:01 hotspot,info user001 logged out from 192.168.88.10 (session timeout)
13:35:22 hotspot,warning wrong password for user "user999" from 192.168.88.15
```

---

### 25.4 การตั้งค่า Log ให้บันทึกอย่างละเอียด

```
System → Logging → กด "+"
    → Topics: hotspot
    → Action: memory  ← เก็บใน RAM (หายเมื่อ Reboot)
    หรือ
    → Action: disk    ← เก็บใน Flash (ถาวรกว่า)
```

> ⚠️ **Note กฎหมาย:** ตาม พ.ร.บ. คอมพิวเตอร์ ผู้ให้บริการอินเทอร์เน็ตต้องเก็บ Log อย่างน้อย **90 วัน** และต้องสามารถระบุได้ว่าใครใช้ IP ไหน เมื่อไหร่ Log ใน RAM จะหายเมื่อ Reboot ดังนั้นต้องตั้งให้ส่ง Log ออกไปยังภายนอกด้วย (เรียนเรื่องนี้ในกลุ่มที่ 8)

> 📌 **ไปศึกษาต่อ:** การส่ง Log ออกไปยัง Syslog Server ภายนอกเพื่อเก็บระยะยาว ดูในหัวข้อ "System Logging" และ "Syslog" ใน MikroTik Wiki

---

## ภาคผนวก — คำสั่ง Hotspot ใน Terminal

```bash
# ===== ดูข้อมูล Hotspot =====
/ip hotspot print                      # ดู Hotspot Server
/ip hotspot profile print              # ดู Server Profile
/ip hotspot user profile print         # ดู User Profile
/ip hotspot user print                 # ดู User ทั้งหมด
/ip hotspot active print               # ดู Active User
/ip hotspot host print                 # ดูอุปกรณ์ทั้งหมดใน LAN

# ===== จัดการ User =====
# เพิ่ม User
/ip hotspot user add name=user001 password=pass1234 profile=basic

# เปลี่ยน Password
/ip hotspot user set [find name="user001"] password=newpass

# Disable User
/ip hotspot user disable [find name="user001"]

# Enable User
/ip hotspot user enable [find name="user001"]

# ลบ User
/ip hotspot user remove [find name="user001"]

# Reset Counter (เคลียร์เวลาและ Data)
/ip hotspot user set [find name="user001"] \
    bytes-in=0 bytes-out=0 uptime=0

# ===== จัดการ Active Session =====
# Kick User ออก
/ip hotspot active remove [find user="user001"]

# ดู Session ของ User คนนั้น
/ip hotspot active print where user="user001"

# ===== Walled Garden =====
/ip hotspot walled-garden print        # ดูรายการ Walled Garden
/ip hotspot walled-garden ip print     # ดูรายการ Walled Garden แบบ IP

# ===== Log =====
/log print where topics~"hotspot"      # Log เฉพาะ Hotspot
```

---

## สรุปภาพรวม — Checklist ก่อนเปิดบริการ Hotspot จริง

```
✅ Checklist ระบบ Hotspot พร้อมให้บริการ
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[ ] 1.  ตั้ง Hotspot บน Interface ถูกต้อง (LAN ไม่ใช่ WAN)
[ ] 2.  ทดสอบ Redirect ไปหน้า Login ได้ (เปิดเบราว์เซอร์แล้วเห็นหน้า Login)
[ ] 3.  Login ด้วย User ทดสอบแล้วออกเน็ตได้
[ ] 4.  Logout แล้วถูก Redirect กลับมาหน้า Login
[ ] 5.  ตั้ง User Profile ครบทุกแพ็กเกจที่จะขาย
[ ] 6.  ทดสอบ Rate Limit ว่าจำกัดความเร็วได้จริง
[ ] 7.  ทดสอบ Session Timeout ว่าหมดเวลาแล้ว Kick ออก
[ ] 8.  ตั้ง Walled Garden สำหรับเว็บที่ต้องการเปิดฟรี
[ ] 9.  ปรับแต่งหน้า Login Page ให้มีชื่อร้าน/โลโก้
[ ] 10. ตั้ง Idle Timeout ป้องกัน Session ค้าง
[ ] 11. ทดสอบจาก มือถือ iOS, Android, Windows
[ ] 12. ตรวจสอบว่า Log ถูกบันทึกอย่างถูกต้อง
[ ] 13. Backup Config ก่อนเปิดบริการจริง
[ ] 14. มีแผนรับมือถ้า MikroTik Reboot โดยไม่ตั้งใจ
        (ตั้งค่าให้ Start Hotspot อัตโนมัติ — ปกติทำได้เอง)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## ปัญหาที่พบบ่อยและวิธีแก้ไข

| ปัญหา | สาเหตุที่พบบ่อย | วิธีแก้ไข |
|------|---------------|---------|
| ลูกค้าเชื่อมต่อได้แต่ไม่มีหน้า Login | DNS ไม่ทำงาน / Allow Remote Requests ไม่ได้ติ๊ก | ตรวจสอบ IP → DNS → ติ๊ก Allow Remote Requests |
| Login แล้วยังออกเน็ตไม่ได้ | NAT Masquerade ไม่ได้ตั้งหรือตั้งผิด Interface | ตรวจสอบ IP → Firewall → NAT |
| หน้า Login ไม่ปรากฏบน iOS/Chrome | HTTPS Captive Portal Detection | เพิ่ม Walled Garden สำหรับ apple.com, google.com |
| ลูกค้า Login แล้วหลุดบ่อย | Idle Timeout สั้นเกินไป | เพิ่ม Idle Timeout เป็น 10-15 นาที |
| Login ได้แต่ผิด Password ทุกครั้ง | Caps Lock, เข้ารหัสผิดแบบ | ลองเปลี่ยน Login Method เป็น PAP ชั่วคราว |
| ลูกค้า 2 คนใช้ User เดียวกันไม่ได้ | Shared Users ตั้งเป็น 1 | เพิ่ม Shared Users ใน Profile |
| หน้า Login แสดงภาษาอังกฤษทั้งหมด | ไม่ได้แก้ไข login.html | แก้ไขหน้า Login ตามบทที่ 21 |

---

## แบบทดสอบตัวเอง

1. Hotspot ต่างจาก Router ธรรมดาอย่างไรใน 3 ประเด็นหลัก?
2. Walled Garden คืออะไร? ยกตัวอย่างเว็บที่ควรใส่ใน Walled Garden?
3. ความแตกต่างระหว่าง Session Timeout, Idle Timeout, และ Keepalive Timeout?
4. Rate Limit `10M/5M` หมายความว่าอะไร? Download กับ Upload อันไหนเร็วกว่า?
5. ถ้าต้องการสร้าง User 100 คนพร้อมกัน ควรใช้ฟีเจอร์อะไร?
6. ทำไม Login Page ถึงต้องมี `action="$(link-login-only)"` และห้ามเปลี่ยน?
7. ถ้าต้องการ Disable User ชั่วคราวโดยไม่ลบ Account ทำอย่างไร?

---

## แหล่งศึกษาต่อ

| แหล่ง | เนื้อหา |
|------|--------|
| wiki.mikrotik.com/wiki/Hotspot | เอกสาร Hotspot อย่างเป็นทางการ |
| wiki.mikrotik.com/wiki/HotSpot_Login_Page | วิธีแก้ไข Login Page |
| YouTube: "MikroTik Hotspot Setup" | ดูวิดีโอการตั้งค่าเป็นขั้นตอน |
| forum.mikrotik.com | ถามปัญหา มีชุมชนช่วยตอบ |
| Facebook: MikroTik Thailand | ชุมชนไทย ถามตอบภาษาไทย |

---

## หัวข้อที่ต้องเรียนต่อในกลุ่มที่ 4

เมื่อตั้งค่า Hotspot ได้แล้ว ขั้นต่อไปคือการจัดการ Traffic ให้มีประสิทธิภาพ:

- [ ] Queue — Simple Queue vs Queue Tree
- [ ] การจำกัด Bandwidth รายคน / รายกลุ่ม
- [ ] Burst คืออะไร ตั้งค่าอย่างไร
- [ ] IP Binding — ล็อก IP ให้ User
- [ ] Address List — การจัดกลุ่ม IP
- [ ] Mangle — การ Mark Traffic เบื้องต้น

---

*เอกสารนี้จัดทำสำหรับผู้เรียน MikroTik Hotspot ระดับเริ่มต้น | อัปเดตล่าสุด: 2026*
