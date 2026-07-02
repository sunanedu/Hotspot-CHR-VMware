# กลุ่มที่ 2 — พื้นฐาน MikroTik
### สำหรับผู้เรียนที่เริ่มต้นจากศูนย์ | ต่อยอดจากกลุ่มที่ 1

---

> **📌 ข้อกำหนดก่อนเรียน**
> ควรอ่านกลุ่มที่ 1 (พื้นฐานเครือข่าย) มาก่อน โดยเฉพาะเรื่อง IP Address, Subnet Mask, Gateway, DHCP และ NAT เพราะเนื้อหากลุ่มนี้จะนำแนวคิดเหล่านั้นมาใช้งานจริงบน MikroTik

---

## บทที่ 8 — RouterOS คืออะไร? CHR คืออะไร?

---

### 8.1 MikroTik คืออะไร?

**MikroTik** (อ่านว่า ไมโคร-ทิค) คือบริษัทจากประเทศลัตเวีย (Latvia) ในยุโรป
ก่อตั้งปี 1996 ผลิตทั้ง **Hardware** (อุปกรณ์เน็ตเวิร์ก) และ **Software** (ระบบปฏิบัติการ)

สิ่งที่ทำให้ MikroTik โดดเด่น:
- ราคาถูกกว่าคู่แข่งอย่าง Cisco หรือ Juniper มาก
- มีฟีเจอร์ครบครัน เหมาะกับ ISP ขนาดเล็ก-กลาง
- นิยมมากในไทย เอเชียตะวันออกเฉียงใต้ และยุโรปตะวันออก

---

### 8.2 RouterOS คืออะไร?

**RouterOS** (อ่านว่า เราต์-เตอร์-โอ-เอส) คือ **ระบบปฏิบัติการ** (Operating System)
ของ MikroTik พัฒนามาจาก Linux

เหมือนกับว่า:
- คอมพิวเตอร์ใช้ Windows หรือ macOS
- Router ของ MikroTik ใช้ **RouterOS**

RouterOS ทำหน้าที่ควบคุม:
- การรับ-ส่งข้อมูลระหว่าง Network
- Firewall (ไฟร์-วอลล์) ป้องกันการบุกรุก
- DHCP Server แจก IP
- Hotspot ระบบ Login
- QoS (คิว-โอ-เอส) จัดการความเร็วอินเทอร์เน็ต
- VPN (วี-พี-เอ็น) เชื่อมต่อเครือข่ายปลอดภัย
- และอื่น ๆ อีกมาก

**เวอร์ชันของ RouterOS:**

| เวอร์ชัน | สถานะ | คำแนะนำ |
|---------|------|---------|
| v6.x | เก่า แต่ยังใช้งานได้ | ไม่แนะนำสำหรับระบบใหม่ |
| v7.x | ล่าสุด (คุณใช้อยู่) | **แนะนำ** มีฟีเจอร์ใหม่ครบ |

> ⚠️ **Note:** RouterOS v7 มี Interface การใช้งานแตกต่างจาก v6 บ้าง บางเมนูย้ายที่ ถ้าดู Tutorial เก่า ๆ บน YouTube อาจเห็นเมนูไม่ตรงกัน ให้ระวังและดูเวอร์ชันของ Tutorial ด้วย

---

### 8.3 CHR คืออะไร?

**CHR** (อ่านว่า ซี-เอช-อาร์) ย่อมาจาก **Cloud Hosted Router**
(คลาวด์ โฮสต์-เต็ด เราต์-เตอร์)

แปลว่า: "Router ที่รันอยู่บน Virtual Machine (เครื่องเสมือน)"

คุณกำลังใช้ MikroTik CHR รัน **บน VMware** — นั่นหมายความว่า:
- ไม่ต้องซื้อ Hardware Router จริง ๆ
- RouterOS ทำงานเหมือนโปรแกรมทั่วไปบน Windows/Linux
- สามารถ Snapshot (สแน็ป-ช็อต) หรือ Backup VM ได้ง่าย

**ข้อจำกัดของ CHR (Free License):**

| License | ความเร็วสูงสุด | ราคา |
|---------|--------------|------|
| Free (Trial) | 1 Mbps | ฟรี แต่ช้ามาก |
| P1 | 1 Gbps | ~$45/ปี |
| P10 | 10 Gbps | ~$95/ปี |
| P-Unlimited | ไม่จำกัด | ~$195/ปี |

> ⚠️ **Note สำคัญ!** CHR แบบฟรีจะ **จำกัดความเร็วที่ 1 Mbps** ซึ่งใช้งานจริงไม่ได้ ต้องซื้อ License หรือสมัคร Trial 60 วัน ก่อนให้บริการจริงต้องวางแผนเรื่อง License ด้วย

> 💡 **Tip:** สำหรับทดลองระบบใช้ Free License ได้ก่อน แต่พอจะทดสอบความเร็วจริง ให้สมัคร Trial License ที่ mikrotik.com ได้ฟรี 60 วัน

> ❓ **คำถามทิ้งไว้:** ถ้าคุณให้บริการลูกค้า 50 คน แต่ใช้ CHR Free License ที่จำกัด 1 Mbps จะเกิดอะไรขึ้น? ลูกค้าแต่ละคนจะได้ความเร็วเท่าไหร่?

---

## บทที่ 9 — การเข้าใช้งาน Winbox / WebFig / Terminal

---

### 9.1 วิธีเข้าจัดการ MikroTik มี 3 ทาง

```
MikroTik RouterOS
      │
      ├── 1. Winbox      ← โปรแกรม Windows (แนะนำมากที่สุด)
      ├── 2. WebFig      ← เว็บเบราว์เซอร์
      └── 3. Terminal    ← Command Line (พิมพ์คำสั่ง)
```

---

### 9.2 Winbox — วิธีที่แนะนำสำหรับมือใหม่

**Winbox** (อ่านว่า วิน-บ็อกซ์) คือโปรแกรมบน Windows ที่ MikroTik พัฒนาขึ้นมาเอง
ใช้งานง่าย มีเมนูครบ เห็นภาพรวมระบบได้ชัดเจน

**ดาวน์โหลด Winbox:**
- ไปที่ https://mikrotik.com/download
- เลือก "Winbox" (มีทั้งแบบ .exe และ .dmg สำหรับ Mac)
- ใช้ **Winbox 4** สำหรับ RouterOS v7 (หน้าตาใหม่กว่า)
- ใช้ **Winbox 3** ถ้าคุ้นเคยหรือดู Tutorial เก่า

**ขั้นตอนการเชื่อมต่อ Winbox:**

```
เปิด Winbox
    │
    ↓
ช่อง "Connect To" → ใส่ IP ของ MikroTik
    │            (เช่น 192.168.88.1)
    │            หรือใส่ MAC Address ก็ได้
    ↓
Login: admin
Password: (ว่างเปล่าถ้าเพิ่ง Install ใหม่)
    │
    ↓
กด "Connect"
    │
    ↓
เข้าสู่หน้าจัดการ MikroTik ✓
```

**การเชื่อมต่อผ่าน MAC Address:**

Winbox สามารถค้นหา MikroTik ในเครือข่าย และเชื่อมต่อผ่าน MAC Address ได้
ประโยชน์คือ: **แม้ยังไม่ได้ตั้ง IP ให้ Router ก็เชื่อมต่อได้**

```
กดปุ่ม "..." ข้างช่อง Connect To
    │
    ↓
Winbox จะสแกนหา MikroTik ในวงเครือข่ายเดียวกัน
    │
    ↓
คลิกที่ MAC Address ของ MikroTik ที่ปรากฏขึ้น
    │
    ↓
กด Connect
```

> 💡 **Tip:** ถ้าเชื่อมต่อผ่าน MAC Address แล้วทำงานช้ามาก ให้เปลี่ยนมาใช้ IP Address แทน เพราะ MAC Address ทำงานแค่ใน Layer 2 ซึ่งมีข้อจำกัดด้านระยะทางและประสิทธิภาพ

> ⚠️ **Note:** Winbox ทำงานได้บน Windows เท่านั้น (ถ้าใช้ Mac/Linux ต้องใช้ Wine หรือ WebFig แทน)

---

### 9.3 WebFig — เข้าผ่านเบราว์เซอร์

**WebFig** (อ่านว่า เว็บ-ฟิก) คือ Web Interface ของ MikroTik
เข้าใช้งานได้จากเบราว์เซอร์ทุกตัว โดยไม่ต้องติดตั้งโปรแกรมเพิ่ม

**วิธีเข้าใช้:**
1. เปิดเบราว์เซอร์ (Chrome, Firefox, Edge)
2. พิมพ์ IP ของ MikroTik ในช่อง URL เช่น `http://192.168.88.1`
3. Login ด้วย admin / password

> 💡 **Tip:** WebFig มีเมนูและฟีเจอร์เหมือน Winbox เกือบทุกอย่าง เหมาะสำหรับการจัดการระยะไกลโดยไม่ต้องติดตั้ง Winbox

---

### 9.4 Terminal — การพิมพ์คำสั่ง

**Terminal** (อ่านว่า เทอร์-มิ-เนิล) หรือ **CLI** (ซี-แอล-ไอ = Command Line Interface)
คือการจัดการ MikroTik โดยพิมพ์คำสั่งตรง ๆ

เข้าใช้งานได้จาก:
- Winbox → เมนู "New Terminal"
- SSH (เอส-เอส-เอช): `ssh admin@192.168.88.1`
- Telnet (เทล-เน็ต): ไม่แนะนำ ไม่ปลอดภัย

ตัวอย่างคำสั่งง่าย ๆ:
```bash
# ดู IP ที่ตั้งบน Interface ทั้งหมด
/ip address print

# ดู Route ทั้งหมด
/ip route print

# ดู Interface ทั้งหมด
/interface print

# Ping ทดสอบการเชื่อมต่อ
/ping 8.8.8.8
```

> 📌 **ไปศึกษาต่อ:** การใช้ Terminal ใน MikroTik เรียกว่า "RouterOS CLI" มีคำสั่งหลายร้อยคำสั่ง แต่สำหรับมือใหม่ให้เน้นใช้ Winbox ก่อน แล้วค่อย ๆ เรียน CLI เพิ่มเติมทีหลัง

---

## บทที่ 10 — การตั้งชื่อ Interface และ Bridge

---

### 10.1 Interface คืออะไร?

**Interface** (อ่านว่า อิน-เทอร์-เฟส) แปลว่า "ส่วนต่อประสาน" หรือในที่นี้หมายถึง **ช่องรับส่งข้อมูล**

ใน MikroTik แต่ละ Interface คือ:
- การ์ดแลนหนึ่งใบ → `ether1`, `ether2`, `ether3`...
- การ์ด Wi-Fi → `wlan1`, `wlan2`...
- อินเทอร์เฟสเสมือน → `bridge1`, `vlan10`...

**ดู Interface ใน Winbox:**
```
Winbox → Interfaces → Interface List
```

จะเห็นรายการเช่น:
```
ether1   ── ต่อกับโมเด็ม ISP (WAN)
ether2   ── ต่อกับ Switch หรือ PC (LAN)
```

---

### 10.2 ทำไมต้องตั้งชื่อ Interface?

โดย Default MikroTik จะตั้งชื่อให้ว่า `ether1`, `ether2`... ซึ่งจำได้ยากว่าอันไหนต่ออะไร

**แนะนำให้เปลี่ยนชื่อให้เข้าใจง่าย** เช่น:

| ชื่อ Default | เปลี่ยนเป็น | เหตุผล |
|-------------|-----------|--------|
| ether1 | WAN หรือ ISP | ต่อกับโมเด็ม |
| ether2 | LAN หรือ LAN-Clients | ต่อกับลูกค้า |
| ether3 | MGMT หรือ Admin | ใช้จัดการ Router |

**วิธีเปลี่ยนชื่อ Interface ใน Winbox:**
```
Interfaces → ดับเบิ้ลคลิกที่ Interface ที่ต้องการ
    → ช่อง "Name" → พิมพ์ชื่อใหม่
    → กด "OK"
```

> 💡 **Tip:** ตั้งชื่อให้ชัดเจนตั้งแต่แรก เพราะเวลาเขียน Firewall Rule หรือ NAT Rule จะต้องอ้างอิงชื่อ Interface เสมอ ถ้าชื่อสับสน จะทำให้ตั้งค่าผิดได้ง่าย

---

### 10.3 Bridge คืออะไร?

**Bridge** (อ่านว่า บริดจ์) แปลว่า "สะพาน"

ใน MikroTik, Bridge ทำหน้าที่ **รวม Interface หลาย ๆ อันให้ทำงานเป็นเครือข่ายเดียวกัน**

**อุปมา:**
- นึกถึง Switch ตัวเล็ก ๆ
- ถ้าคุณมี ether2, ether3, ether4 และอยากให้ทุกพอร์ตอยู่ใน LAN เดียวกัน
- ให้สร้าง Bridge แล้วดึง Interface ทั้งหมดเข้ามา

```
[ether2] ─┐
[ether3] ─┤── [Bridge1 = 192.168.88.1/24] ── ใช้งานเป็น LAN เดียวกัน
[ether4] ─┘
```

**วิธีสร้าง Bridge ใน Winbox:**

ขั้นตอนที่ 1 — สร้าง Bridge:
```
Interfaces → Bridge → กด "+" (เพิ่ม)
    → Name: bridge1 (หรือตั้งชื่อเอง)
    → กด OK
```

ขั้นตอนที่ 2 — เพิ่ม Port เข้า Bridge:
```
Interfaces → Bridge → แท็บ "Ports" → กด "+"
    → Interface: ether2
    → Bridge: bridge1
    → กด OK
    (ทำซ้ำสำหรับ ether3, ether4...)
```

ขั้นตอนที่ 3 — ตั้ง IP บน Bridge:
```
IP → Addresses → กด "+"
    → Address: 192.168.88.1/24
    → Interface: bridge1
    → กด OK
```

> ⚠️ **Note:** ห้ามตั้ง IP บน Interface ที่เป็น Bridge Port โดยตรง ต้องตั้ง IP บน Bridge Interface เท่านั้น เช่น ถ้า ether2 อยู่ใน bridge1 อย่าตั้ง IP บน ether2 ให้ตั้งบน bridge1 แทน

> ❓ **คำถามทิ้งไว้:** ในระบบของคุณที่มีการ์ดแลน 2 ใบ (WAN และ LAN) จำเป็นต้องสร้าง Bridge ไหม? สถานการณ์แบบไหนที่ต้องสร้าง Bridge?

---

## บทที่ 11 — การตั้งค่า IP Address บน Interface

---

### 11.1 หลักการตั้ง IP ใน MikroTik

MikroTik ต้องการ IP Address บน Interface เพื่อ:
- รับการเชื่อมต่อจากลูกค้า (LAN Side)
- ส่งข้อมูลออกอินเทอร์เน็ต (WAN Side)
- จัดการ Router ผ่าน Winbox/WebFig

**โครงสร้าง IP ในระบบของคุณ:**
```
[ISP / โมเด็ม]
192.168.1.1
      │
      │ WAN (ether1)
      │ 192.168.1.x (ได้จาก DHCP ของโมเด็ม)
[MikroTik CHR]
      │ LAN (ether2 หรือ bridge1)
      │ 192.168.88.1/24
      │
[ลูกค้า / PC Windows 7]
192.168.88.x
```

---

### 11.2 วิธีดู IP ที่มีอยู่

```
Winbox → IP → Addresses
```

จะเห็นรายการ IP ที่ตั้งบน Interface ต่าง ๆ เช่น:
```
192.168.88.1/24    bridge1    (LAN)
192.168.1.100/24   ether1     (WAN - ได้จาก DHCP)
```

---

### 11.3 วิธีเพิ่ม IP Address ใหม่

```
IP → Addresses → กด "+" (เพิ่ม)
    │
    ├── Address: 192.168.88.1/24
    │   (ใส่ IP พร้อม Subnet Mask แบบ CIDR)
    │
    ├── Network: (ปล่อยว่างไว้ MikroTik คำนวณให้อัตโนมัติ)
    │
    ├── Interface: bridge1 (หรือ ether2 แล้วแต่กรณี)
    │
    └── กด "OK"
```

> 💡 **Tip:** เมื่อกรอก Address เช่น `192.168.88.1/24` แล้วกด Tab MikroTik จะคำนวณค่า Network (`192.168.88.0`) และ Broadcast (`192.168.88.255`) ให้อัตโนมัติ

---

### 11.4 การตั้ง IP ฝั่ง WAN

ฝั่ง WAN มีวิธีรับ IP ได้ 2 แบบ:

**แบบที่ 1 — DHCP Client (รับ IP อัตโนมัติจากโมเด็ม ISP)** ← แนะนำ
```
IP → DHCP Client → กด "+"
    → Interface: ether1 (WAN)
    → ติ๊ก "Use Peer DNS" และ "Use Peer NTP"
    → กด OK
```

**แบบที่ 2 — Static IP (ตั้งเองถ้า ISP กำหนด IP คงที่ให้)**
```
IP → Addresses → กด "+"
    → Address: x.x.x.x/xx (ตามที่ ISP กำหนด)
    → Interface: ether1
    → กด OK

แล้วตั้ง Default Route:
IP → Routes → กด "+"
    → Dst. Address: 0.0.0.0/0
    → Gateway: IP ของโมเด็ม
    → กด OK
```

> ⚠️ **Note:** ถ้าใช้ VMware ต้องตั้ง Network Adapter ของ VM ให้ถูกต้องด้วย:
> - การ์ดแลนใบที่ 1 (WAN) ตั้งเป็น **Bridged** หรือ **NAT** (ตามที่คุณตั้งไว้)
> - การ์ดแลนใบที่ 2 (LAN) ตั้งเป็น **Host-only** หรือ **Custom VMnet**
> ถ้าตั้งผิด MikroTik จะไม่เห็น Interface ที่ถูกต้อง

---

## บทที่ 12 — การตั้งค่า DHCP Server

---

### 12.1 ทำไม MikroTik ต้องเป็น DHCP Server?

ระบบ Hotspot ต้องการให้ลูกค้าได้ IP อัตโนมัติเมื่อเชื่อมต่อ Wi-Fi
MikroTik จะทำหน้าที่ DHCP Server แจก IP ในช่วงที่กำหนดให้ลูกค้าทุกคน

---

### 12.2 วิธีตั้งค่า DHCP Server แบบ Wizard (ง่ายที่สุด)

MikroTik มี Setup Wizard ที่ช่วยตั้งค่าทุกอย่างให้ในไม่กี่คลิก:

```
Winbox → IP → DHCP Server → กด "DHCP Setup"
```

จะมีหน้าต่าง Wizard ถามทีละขั้น:

```
ขั้น 1: DHCP Server Interface
    → เลือก Interface ที่จะให้บริการ (เช่น bridge1 หรือ ether2)
    → กด "Next"

ขั้น 2: DHCP Address Space
    → MikroTik จะกรอกให้อัตโนมัติ (เช่น 192.168.88.0/24)
    → กด "Next"

ขั้น 3: Gateway for DHCP Network
    → IP ของ MikroTik (เช่น 192.168.88.1)
    → กด "Next"

ขั้น 4: IP Addresses to Give Out
    → ช่วง IP ที่จะแจก (เช่น 192.168.88.2 - 192.168.88.254)
    → กด "Next"

ขั้น 5: DNS Servers
    → ใส่ 8.8.8.8, 8.8.4.4
    → กด "Next"

ขั้น 6: Lease Time
    → กำหนดเวลา (แนะนำ 1d = 1 วัน สำหรับ Hotspot)
    → กด "Next"

ขั้น 7: Setup has completed successfully ✓
```

---

### 12.3 การตั้งค่า DHCP Server แบบ Manual (ละเอียดกว่า)

ถ้าต้องการควบคุมทุกอย่างด้วยตัวเอง:

**ขั้น 1 — สร้าง IP Pool (ช่วง IP ที่จะแจก):**
```
IP → Pool → กด "+"
    → Name: hotspot-pool
    → Addresses: 192.168.88.10-192.168.88.254
      (เริ่มจาก .10 เพื่อเว้น .1-.9 ไว้ใช้งานอื่น)
    → กด OK
```

**ขั้น 2 — สร้าง DHCP Network (ข้อมูลที่จะแจกพร้อม IP):**
```
IP → DHCP Server → แท็บ "Networks" → กด "+"
    → Address: 192.168.88.0/24
    → Gateway: 192.168.88.1
    → DNS Servers: 8.8.8.8
    → กด OK
```

**ขั้น 3 — สร้าง DHCP Server:**
```
IP → DHCP Server → แท็บ "DHCP" → กด "+"
    → Name: dhcp-lan
    → Interface: bridge1
    → Address Pool: hotspot-pool
    → Lease Time: 00:30:00 (30 นาที สำหรับ Hotspot)
    → กด OK
```

> 💡 **Tip สำหรับ Hotspot:** ตั้ง Lease Time ให้สั้น (30 นาที ถึง 1 ชั่วโมง) เพื่อให้ IP หมุนเวียนได้เร็ว เมื่อลูกค้าออกไปแล้ว IP จะถูกนำกลับมาแจกให้คนใหม่ได้เร็วขึ้น

> ⚠️ **Note:** ถ้ากำหนด IP Pool ไว้น้อยเกินไป เช่น 192.168.88.10-192.168.88.50 (แค่ 41 IP) แต่มีลูกค้า 100 คนพร้อมกัน ลูกค้าที่เข้ามาทีหลังจะไม่ได้รับ IP และเชื่อมต่อไม่ได้

---

### 12.4 การดู Leases (รายการอุปกรณ์ที่ได้รับ IP)

```
IP → DHCP Server → แท็บ "Leases"
```

จะเห็นรายการ เช่น:
```
IP Address        MAC Address         Host Name    Status
192.168.88.10     AA:BB:CC:DD:EE:FF   iPhone       bound
192.168.88.11     11:22:33:44:55:66   Samsung      bound
```

- **bound** = ได้รับ IP และใช้งานอยู่
- **waiting** = รอหมดอายุ

> 💡 **Tip:** ถ้าต้องการ "ล็อก" IP ให้กับอุปกรณ์ใดอุปกรณ์หนึ่งเสมอ (Static Lease) ให้คลิกขวาที่ Lease นั้น → Make Static

---

## บทที่ 13 — การตั้งค่า DNS

---

### 13.1 ทำไม MikroTik ต้องเป็น DNS ด้วย?

ใน Hotspot, MikroTik ต้องดักจับ DNS Request ของลูกค้าก่อน
เพื่อ Redirect ลูกค้าที่ยังไม่ได้ Login ไปหน้า Login Page

ถ้า MikroTik ไม่ได้ทำ DNS แจกให้ลูกค้า แต่ลูกค้าใช้ DNS อื่น (เช่น 8.8.8.8 โดยตรง)
MikroTik อาจไม่สามารถ Redirect ไปหน้า Login ได้ถูกต้อง

---

### 13.2 ตั้งค่า DNS บน MikroTik

```
Winbox → IP → DNS
```

ตั้งค่าดังนี้:

```
Servers: 8.8.8.8
         8.8.4.4
         (กด "+" เพื่อเพิ่ม DNS ตัวที่สอง)

Allow Remote Requests: ✓ (ติ๊กช่องนี้!)
Max UDP Packet Size: 4096 (ปล่อย Default)
```

> ⚠️ **Note สำคัญ:** ต้องติ๊ก **"Allow Remote Requests"** เสมอ!
> ถ้าไม่ติ๊ก MikroTik จะไม่ยอมตอบคำถาม DNS ให้ลูกค้า
> ลูกค้าจะเปิดเว็บไม่ได้และหน้า Hotspot Login จะไม่ปรากฏ

---

### 13.3 DNS Cache ใน MikroTik

MikroTik มี DNS Cache (แคช ดี-เอ็น-เอส) เก็บผลการค้นหา DNS ไว้ชั่วคราว
ทำให้การเปิดเว็บครั้งต่อไปเร็วขึ้น เพราะไม่ต้องถาม DNS Server ข้างนอกซ้ำ

**ดู Cache:**
```
IP → DNS → กด "Cache" → "DNS Cache"
```

**ล้าง Cache (กรณีเว็บเปิดไม่ถูก):**
```
IP → DNS → กด "Cache" → "Flush Cache"
```

> 💡 **Tip:** เมื่อแก้ไข DNS หรือเปลี่ยน IP ของบริการ ให้ Flush Cache ทุกครั้ง มิฉะนั้น MikroTik อาจยังจำ IP เก่าอยู่และ Route ผิด

---

## บทที่ 14 — การตั้งค่า NAT (Masquerade)

---

### 14.1 ทำไมต้องตั้ง NAT?

จำบทที่ 5 ในกลุ่มที่ 1 ได้ไหม? ลูกค้าที่มี Private IP ต้องผ่าน NAT ถึงจะออกอินเทอร์เน็ตได้
MikroTik ต้องตั้งค่า NAT แบบ Masquerade เพื่อแปลง Private IP ของลูกค้าเป็น Public IP ของ WAN

---

### 14.2 วิธีตั้ง NAT Masquerade

```
Winbox → IP → Firewall → แท็บ "NAT" → กด "+"
```

ตั้งค่าในแท็บ **General:**
```
Chain: srcnat
Out. Interface: ether1 (หรือชื่อ Interface WAN ของคุณ)
```

ตั้งค่าในแท็บ **Action:**
```
Action: masquerade
```

กด **OK**

---

### 14.3 ทำความเข้าใจ Chain (เชน)

**Chain** (อ่านว่า เชน) แปลว่า "ลูกโซ่" หรือ "ห่วงโซ่"
ใน Firewall ของ MikroTik มี Chain สำคัญ 3 ประเภท:

| Chain | ความหมาย | ใช้กับ NAT แบบไหน |
|-------|----------|-----------------|
| srcnat | Source NAT = แปลง IP ต้นทาง | Masquerade (ลูกค้าออกอินเทอร์เน็ต) |
| dstnat | Destination NAT = แปลง IP ปลายทาง | Port Forwarding (เปิด Port) |
| input | Traffic เข้า Router | Firewall ป้องกัน Router |
| forward | Traffic ผ่าน Router | Firewall กรองข้อมูลลูกค้า |
| output | Traffic ออกจาก Router | ไม่ค่อยใช้ |

> 💡 **Tip:** Rule NAT Masquerade นี้ต้องมีแค่ **1 Rule เดียว** ก็พอสำหรับระบบพื้นฐาน
> ถ้าเพิ่มซ้ำ อาจเกิดปัญหาได้ ให้ตรวจสอบว่ามี Rule นี้แล้วหรือยังก่อนเพิ่ม

> ⚠️ **Note:** ถ้าใช้ Quick Setup หรือ Setup Wizard ของ MikroTik อาจสร้าง Rule NAT ให้อัตโนมัติแล้ว ให้ตรวจสอบก่อนเพิ่ม Rule ใหม่

---

### 14.4 ทดสอบว่า NAT ทำงาน

หลังตั้งค่า NAT แล้ว ทดสอบโดย:

**ใน MikroTik Terminal:**
```bash
/ping 8.8.8.8
```

ถ้า Ping ได้ → NAT และ WAN ทำงาน ✓
ถ้า Ping ไม่ได้ → ตรวจสอบ:
1. Interface WAN ได้รับ IP จาก ISP หรือยัง?
2. Default Route ตั้งค่าถูกต้องไหม?
3. NAT Rule ตั้ง Out. Interface ถูกต้องไหม?

---

## บทที่ 15 — Firewall Filter พื้นฐาน

---

### 15.1 Firewall คืออะไร?

**Firewall** (อ่านว่า ไฟร์-วอลล์) แปลตรงตัวว่า "กำแพงไฟ"
ทำหน้าที่เป็นด่านตรวจสอบข้อมูลที่ผ่านเข้า-ออก Router

ถ้าไม่มี Firewall:
- ใครก็ตามบนอินเทอร์เน็ตสามารถลองเชื่อมต่อเข้า MikroTik ได้
- Bot อัตโนมัติสแกนหา Router ที่ไม่มีการป้องกันตลอด 24 ชั่วโมง
- Router อาจถูกแฮ็กและใช้เป็นเครื่องมือโจมตีผู้อื่น

---

### 15.2 Default Firewall Rules ของ MikroTik

เมื่อลง RouterOS ใหม่ MikroTik จะมี Default Config ที่มี Firewall Rule พื้นฐานอยู่แล้ว

ดูได้ที่:
```
IP → Firewall → Filter Rules
```

Rule สำคัญที่ควรมี (ฝั่ง input = ป้องกัน Router):
```
chain=input  action=accept  connection-state=established,related
chain=input  action=drop    in-interface=ether1 (WAN)
```

ความหมาย:
- Rule 1: อนุญาตข้อมูลที่เป็นการตอบสนองจาก Connection ที่ Router เริ่มต้นเอง
- Rule 2: บล็อกการเชื่อมต่อใหม่จาก WAN เข้า Router โดยตรง

---

### 15.3 Connection State คืออะไร?

**Connection State** (อ่านว่า คอน-เน็ค-ชั่น สเตท) คือ สถานะของการเชื่อมต่อ:

| State | ความหมาย | ตัวอย่าง |
|-------|---------|---------|
| new | เชื่อมต่อใหม่ที่เพิ่งเริ่มต้น | คลิกลิงก์เว็บใหม่ |
| established | เชื่อมต่ออยู่ระหว่างรับส่งข้อมูล | ดาวน์โหลดไฟล์ที่กำลังโหลด |
| related | เชื่อมต่อเสริมที่เกี่ยวข้อง | FTP Data Connection |
| invalid | ผิดปกติ ไม่ตรงกับ Session ใด ๆ | อาจเป็นการโจมตี |

> 💡 **Tip:** Rule "accept established,related" เป็น Rule ที่ต้องมีเสมอ เพราะถ้าไม่มี MikroTik จะบล็อกข้อมูลที่ตอบกลับมาจากอินเทอร์เน็ตด้วย ทำให้เล่นเน็ตไม่ได้

---

### 15.4 Action ที่สำคัญใน Firewall

| Action | ความหมาย | ผลที่เกิด |
|--------|---------|---------|
| accept | ยอมรับ | ปล่อยผ่าน |
| drop | ทิ้ง | บล็อก ไม่แจ้งผู้ส่ง |
| reject | ปฏิเสธ | บล็อก แต่แจ้งผู้ส่งด้วย |
| log | บันทึก | บันทึก Log แล้วส่งต่อ Rule ถัดไป |

> ⚠️ **Note:** ควรใช้ `drop` แทน `reject` สำหรับ Traffic จากอินเทอร์เน็ต เพราะ `reject` จะแจ้งให้ผู้โจมตีรู้ว่า Router ยังมีอยู่ ทำให้เป็นเป้าหมายต่อไป

> 📌 **ไปศึกษาต่อ:** Firewall เป็นหัวข้อที่ซับซ้อนมาก สำหรับมือใหม่ให้ใช้ Default Rules ที่ MikroTik ตั้งมาให้ก่อน แล้วค่อยเรียนเพิ่มเติมในกลุ่มที่ 5 (ความมั่นคงปลอดภัย)

---

## บทที่ 16 — การ Backup และ Restore Config

---

### 16.1 ทำไมต้อง Backup?

การ Backup (แบ็ค-อัพ) Config คือสิ่งที่ **ต้องทำ** ก่อนการเปลี่ยนแปลงทุกครั้ง เพราะ:
- ถ้าตั้งค่าผิดพลาดและออกจากระบบ อาจเข้า Router ไม่ได้อีกเลย
- Router อาจเสีย, ไฟดับ, หรือ VM เสียหาย
- มี Backup = กู้คืนได้ใน 2 นาที
- ไม่มี Backup = ต้องตั้งค่าใหม่ทุกอย่างจากต้น

---

### 16.2 วิธี Backup (2 แบบ)

**แบบที่ 1 — Binary Backup (.backup) ← แนะนำ**

เก็บทุกอย่างรวมถึง Password ด้วย แต่ใช้คืนได้กับ RouterOS Version เดียวกันเท่านั้น

```
Winbox → Files → กด "Backup"
    → Name: ตั้งชื่อไฟล์ (เช่น backup-2026-06-01)
    → Password: ตั้ง Password ป้องกันไฟล์ (แนะนำ)
    → กด "Backup"
```

ไฟล์จะปรากฏใน File List → คลิกขวา → Download เก็บไว้ในเครื่อง

**แบบที่ 2 — Export (.rsc) — Text Config**

เก็บเป็นไฟล์ Text อ่านได้ แต่ **ไม่เก็บ Password**
ใช้ได้ข้ามเวอร์ชัน RouterOS ได้บ้าง

```
Terminal → พิมพ์:
/export file=config-backup-2026-06-01
```

ไฟล์จะอยู่ใน Files ให้ Download มาเก็บ

---

### 16.3 วิธี Restore (กู้คืน Config)

**กู้จาก Binary Backup:**
```
Files → ลาก .backup file เข้าไปใน Files
    → กด "Restore"
    → ใส่ Password (ถ้าตั้งไว้)
    → MikroTik จะ Reboot และโหลด Config เก่ากลับมา
```

**กู้จาก Export File (.rsc):**
```
Terminal → พิมพ์:
/import file-name=config-backup-2026-06-01.rsc
```

---

### 16.4 ตั้ง Backup อัตโนมัติด้วย Script

ถ้าต้องการ Backup อัตโนมัติทุกวัน:

```bash
# สร้าง Script ใน System → Scripts
/system script add name="daily-backup" source={
  /system backup save name=("backup-" . [/system clock get date])
}

# ตั้ง Scheduler รันทุกวันตี 2
/system scheduler add name="backup-scheduler" \
  interval=1d start-time=02:00:00 \
  on-event="daily-backup"
```

> 💡 **Tip:** Download Backup ออกไปเก็บใน Google Drive หรือ NAS ทุกสัปดาห์ อย่าเก็บแค่ใน Router เพราะถ้า Router เสียไปพร้อมกัน ไฟล์ก็หายด้วย

> ⚠️ **Note:** Backup แบบ Binary จะเก็บ Password ทั้งหมดรวมถึง Hotspot User ด้วย ให้ตั้ง Password ป้องกันไฟล์ Backup เสมอ

---

## ภาคผนวก — คำสั่ง Terminal ที่ใช้บ่อย

รวมคำสั่ง RouterOS CLI ที่มือใหม่ควรรู้:

```bash
# ===== ดูข้อมูลระบบ =====
/system resource print          # ดู CPU, RAM, Uptime
/system identity print          # ดูชื่อ Router
/system clock print             # ดูเวลาระบบ

# ===== Interface =====
/interface print                # ดู Interface ทั้งหมด
/interface monitor-traffic ether1   # ดู Traffic Real-time

# ===== IP =====
/ip address print               # ดู IP ที่ตั้งทั้งหมด
/ip route print                 # ดู Routing Table
/ip dhcp-server lease print     # ดูรายการ DHCP Lease

# ===== Firewall =====
/ip firewall filter print       # ดู Firewall Rules
/ip firewall nat print          # ดู NAT Rules

# ===== ทดสอบ =====
/ping 8.8.8.8                   # Ping ออกอินเทอร์เน็ต
/ping 192.168.88.10             # Ping ลูกค้า
/tool traceroute 8.8.8.8        # Traceroute

# ===== Backup =====
/export file=myconfig           # Export Config เป็น Text
/system backup save name=mybackup  # Backup แบบ Binary

# ===== Log =====
/log print                      # ดู Log ล่าสุด
/log print follow               # ดู Log แบบ Real-time (Ctrl+C เพื่อหยุด)
```

---

## สรุปภาพรวม — Config พื้นฐานที่ต้องมีก่อนทำ Hotspot

```
✅ Checklist ก่อนเปิด Hotspot ให้บริการ
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[ ] 1. ตั้งชื่อ Interface ให้ชัดเจน (WAN, LAN)
[ ] 2. Interface WAN ได้รับ IP จาก ISP แล้ว
[ ] 3. Interface LAN มี IP (เช่น 192.168.88.1/24)
[ ] 4. DHCP Server ทำงานบน LAN Interface
[ ] 5. DNS ตั้งค่า + ติ๊ก Allow Remote Requests
[ ] 6. NAT Masquerade สร้างแล้ว (srcnat → masquerade)
[ ] 7. ทดสอบ Ping จาก MikroTik ออกอินเทอร์เน็ตได้
[ ] 8. ทดสอบ PC หรือมือถือเชื่อมต่อ LAN → ได้ IP → เล่นเน็ตได้
[ ] 9. Backup Config เก็บไว้แล้ว
[ ] 10. เปลี่ยน Password ของ admin แล้ว (ไม่ใช้ค่า Default)
```

---

## แบบทดสอบตัวเอง

1. ความแตกต่างระหว่าง Winbox และ WebFig คืออะไร? แต่ละอันเหมาะกับสถานการณ์ไหน?
2. ทำไม CHR Free License ถึงใช้งานจริงไม่ได้? ต้องทำอย่างไร?
3. Bridge Interface คืออะไร? ต่างจาก Physical Interface อย่างไร?
4. ถ้าลูกค้าเชื่อมต่อ Wi-Fi ได้แต่เล่นเน็ตไม่ได้ ให้ตรวจสอบอะไรบ้าง?
5. ทำไมต้องติ๊ก "Allow Remote Requests" ใน DNS Settings?
6. ความแตกต่างระหว่าง Backup แบบ Binary (.backup) และ Export (.rsc)?
7. NAT Masquerade ต้องตั้ง Chain เป็นอะไร และ Out. Interface คืออะไร?

---

## แหล่งศึกษาต่อ

| แหล่ง | ลิงก์ / คำค้นหา |
|------|----------------|
| MikroTik Wiki อย่างเป็นทางการ | wiki.mikrotik.com |
| MikroTik Forum | forum.mikrotik.com |
| YouTube ภาษาไทย | ค้นหา "MikroTik พื้นฐาน", "ตั้งค่า MikroTik เบื้องต้น" |
| Facebook Group | "MikroTik Thailand", "MikroTik ไทยแลนด์" |
| Winbox ดาวน์โหลด | mikrotik.com/download |

---

## หัวข้อที่ต้องเรียนต่อในกลุ่มที่ 3

เมื่อตั้งค่าพื้นฐาน MikroTik ได้แล้ว ขั้นต่อไปคือ:

- [ ] การสร้าง Hotspot Profile
- [ ] การสร้าง User / Password สำหรับลูกค้า
- [ ] การตั้ง Walled Garden
- [ ] การปรับแต่งหน้า Login Page
- [ ] การตั้ง Session Limit และ Speed Limit
- [ ] การดู Active User และ Log

---

*เอกสารนี้จัดทำสำหรับผู้เรียน MikroTik Hotspot ระดับเริ่มต้น | อัปเดตล่าสุด: 2026*
