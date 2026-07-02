# กลุ่มที่ 4 — การจัดการและควบคุม Traffic
### สำหรับผู้เรียนที่เริ่มต้นจากศูนย์ | ต่อยอดจากกลุ่มที่ 1-3

---

> **📌 ข้อกำหนดก่อนเรียน**
> ควรเข้าใจพื้นฐาน IP Address, Interface, DHCP, NAT และระบบ Hotspot มาก่อน
> เนื้อหากลุ่มนี้จะช่วยให้คุณควบคุมได้ว่า "ใครได้ความเร็วเท่าไหร่" และ "ข้อมูลไหลอย่างไร"

---

## ทำไมต้องจัดการ Traffic?

ลองนึกภาพร้านคุณมีอินเทอร์เน็ต 100 Mbps และลูกค้า 30 คน
ถ้าไม่มีการควบคุมเลย:

```
ลูกค้าคนที่ 1  → ดาวน์โหลดหนัง 4K  → กิน 80 Mbps คนเดียว!
ลูกค้าคนที่ 2-30 → แต่ละคนได้แค่ ~0.6 Mbps → ใช้งานไม่ได้เลย
```

**การจัดการ Traffic** แก้ปัญหานี้ด้วยการ:
- จำกัดความเร็วต่อคน (เช่น คนละไม่เกิน 5 Mbps)
- จัดลำดับความสำคัญ (เช่น VoIP สำคัญกว่า YouTube)
- กรอง IP หรือ Protocol ที่ไม่ต้องการ

---

## บทที่ 26 — Queue คืออะไร? Simple Queue vs Queue Tree

---

### 26.1 Queue คืออะไร?

**Queue** (อ่านว่า คิว) แปลว่า "คิว" หรือ "แถวรอ"

ในทางเครือข่าย Queue คือ **กลไกควบคุมการไหลของข้อมูล**
โดยการกำหนดว่าข้อมูลจะถูกส่งออกไปได้เร็วแค่ไหน

**อุปมา:**
นึกถึงท่อน้ำ:
- ท่อหลักของ ISP = 100 Mbps
- ถ้าไม่มีวาล์วควบคุม ใครก็ดึงน้ำได้ไม่จำกัด
- Queue = **วาล์วควบคุม** ที่กำหนดว่าแต่ละคนดึงน้ำได้เท่าไหร่

```
อินเทอร์เน็ต 100 Mbps
        │
    [MikroTik Queue]
        │
   ┌────┴────┐
   ↓         ↓
ลูกค้า A   ลูกค้า B
max 5Mbps  max 5Mbps
```

---

### 26.2 ประเภทของ Queue ใน MikroTik

MikroTik มี Queue 2 แบบหลัก:

| | Simple Queue | Queue Tree |
|--|-------------|------------|
| ความซับซ้อน | ง่าย | ซับซ้อนกว่า |
| เหมาะกับ | มือใหม่ จำกัดรายคน | มืออาชีพ จัดลำดับ Priority |
| ต้องใช้ Mangle | ไม่ต้อง | **ต้องใช้** |
| จัดการแบบลำดับชั้น | ไม่ได้ | ได้ (Parent/Child) |
| ใช้กับ Hotspot | ได้ดี | ได้ดีมากกว่า |

---

### 26.3 Simple Queue — จำกัดความเร็วรายคน

**Simple Queue** (อ่านว่า ซิม-เพิล คิว) คือวิธีที่ง่ายที่สุดในการจำกัดความเร็ว

**ดูใน Winbox:**
```
Queues → Simple Queues
```

#### การสร้าง Simple Queue สำหรับลูกค้า 1 คน:

```
Queues → Simple Queues → กด "+"

แท็บ General:
    Name: client-192.168.88.10
    Target: 192.168.88.10/32
        (IP ของลูกค้าคนนั้น /32 หมายถึง 1 IP เดียว)

    Max Limit:
        Target Upload: 5M    ← ความเร็ว Upload สูงสุด (ลูกค้าส่งขึ้น)
        Target Download: 10M ← ความเร็ว Download สูงสุด (ลูกค้ารับลง)

กด OK
```

> 💡 **Tip — ตัวย่อขนาด:**
> - `k` หรือ `K` = Kilobits per second (กิโลบิตต่อวินาที)
> - `M` = Megabits per second (เมกะบิตต่อวินาที)
> - `G` = Gigabits per second (กิกะบิตต่อวินาที)
>
> ตัวอย่าง: `5M` = 5 Mbps, `512k` = 512 Kbps

---

### 26.4 Simple Queue สำหรับทั้งเครือข่าย (จำกัดรวม)

ถ้าต้องการจำกัดว่าทุกคนรวมกันใช้ได้ไม่เกิน 80 Mbps (เหลือ 20 Mbps ไว้ใช้งานอื่น):

```
Queues → Simple Queues → กด "+"

    Name: total-bandwidth-limit
    Target: 192.168.88.0/24
        (ทั้ง Subnet = ทุกคนในวงนี้รวมกัน)

    Max Limit:
        Target Upload: 80M
        Target Download: 80M

กด OK
```

> ⚠️ **Note:** Queue ที่ Target เป็น Subnet (/24) จะจำกัดรวมทั้งหมด ไม่ใช่รายคน
> ถ้าต้องการจำกัดรายคนต้องสร้าง Queue ที่ Target เป็น /32 ของแต่ละ IP

---

### 26.5 Queue Tree — ระบบขั้นสูง

**Queue Tree** (อ่านว่า คิว ทรี) แปลว่า "คิวแบบต้นไม้" เพราะมีโครงสร้างแบบ Parent-Child (พ่อแม่-ลูก)

**ทำไมถึงดีกว่า Simple Queue:**

```
Simple Queue:           Queue Tree:
ลูกค้า A → 5Mbps      [รวมทั้งหมด 100Mbps] ← Parent
ลูกค้า B → 5Mbps          │
ลูกค้า C → 5Mbps     ┌────┼────┐
(ต่างคนต่างทำงาน)    A    B    C    ← Children
                     5M   5M   5M
                  (แบ่งจาก Parent อัจฉริยะ)
```

Queue Tree ต้องทำงานร่วมกับ **Mangle** (ดูบทที่ 31)
เพื่อ Mark (ติดป้าย) Traffic ก่อนแล้วค่อย Queue

> 📌 **ไปศึกษาต่อ:** Queue Tree เป็นเรื่องขั้นกลาง-สูง แนะนำให้เข้าใจ Simple Queue ให้ดีก่อน แล้วค่อยเรียน Queue Tree ตอนมีพื้นฐานแน่นขึ้น

---

## บทที่ 27 — การจำกัด Bandwidth รายคน / รายกลุ่ม

---

### 27.1 วิธีคิดก่อนตั้งค่า Bandwidth

ก่อนตั้งค่าควรวางแผน:

```
อินเทอร์เน็ตที่มี:         100 Mbps

ลูกค้าสูงสุดพร้อมกัน:     50 คน
ความเร็วต่อคน (อุดมคติ):  100/50 = 2 Mbps ต่อคน

แต่ในความเป็นจริง ไม่ใช่ทุกคนใช้พร้อมกัน 100%
จึงตั้งได้สูงกว่า:         5-10 Mbps ต่อคน (แล้วแต่ Package)
```

**ตัวอย่าง Package ที่นิยมทำ:**

| Package | Download | Upload | เวลา | ราคา |
|---------|----------|--------|------|------|
| Basic | 2 Mbps | 1 Mbps | 1 ชั่วโมง | 20 บาท |
| Standard | 5 Mbps | 2 Mbps | 3 ชั่วโมง | 40 บาท |
| Premium | 10 Mbps | 5 Mbps | 1 วัน | 80 บาท |
| Unlimited | 20 Mbps | 10 Mbps | ไม่จำกัด | 150 บาท |

---

### 27.2 การผูก Speed Limit กับ Hotspot User Profile

วิธีที่ดีที่สุดสำหรับระบบ Hotspot คือผูก Bandwidth Limit ไว้กับ **User Profile**
(โปรไฟล์ผู้ใช้) เพื่อที่เวลาสร้าง User ใหม่ แค่เลือก Profile ก็ได้ Speed ที่กำหนดไว้เลย

**สร้าง Hotspot User Profile พร้อม Speed Limit:**

```
Winbox → IP → Hotspot → แท็บ "User Profiles" → กด "+"

    Name: package-5mbps

    แท็บ General:
        Rate Limit: 5M/2M
        (รูปแบบ: Download/Upload)
        (หมายถึง Download 5Mbps, Upload 2Mbps)

    กด OK
```

**ตัวอย่าง Rate Limit Format:**

```
5M/2M        = Download 5Mbps / Upload 2Mbps
10M/5M       = Download 10Mbps / Upload 5Mbps
1M/512k      = Download 1Mbps / Upload 512Kbps
2M/1M/1M/512k/8/8  = มี Burst ด้วย (ดูบทที่ 28)
```

---

### 27.3 การผูก Queue กับ IP หลาย ๆ คนพร้อมกัน (PCQ)

**PCQ** (อ่านว่า พี-ซี-คิว) ย่อมาจาก **Per Connection Queue**
(เพอร์ คอน-เน็ค-ชั่น คิว)

แปลว่า: **"จำกัด Bandwidth โดยอัตโนมัติแบบเท่าเทียมต่อการเชื่อมต่อ"**

**ทำไมต้องใช้ PCQ:**
- Simple Queue ต้องสร้าง Rule ทีละ IP → ถ้ามีลูกค้า 100 คน ต้องสร้าง 100 Rule!
- PCQ สร้าง Rule เดียว ระบบแบ่งความเร็วให้ทุกคนเท่ากันอัตโนมัติ

**สร้าง PCQ Queue Type:**
```
Queues → Queue Types → กด "+"

    Name: pcq-download
    Kind: pcq
    Rate: 5M         ← ความเร็ว Download ต่อ 1 Connection
    Classifier: dst-address   ← แบ่งตาม IP ปลายทาง (ฝั่งรับ)

กด "+" อีกครั้ง:
    Name: pcq-upload
    Kind: pcq
    Rate: 2M
    Classifier: src-address   ← แบ่งตาม IP ต้นทาง (ฝั่งส่ง)
```

**สร้าง Simple Queue ใช้ PCQ:**
```
Queues → Simple Queues → กด "+"

    Name: pcq-all-clients
    Target: 192.168.88.0/24

    Queue Type:
        Target Upload: pcq-upload
        Target Download: pcq-download

กด OK
```

> 💡 **Tip:** PCQ เหมาะมากสำหรับ Hotspot ที่มีลูกค้าไม่แน่นอน เพราะไม่ต้องสร้าง Queue รายคน ระบบจะแบ่งปันแบนด์วิธให้เท่ากันเองโดยอัตโนมัติ

> ❓ **คำถามทิ้งไว้:** ถ้าตั้ง PCQ Rate ไว้ที่ 5M แต่มีลูกค้าใช้งานอยู่เพียงคนเดียว เขาจะได้ความเร็วเท่าไหร่? จะได้แค่ 5M หรือมากกว่านั้น?

---

## บทที่ 28 — Burst คืออะไร? ตั้งค่าอย่างไร?

---

### 28.1 Burst คืออะไร?

**Burst** (อ่านว่า เบิร์สต์) แปลว่า "การพุ่ง" หรือ "ระเบิดความเร็ว"

Burst คือการ **อนุญาตให้ลูกค้าใช้ความเร็วสูงกว่าที่กำหนดได้ชั่วคราว**
แล้วค่อยลดลงมาที่ความเร็วปกติ

**ทำไมต้องมี Burst:**
- การเปิดเว็บต้องการความเร็วสูงในช่วงแรก (โหลดหน้าเว็บ)
- หลังจากนั้นใช้ความเร็วต่ำกว่ามาก (อ่านเนื้อหา)
- Burst ทำให้ประสบการณ์ดีขึ้น โดยไม่ต้องเพิ่มความเร็วหลักให้แพงขึ้น

**เปรียบเทียบ:**

```
ไม่มี Burst:          มี Burst:
▄▄▄▄▄▄▄▄▄▄▄▄▄        ████
5Mbps ตลอด           10Mbps ช่วงแรก 10 วินาที
                      ▄▄▄▄▄▄▄▄▄▄
                      5Mbps หลังจากนั้น
```

---

### 28.2 พารามิเตอร์ของ Burst

| พารามิเตอร์ | อ่านว่า | ความหมาย |
|------------|--------|---------|
| Max Limit | แม็กซ์ ลิมิต | ความเร็วปกติสูงสุด |
| Burst Limit | เบิร์สต์ ลิมิต | ความเร็วสูงสุดตอน Burst |
| Burst Threshold | เบิร์สต์ เธรส-โฮลด์ | เส้นแบ่งว่าจะ Burst หรือไม่ |
| Burst Time | เบิร์สต์ ไทม์ | ช่วงเวลาที่ระบบดูค่าเฉลี่ย (วินาที) |

**หลักการทำงาน:**
```
ถ้า ความเร็วเฉลี่ยใน Burst Time < Burst Threshold
    → อนุญาตให้ใช้ความเร็วได้ถึง Burst Limit

ถ้า ความเร็วเฉลี่ยใน Burst Time >= Burst Threshold
    → จำกัดที่ Max Limit ตามปกติ
```

---

### 28.3 ตัวอย่างการตั้งค่า Burst

สถานการณ์: ต้องการให้ลูกค้า Package 5M ได้รับ Burst ที่ 10M ในช่วง 10 วินาทีแรก

```
Queues → Simple Queues → ดับเบิ้ลคลิกที่ Queue ที่ต้องการแก้ไข

แท็บ General:
    Max Limit:
        ↑ (Upload): 2M
        ↓ (Download): 5M

    Burst Limit:
        ↑ (Upload): 4M
        ↓ (Download): 10M

    Burst Threshold:
        ↑ (Upload): 1M
        ↓ (Download): 2.5M

    Burst Time:
        ↑ (Upload): 8
        ↓ (Download): 8

กด OK
```

**อธิบายในตัวอย่างนี้:**
- ปกติ: Download 5M, Upload 2M
- Burst ได้ถึง: Download 10M, Upload 4M
- ระบบดูค่าเฉลี่ยใน 8 วินาที
- ถ้าเฉลี่ยต่ำกว่า Threshold (2.5M สำหรับ DL) → ปล่อย Burst

**ใน Rate Limit Format (สำหรับ Hotspot Profile):**
```
Rate Limit: 5M/2M 10M/4M 2500k/1M 8 8
            │     │       │         │
            │     │       │         └─ Burst Time (DL/UL ใช้ค่าเดียวกัน)
            │     │       └─ Burst Threshold (DL/UL)
            │     └─ Burst Limit (DL/UL)
            └─ Max Limit (DL/UL)
```

> 💡 **Tip:** ค่า Burst Threshold ควรตั้งที่ครึ่งหนึ่งของ Max Limit
> และ Burst Limit ควรตั้งเป็น 2 เท่าของ Max Limit เพื่อให้รู้สึกได้ถึงความต่าง

> ⚠️ **Note:** Burst ไม่ใช่ความเร็วฟรี ระบบดูค่าเฉลี่ย ถ้าใช้ Burst ไปมาก ๆ จะถูกลดความเร็วลงมาต่ำกว่า Max Limit ชั่วคราว เพื่อให้ค่าเฉลี่ยสมดุล

> ❓ **คำถามทิ้งไว้:** ถ้าตั้ง Burst Time สั้นมาก เช่น 4 วินาที เทียบกับ Burst Time ยาว เช่น 16 วินาที จะมีผลต่อประสบการณ์ลูกค้าอย่างไร?

---

## บทที่ 29 — IP Binding — ล็อก IP ให้ User คนใดคนหนึ่ง

---

### 29.1 IP Binding คืออะไร?

**IP Binding** (อ่านว่า ไอพี ไบน์-ดิ้ง) แปลว่า "การผูก IP"

ใน Hotspot, IP Binding ทำให้คุณ:
1. **ล็อก IP คงที่ให้กับอุปกรณ์** (ตาม MAC Address)
2. **บายพาส Hotspot Login** ให้บางอุปกรณ์ (ไม่ต้อง Login)
3. **บล็อกอุปกรณ์บางตัว** ไม่ให้ใช้งาน

---

### 29.2 ประเภทของ IP Binding

| Type | ความหมาย | ใช้เมื่อ |
|------|---------|---------|
| regular | ปกติ (ต้อง Login) | ลูกค้าทั่วไป |
| bypassed | ข้าม Login ได้เลย | อุปกรณ์ที่ไว้ใจ เช่น Printer, NAS, POS |
| blocked | บล็อก ห้ามใช้งาน | อุปกรณ์ที่ต้องการแบน |

---

### 29.3 วิธีสร้าง IP Binding

**กรณีที่ 1 — ให้อุปกรณ์ Bypass ไม่ต้อง Login (เช่น เครื่อง POS ในร้าน):**

```
IP → Hotspot → แท็บ "IP Bindings" → กด "+"

    MAC Address: AA:BB:CC:DD:EE:FF
        (MAC ของเครื่อง POS)
    Address: 192.168.88.200
        (IP ที่ต้องการล็อกให้)
    Type: bypassed

กด OK
```

**กรณีที่ 2 — บล็อกอุปกรณ์:**
```
IP → Hotspot → IP Bindings → กด "+"

    MAC Address: XX:XX:XX:XX:XX:XX
    Type: blocked

กด OK
```

---

### 29.4 วิธีหา MAC Address ของอุปกรณ์

**จาก DHCP Leases:**
```
IP → DHCP Server → Leases
→ มองคอลัมน์ "MAC Address"
```

**จาก Hotspot Active Users:**
```
IP → Hotspot → Active
→ มองคอลัมน์ "MAC Address"
```

**จากอุปกรณ์โดยตรง:**
- Windows: `ipconfig /all` → Physical Address
- Android: Settings → About Phone → WiFi MAC
- iPhone: Settings → General → About → Wi-Fi Address

> 💡 **Tip:** ควรสร้าง IP Binding สำหรับเครื่องที่ใช้จัดการระบบ (PC Admin) ด้วย เพื่อให้ได้ IP คงที่เสมอและ Bypass Login Hotspot ไปเลย

> ⚠️ **Note:** MAC Address สามารถ "ปลอมแปลง" ได้ (MAC Spoofing / แมค สปูฟ-ฟิ้ง) ดังนั้นอย่าพึ่ง IP Binding เป็นระบบ Security หลัก ใช้เป็นแค่ความสะดวกเท่านั้น

---

## บทที่ 30 — Address List — การจัดกลุ่ม IP

---

### 30.1 Address List คืออะไร?

**Address List** (อ่านว่า แอด-เดรส ลิสต์) แปลว่า "รายการที่อยู่"

คือการ **รวบรวม IP Address หรือ IP Range หลาย ๆ อันเข้าเป็นกลุ่มเดียว**
เพื่อใช้ใน Firewall Rule หรือ Queue Rule ได้ง่ายขึ้น

**ทำไมต้องใช้:**
- แทนที่จะเขียน Firewall Rule 10 บรรทัดสำหรับ 10 IP
- สร้าง Address List 1 กลุ่ม แล้วอ้างถึงกลุ่มนั้นแค่ครั้งเดียว

---

### 30.2 ตัวอย่างการใช้งาน Address List

**กรณีที่ 1 — Blacklist (แบล็ค-ลิสต์) เว็บที่ไม่ต้องการ:**

สมมติต้องการบล็อก IP ของบางเว็บ:
```
IP → Firewall → Address Lists → กด "+"

    Name: blacklist-sites
    Address: 203.151.1.1
กด OK

กด "+" อีกครั้ง:
    Name: blacklist-sites
    Address: 103.249.xx.xx
กด OK
```

แล้วสร้าง Firewall Rule อ้างถึง List:
```
IP → Firewall → Filter Rules → กด "+"

    Chain: forward
    Dst. Address List: blacklist-sites
    Action: drop
```

**กรณีที่ 2 — VIP List สำหรับลูกค้า Premium:**
```
สร้าง Address List ชื่อ "vip-clients"
เพิ่ม IP ของ VIP ทุกคนเข้าไป
สร้าง Queue Rule พิเศษให้กับ List นี้
```

---

### 30.3 Address List แบบ Dynamic (อัตโนมัติจาก Firewall)

MikroTik สามารถเพิ่ม IP เข้า Address List อัตโนมัติเมื่อมีเงื่อนไขตรงได้

**ตัวอย่าง — เพิ่ม IP ที่พยายาม Login ผิดเกิน 3 ครั้งเข้า Blacklist:**

```
IP → Firewall → Filter Rules → กด "+"

    Chain: input
    Protocol: tcp
    Dst. Port: 8291    (Winbox Port)
    Connection Limit: 3,32
    Action: add-src-to-address-list
    Address List: brute-force-attackers
    Timeout: 1d        (เก็บไว้ 1 วัน)
```

แล้วสร้าง Rule บล็อก List นั้น:
```
Chain: input
Src. Address List: brute-force-attackers
Action: drop
```

> 💡 **Tip:** Address List เป็นเครื่องมือที่ทรงพลังมาก เรียนรู้การใช้งานเพิ่มเติมจะช่วยให้ตั้ง Firewall ได้ยืดหยุ่นมากขึ้น

> ❓ **คำถามทิ้งไว้:** ถ้าต้องการบล็อกทุก IP ในประเทศใดประเทศหนึ่งไม่ให้เข้ามาใช้งาน จะใช้ Address List ช่วยได้อย่างไร? ค้นหาคำว่า "MikroTik Country Block" เพื่อหาคำตอบ

---

## บทที่ 31 — Mangle — การ Mark Traffic เบื้องต้น

---

### 31.1 Mangle คืออะไร?

**Mangle** (อ่านว่า แมง-เกิล) แปลว่า "การดัดแปลง" หรือ "การติดป้ายกำกับ"

Mangle คือระบบของ MikroTik ที่ใช้ **"ติดป้าย" (Mark) ให้กับ Packet** ที่ผ่านมา
เพื่อให้ระบบอื่น (เช่น Queue Tree หรือ Routing) รู้ว่าต้องจัดการ Packet นั้นอย่างไร

**อุปมา:**
นึกถึงสายพานในโรงงาน:
- สินค้าทุกชิ้นวิ่งบนสายพานเดียวกัน
- Mangle คือคนที่ **ติดสติ๊กเกอร์สี** บนสินค้า
  - สีแดง = VIP ส่งด่วน
  - สีเขียว = ปกติ
  - สีเหลือง = ลูกค้า Premium
- Queue Tree คือคนที่ดูสี และแยกเส้นทางให้

```
Packet เข้ามา
      │
   [Mangle]
   ติดป้าย "mark-vip" หรือ "mark-normal"
      │
   [Queue Tree]
   เห็นป้าย "mark-vip" → ให้ความเร็วพิเศษ
   เห็นป้าย "mark-normal" → ความเร็วปกติ
```

---

### 31.2 Chain ใน Mangle

| Chain | ใช้เมื่อ |
|-------|---------|
| prerouting | Traffic ทุกอย่างก่อนตัดสินว่าจะส่งไปไหน |
| input | Traffic ที่มุ่งเข้า Router |
| forward | Traffic ที่ผ่าน Router ไปยังปลายทาง |
| output | Traffic ที่ออกจาก Router |
| postrouting | Traffic ทุกอย่างหลังตัดสินทิศทางแล้ว |

สำหรับการจำกัด Bandwidth ลูกค้า ส่วนใหญ่ใช้ **forward**

---

### 31.3 ประเภทของ Mark

| Mark | ความหมาย | ใช้กับ |
|------|---------|-------|
| mark-connection | ติดป้าย Connection | Queue Tree, Routing |
| mark-packet | ติดป้าย Packet แต่ละอัน | Queue Tree |
| mark-routing | ติดป้ายสำหรับ Routing | Policy Routing |

---

### 31.4 ตัวอย่างการใช้ Mangle + Queue Tree

**โจทย์:** ต้องการแบ่ง Bandwidth รวม 100M ออกเป็น:
- Upload รวม: 30M (แบ่งให้ทุกคนเท่า ๆ กัน)
- Download รวม: 80M (แบ่งให้ทุกคนเท่า ๆ กัน)

**ขั้นตอนที่ 1 — สร้าง Mangle Mark:**

```
IP → Firewall → Mangle → กด "+"

Rule ที่ 1 (Mark Download):
    Chain: forward
    In. Interface: ether1    (WAN เข้า)
    Action: mark-packet
    New Packet Mark: download-traffic
    Passthrough: yes

Rule ที่ 2 (Mark Upload):
    Chain: forward
    Out. Interface: ether1   (WAN ออก)
    Action: mark-packet
    New Packet Mark: upload-traffic
    Passthrough: yes
```

**ขั้นตอนที่ 2 — สร้าง Queue Tree Parent:**

```
Queues → Queue Tree → กด "+"

Parent Queue (รวมทั้งหมด):
    Name: wan-total
    Parent: ether1    (Interface WAN)
    Max Limit: 100M
    Priority: 8
```

**ขั้นตอนที่ 3 — สร้าง Child Queue:**

```
Child Download:
    Name: all-download
    Parent: wan-total
    Packet Marks: download-traffic
    Queue Type: pcq-download
    Max Limit: 80M
    Priority: 8

Child Upload:
    Name: all-upload
    Parent: wan-total
    Packet Marks: upload-traffic
    Queue Type: pcq-upload
    Max Limit: 30M
    Priority: 8
```

> ⚠️ **Note:** Queue Tree ต้องระบุ Parent Interface ให้ถูกต้อง ถ้าระบุผิด Queue จะไม่ทำงานและไม่มี Error บอก ทำให้แก้ปัญหายาก

> 📌 **ไปศึกษาต่อ:** Mangle เป็นหัวข้อที่ลึกมาก มีการใช้งานหลากหลาย เช่น Policy Routing, Load Balancing, QoS สำหรับ VoIP แนะนำให้ศึกษาเพิ่มเติมจาก wiki.mikrotik.com/wiki/Manual:IP/Firewall/Mangle

---

## ภาคผนวก A — การแก้ปัญหา Queue ที่พบบ่อย

---

### ปัญหาที่ 1: Queue ตั้งไว้แต่ไม่ทำงาน (ลูกค้ายังเร็วเต็มที่)

**สาเหตุที่พบบ่อย:**

| สาเหตุ | วิธีตรวจสอบ | วิธีแก้ |
|-------|-----------|--------|
| Queue ถูก Disable | ดูว่าไอคอนเป็นสีเทาไหม | คลิกขวา → Enable |
| Target IP ผิด | ตรวจสอบ IP ลูกค้า | แก้ Target ใน Queue |
| ลูกค้าต่อ Interface ที่ไม่ได้ Monitor | ดู Interface ใน Queue | แก้ Interface ใน Queue |
| มี Queue อื่น Override | ดูลำดับ Queue | จัดลำดับ Queue ใหม่ |

**วิธีทดสอบว่า Queue ทำงาน:**
```
Queues → Simple Queues
→ ดูคอลัมน์ "Bytes" และ "Packets"
→ ถ้าตัวเลขเปลี่ยนไปเรื่อย ๆ = Queue กำลังทำงาน
→ ถ้าค้างที่ 0 = Queue ไม่ได้รับ Traffic
```

---

### ปัญหาที่ 2: ลูกค้าเล่นเน็ตได้ แต่ช้ามากผิดปกติ

**ตรวจสอบ:**
```
Queues → Simple Queues → ดูที่ Queue ของลูกค้าคนนั้น
    → กราฟสีแดง = ถูกจำกัดความเร็วอยู่
    → ถ้า Max Limit ต่ำเกินไป → แก้ไข
```

---

### ปัญหาที่ 3: Burst ไม่ทำงาน

**สาเหตุที่พบบ่อย:**
- Burst Threshold สูงกว่า Max Limit → ระบบไม่มีทาง Burst ได้
- Burst Time = 0 → Burst ถูก Disable โดยปริยาย

**ตรวจสอบ:**
- Burst Threshold ต้องน้อยกว่า Max Limit เสมอ
- Burst Limit ต้องมากกว่า Max Limit เสมอ

---

## ภาคผนวก B — คำสั่ง Terminal ที่ใช้กับ Queue

```bash
# ดู Queue ทั้งหมด
/queue simple print

# ดู Queue พร้อมสถิติ Traffic
/queue simple print stats

# ดูแบบ Real-time
/queue simple monitor [find name="ชื่อ-queue"]

# เพิ่ม Simple Queue ผ่าน CLI
/queue simple add name="client-10" \
    target=192.168.88.10/32 \
    max-limit=5M/2M

# แก้ไข Queue
/queue simple set [find name="client-10"] max-limit=10M/5M

# ลบ Queue
/queue simple remove [find name="client-10"]

# ดู Mangle Rules
/ip firewall mangle print

# ดู Address List
/ip firewall address-list print

# เพิ่ม IP เข้า Address List
/ip firewall address-list add \
    list=blacklist-sites \
    address=203.151.1.1
```

---

## ภาคผนวก C — ตัวอย่าง Config สมบูรณ์สำหรับ Hotspot พื้นฐาน

สถานการณ์: ร้านกาแฟ มีอินเทอร์เน็ต 100 Mbps ลูกค้าสูงสุด 50 คน
Package เดียว: Download 5M, Upload 2M พร้อม Burst

**ขั้นตอนที่ 1 — สร้าง PCQ Queue Types:**
```bash
/queue type add name=pcq-dl kind=pcq pcq-rate=5M pcq-classifier=dst-address
/queue type add name=pcq-ul kind=pcq pcq-rate=2M pcq-classifier=src-address
```

**ขั้นตอนที่ 2 — สร้าง Simple Queue ครอบทั้งหมด:**
```bash
/queue simple add \
    name=hotspot-all-clients \
    target=192.168.88.0/24 \
    queue=pcq-ul/pcq-dl \
    max-limit=100M/100M
```

**ขั้นตอนที่ 3 — ตั้ง Speed Limit ใน Hotspot Profile:**
```
IP → Hotspot → User Profiles → package-5m
    Rate Limit: 5M/2M 10M/4M 2500k/1000k 8 8
```

**ผลลัพธ์:**
- ลูกค้าทุกคนได้ Download 5M, Upload 2M
- มี Burst ที่ 10M/4M ช่วงแรก
- รวมทุกคนไม่เกิน 100M

---

## สรุปภาพรวม — เลือกใช้เครื่องมืออะไร?

```
ต้องการควบคุม Traffic แบบไหน?
            │
    ┌───────┼────────────┐
    ↓       ↓            ↓
จำกัด    จำกัดหลาย    จัดการขั้นสูง
รายคน    คนพร้อมกัน   (Priority/Policy)
    │       │            │
Simple   PCQ Queue    Mangle +
Queue    Type         Queue Tree
    │       │            │
เหมาะกับ  เหมาะกับ    เหมาะกับ
IP คงที่  Hotspot     ISP/Enterprise
ไม่กี่คน ลูกค้าเยอะ
```

---

## แบบทดสอบตัวเอง

1. ความแตกต่างระหว่าง Simple Queue และ Queue Tree คืออะไร?
2. ถ้าต้องการจำกัด Download ที่ 5Mbps และ Upload ที่ 2Mbps ให้ลูกค้า IP 192.168.88.50 จะตั้งค่า Simple Queue อย่างไร?
3. PCQ ทำงานต่างจาก Simple Queue ธรรมดาอย่างไร? เหมาะกับสถานการณ์แบบไหน?
4. Burst คืออะไร? Burst Limit ต้องมีค่ามากกว่าหรือน้อยกว่า Max Limit?
5. IP Binding ประเภท "bypassed" ใช้ทำอะไร? ยกตัวอย่างอุปกรณ์ที่ควรตั้งแบบนี้
6. Mangle ทำหน้าที่อะไร? ทำงานร่วมกับ Queue Tree อย่างไร?
7. Address List มีประโยชน์อย่างไรเมื่อเทียบกับการระบุ IP ทีละตัวใน Firewall Rule?

---

## แหล่งศึกษาต่อ

| แหล่ง | หัวข้อที่แนะนำ |
|------|--------------|
| wiki.mikrotik.com | Manual:Queue, Manual:Mangle, PCQ |
| YouTube | "MikroTik Simple Queue", "MikroTik PCQ Tutorial", "MikroTik Mangle" |
| forum.mikrotik.com | ค้นหา "bandwidth management" |
| Facebook Group MikroTik Thailand | ถามตอบปัญหาเฉพาะ |

---

## หัวข้อที่ต้องเรียนต่อในกลุ่มที่ 5

เมื่อจัดการ Traffic ได้แล้ว ขั้นต่อไปคือความมั่นคงปลอดภัย:

- [ ] การเปลี่ยน Password Admin
- [ ] การปิด Service ที่ไม่ใช้ (Telnet, FTP, API)
- [ ] Firewall ป้องกัน Router จากภายนอก
- [ ] การป้องกัน Brute Force Login
- [ ] VLAN เบื้องต้น — แยก Network ลูกค้าออกจาก Admin

---

*เอกสารนี้จัดทำสำหรับผู้เรียน MikroTik Hotspot ระดับเริ่มต้น | อัปเดตล่าสุด: 2026*
