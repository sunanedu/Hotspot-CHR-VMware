# คู่มือการติดตั้งระบบ Hotspot ด้วย Mikrotik CHR บน VMware

> **เวอร์ชัน:** 1.0  
> **RouterOS:** 7.21.4  
> **สภาพแวดล้อม:** VMware Workstation / VMware Player  
> **ระบบปฏิบัติการ Client:** Windows 7  
> **Radius Server:** Ubuntu 22.04 LTS

---

## สารบัญ

1. [ดาวน์โหลด Mikrotik CHR](#1-ดาวน์โหลด-mikrotik-chr)
2. [สร้างเครื่องจำลอง VMware Mikrotik](#2-สร้างเครื่องจำลอง-vmware-mikrotik)
3. [เริ่มต้นระบบ Mikrotik OS และตั้งค่าเบื้องต้น](#3-เริ่มต้นระบบ-mikrotik-os-และตั้งค่าเบื้องต้น)
4. [เปิด Client เครื่อง VM Windows 7 ใน VMware](#4-เปิด-client-เครื่อง-vm-windows-7-ใน-vmware)
5. [ตั้งค่า Network ระหว่าง Windows 7 กับ VM Mikrotik CHR](#5-ตั้งค่า-network-ระหว่าง-windows-7-กับ-vm-mikrotik-chr)
6. [การใช้โปรแกรม Winbox](#6-การใช้โปรแกรม-winbox)
7. [Mikrotik CHR ตั้งค่า Bridge Network](#7-mikrotik-chr-ตั้งค่า-bridge-network)
8. [การตั้งค่า Hotspot ของ VM Mikrotik CHR](#8-การตั้งค่า-hotspot-ของ-vm-mikrotik-chr)
9. [VM Windows 7 กับการเข้าใช้งานบริการ Hotspot](#9-vm-windows-7-กับการเข้าใช้งานบริการ-hotspot)
10. [การปรับแต่งระบบของ Hotspot ของ Mikrotik](#10-การปรับแต่งระบบของ-hotspot-ของ-mikrotik)
11. [การใช้งาน Radius Server เป็น Linux Ubuntu 22.04](#11-การใช้งาน-radius-server-เป็น-linux-ubuntu-2204)

---

## 1. ดาวน์โหลด Mikrotik CHR

### CHR คืออะไร?

**Cloud Hosted Router (CHR)** คือ RouterOS เวอร์ชันที่ออกแบบมาสำหรับรันบน Virtual Machine โดยรองรับ VMware, Hyper-V, VirtualBox และ KVM รองรับฟีเจอร์ครบเหมือน RouterOS ปกติ เหมาะสำหรับการทดสอบและ Lab

### ขั้นตอนการดาวน์โหลด

**ดาวน์โหลดไฟล์ VMDK สำหรับ VMware:**

```
https://download.mikrotik.com/routeros/7.21.4/chr-7.21.4.vmdk.zip
```

หรือดาวน์โหลดจากหน้าเว็บทางการ:

```
https://mikrotik.com/download
```

เลือกหัวข้อ **Cloud Hosted Router (CHR)** → เลือกเวอร์ชัน **7.21.4** → ดาวน์โหลดไฟล์ `.vmdk.zip`

### แตกไฟล์

หลังดาวน์โหลดเสร็จ แตกไฟล์ ZIP จะได้ไฟล์:

```
chr-7.21.4.vmdk
```

เก็บไฟล์นี้ไว้ในโฟลเดอร์ที่ต้องการ เช่น `C:\VMs\Mikrotik\`

---

## 2. สร้างเครื่องจำลอง VMware Mikrotik

### ความต้องการของระบบ

| รายการ | ขั้นต่ำ | แนะนำ |
|--------|---------|-------|
| CPU | 1 Core | 2 Cores |
| RAM | 128 MB | 256 MB |
| Disk | 1 GB (VMDK ที่ดาวน์โหลด) | - |
| Network | 2 NICs | 3 NICs |

### ขั้นตอนสร้าง VM ใน VMware Workstation

**ขั้นที่ 1: สร้าง VM ใหม่**

1. เปิด VMware Workstation
2. คลิก **File** → **New Virtual Machine** (หรือกด `Ctrl+N`)
3. เลือก **Custom (advanced)** → คลิก **Next**

**ขั้นที่ 2: เลือกประเภทการติดตั้ง**

4. Hardware compatibility: เลือกเวอร์ชัน VMware ที่ใช้อยู่ → **Next**
5. Install from: เลือก **I will install the operating system later** → **Next**

**ขั้นที่ 3: ตั้งค่าระบบปฏิบัติการ**

6. Guest OS: เลือก **Linux**
7. Version: เลือก **Other Linux 5.x kernel 64-bit** → **Next**

**ขั้นที่ 4: ตั้งชื่อและที่เก็บ**

8. Virtual machine name: `Mikrotik-CHR`
9. Location: ระบุโฟลเดอร์ที่ต้องการ → **Next**

**ขั้นที่ 5: ตั้งค่า CPU และ RAM**

10. Number of processors: `1`, Cores per processor: `1` → **Next**
11. Memory: `256` MB → **Next**

**ขั้นที่ 6: ตั้งค่า Network**

12. Network type: เลือก **Use bridged networking** → **Next**  
    *(จะเพิ่ม NIC เพิ่มเติมภายหลัง)*

**ขั้นที่ 7: ตั้งค่า Disk Controller และ Disk**

13. I/O Controller: เลือก **LSI Logic** → **Next**
14. Disk type: เลือก **SCSI** → **Next**
15. Select a Disk: เลือก **Use an existing virtual disk** → **Next**
16. คลิก **Browse** → ชี้ไปที่ไฟล์ `chr-7.21.4.vmdk` ที่แตกไว้ → **Next**

**ขั้นที่ 8: เสร็จสิ้น**

17. คลิก **Finish**

### เพิ่ม Network Adapter

หลังสร้าง VM เสร็จ ให้เพิ่ม NIC สำหรับเชื่อมต่อ Client:

1. คลิกขวาที่ VM `Mikrotik-CHR` → **Settings**
2. คลิก **Add** → เลือก **Network Adapter** → **Finish**
3. ตั้งค่า NIC ที่ 1: **Bridged** (เชื่อมต่ออินเทอร์เน็ต)
4. ตั้งค่า NIC ที่ 2: **Host-only** หรือ **Custom: VMnet2** (สำหรับ LAN/Hotspot)

---

## 3. เริ่มต้นระบบ Mikrotik OS และตั้งค่าเบื้องต้น

### เริ่มต้น VM

1. คลิก **Power on this virtual machine** เพื่อเปิด VM
2. รอระบบ Boot ประมาณ 10-30 วินาที

### เข้าสู่ระบบ

เมื่อหน้าจอแสดง Login prompt ให้ใส่:

```
Login:    admin
Password: (ปล่อยว่าง กด Enter)
```

ระบบอาจถามว่าต้องการดู Software License หรือไม่ ให้กด `Q` เพื่อข้ามไป

### ตั้งค่า Password ใหม่

```
[admin@MikroTik] > /user set admin password=YourPassword123
```

> **คำแนะนำ:** ตั้งรหัสผ่านที่แข็งแกร่งในการใช้งานจริง

### ดูข้อมูล Interface

```
[admin@MikroTik] > /interface print
```

ผลลัพธ์จะแสดง Interface ทั้งหมด เช่น:

```
Flags: D - dynamic, X - disabled, R - running, S - slave
 #   NAME      TYPE    ACTUAL-MTU
 0 R ether1    ether         1500
 1 R ether2    ether         1500
```

### ตั้งค่า IP Address เบื้องต้น (ether1 สำหรับ WAN)

```
[admin@MikroTik] > /ip address add address=192.168.1.2/24 interface=ether1 network=192.168.1.0
[admin@MikroTik] > /ip route add gateway=192.168.1.1
[admin@MikroTik] > /ip dns set servers=8.8.8.8,8.8.4.4
```

### ทดสอบการเชื่อมต่ออินเทอร์เน็ต

```
[admin@MikroTik] > /ping 8.8.8.8 count=4
```

### ตั้งชื่อเครื่อง (Hostname)

```
[admin@MikroTik] > /system identity set name=Mikrotik-Hotspot
```

---

## 4. เปิด Client เครื่อง VM Windows 7 ใน VMware

### สร้าง VM Windows 7

**ขั้นที่ 1: สร้าง VM ใหม่**

1. ใน VMware Workstation คลิก **File** → **New Virtual Machine**
2. เลือก **Typical (recommended)** → **Next**
3. เลือก **Installer disc image file (iso)**: ระบุ ISO ของ Windows 7 → **Next**

**ขั้นที่ 2: ตั้งค่า Windows 7**

4. ใส่ข้อมูล Product Key (ถ้ามี), Full name: `User`, Password ตามต้องการ → **Next**
5. ตั้งชื่อ VM: `Windows7-Client`
6. กำหนด Disk size: `40 GB` → **Next** → **Finish**

### ตั้งค่า Hardware ของ Windows 7

ก่อนเปิด VM แก้ไข Settings:

1. RAM: `1024 MB` ขึ้นไป
2. Network Adapter: เลือก **Custom: VMnet2** (เดียวกับ ether2 ของ Mikrotik)

### เปิด VM Windows 7

1. คลิก **Power on this virtual machine**
2. ติดตั้ง Windows 7 ตามขั้นตอนปกติ
3. หลังติดตั้งเสร็จ ติดตั้ง **VMware Tools** เพื่อประสิทธิภาพที่ดีขึ้น:
   - เมนู VMware → **VM** → **Install VMware Tools**
   - รันไฟล์ `setup.exe` ภายใน VM

---

## 5. ตั้งค่า Network ระหว่าง Windows 7 กับ VM Mikrotik CHR

### แผนผัง Network

```
[อินเทอร์เน็ต]
      |
   [ether1 - Bridged]
   [Mikrotik CHR]
   [ether2 - VMnet2]
      |
[Windows 7 Client]
  (VMnet2)
```

### ตั้งค่า VMware Virtual Network

1. เปิด **VMware Workstation** → **Edit** → **Virtual Network Editor**
2. เลือก **VMnet2** → ตั้งค่าเป็น **Host-only**
3. Subnet IP: `192.168.100.0`
4. Subnet mask: `255.255.255.0`
5. **ปิด** DHCP ของ VMnet2 (เพราะ Mikrotik จะทำหน้าที่ DHCP แทน)
6. คลิก **Apply** → **OK**

### ตั้งค่า Network Adapter ของ VM แต่ละเครื่อง

**Mikrotik CHR:**

| NIC | VMware Network | Mikrotik Interface | บทบาท |
|-----|---------------|-------------------|-------|
| NIC 1 | Bridged | ether1 | WAN (อินเทอร์เน็ต) |
| NIC 2 | VMnet2 | ether2 | LAN (Hotspot) |

**Windows 7 Client:**

| NIC | VMware Network | บทบาท |
|-----|---------------|-------|
| NIC 1 | VMnet2 | LAN (เชื่อมกับ Mikrotik) |

### ตั้งค่า IP ของ Windows 7 (Manual ชั่วคราว)

เพื่อทดสอบการเชื่อมต่อเบื้องต้น:

1. คลิกขวา **Network** → **Properties**
2. คลิก **Local Area Connection** → **Properties**
3. เลือก **Internet Protocol Version 4 (TCP/IPv4)** → **Properties**
4. เลือก **Use the following IP address:**

```
IP address:      192.168.100.10
Subnet mask:     255.255.255.0
Default gateway: 192.168.100.1
DNS:             8.8.8.8
```

5. คลิก **OK**

### ทดสอบ Ping จาก Windows 7 ไปยัง Mikrotik

เปิด Command Prompt บน Windows 7:

```cmd
ping 192.168.100.1
```

ถ้า Reply กลับมา แสดงว่าเชื่อมต่อสำเร็จ

---

## 6. การใช้โปรแกรม Winbox

### Winbox คืออะไร?

**Winbox** คือโปรแกรม GUI สำหรับจัดการ Mikrotik RouterOS บน Windows ใช้งานง่ายกว่าการพิมพ์คำสั่งผ่าน Terminal

### ดาวน์โหลด Winbox

ดาวน์โหลดจากเว็บ Mikrotik:

```
https://mikrotik.com/download
```

เลือก **Winbox** (เวอร์ชันล่าสุด) → ดาวน์โหลดไฟล์ `winbox64.exe` หรือ `winbox.exe`

> Winbox เป็น Portable App ไม่ต้องติดตั้ง สามารถรันได้ทันที

### เชื่อมต่อ Mikrotik ด้วย Winbox

1. รันไฟล์ `winbox64.exe` บน **Windows 7 (VM)**
2. ในช่อง **Connect To** ใส่:
   - IP Address: `192.168.100.1`  
   หรือใช้ MAC Address (คลิกปุ่ม `...` เพื่อ Scan หา Mikrotik ในวง LAN)
3. Login: `admin`
4. Password: รหัสผ่านที่ตั้งไว้
5. คลิก **Connect**

### หน้าต่างหลักของ Winbox

หลังเชื่อมต่อสำเร็จ จะเห็นเมนูหลักด้านซ้าย:

| เมนู | ฟังก์ชัน |
|------|---------|
| **Interfaces** | จัดการ Network Interface |
| **Bridge** | สร้างและจัดการ Bridge |
| **IP** | ตั้งค่า IP, DHCP, DNS, Firewall |
| **Hotspot** | ตั้งค่า Hotspot (อยู่ใน IP) |
| **System** | ตั้งค่าระบบ, Reboot |
| **Tools** | Ping, Traceroute, สำรองข้อมูล |

### ใช้ Terminal ใน Winbox

คลิก **New Terminal** เพื่อเปิด Terminal พิมพ์คำสั่งได้ เหมือนกับการ Console โดยตรง

---

## 7. Mikrotik CHR ตั้งค่า Bridge Network

### ทำไมต้องใช้ Bridge?

Bridge ช่วยให้ Interface หลายตัวทำงานเป็น Switch Layer 2 เดียวกัน เหมาะสำหรับการสร้าง Hotspot ที่ต้องการรวม Interface LAN หลายพอร์ตเข้าด้วยกัน

### สร้าง Bridge ผ่าน Winbox

**วิธีที่ 1: ผ่าน GUI Winbox**

1. เปิด Winbox → คลิกเมนู **Bridge**
2. คลิกปุ่ม **+** เพื่อสร้าง Bridge ใหม่
3. ตั้งชื่อ: `bridge-hotspot`
4. คลิก **OK**

**เพิ่ม Interface เข้า Bridge:**

5. คลิก Tab **Ports** → คลิก **+**
6. Interface: เลือก `ether2` (NIC ที่เชื่อมกับ Client)
7. Bridge: เลือก `bridge-hotspot`
8. คลิก **OK**

**วิธีที่ 2: ผ่าน Terminal**

```
/interface bridge add name=bridge-hotspot
/interface bridge port add interface=ether2 bridge=bridge-hotspot
```

### กำหนด IP ให้ Bridge

```
/ip address add address=192.168.100.1/24 interface=bridge-hotspot network=192.168.100.0
```

> หากมี IP เดิมบน ether2 ให้ลบออกก่อน:
> ```
> /ip address remove [find interface=ether2]
> ```

### ตั้งค่า Masquerade (NAT)

เพื่อให้ Client ใช้งานอินเทอร์เน็ตได้:

```
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade
```

### ตรวจสอบ Bridge

```
/interface bridge print
/interface bridge port print
/ip address print
```

---

## 8. การตั้งค่า Hotspot ของ VM Mikrotik CHR

### Hotspot คืออะไร?

**Hotspot** ใน Mikrotik คือระบบ Captive Portal ที่บังคับให้ผู้ใช้ Login ก่อนใช้งานอินเทอร์เน็ต พร้อมระบบจัดการ User, การจำกัด Bandwidth และ Session

### ขั้นตอนตั้งค่า Hotspot ด้วย Wizard

**ผ่าน Winbox:**

1. คลิกเมนู **IP** → **Hotspot**
2. คลิก **Hotspot Setup** (ปุ่มด้านบน)

**ขั้นที่ 1: เลือก Interface**

```
Hotspot Interface: bridge-hotspot
```

คลิก **Next**

**ขั้นที่ 2: IP ของ Hotspot**

```
Local Address of Network: 192.168.100.1/24
Masquerade Network: ✓ (ติ๊กถูก)
```

คลิก **Next**

**ขั้นที่ 3: Address Pool (DHCP)**

```
Address Pool of Network: 192.168.100.2 - 192.168.100.254
```

คลิก **Next**

**ขั้นที่ 4: SSL Certificate**

```
Select Certificate: none (ใช้ HTTP ธรรมดา)
```

คลิก **Next**

**ขั้นที่ 5: SMTP Server**

```
IP Address of SMTP Server: 0.0.0.0 (ปล่อยว่าง)
```

คลิก **Next**

**ขั้นที่ 6: DNS**

```
DNS Servers: 8.8.8.8, 8.8.4.4
DNS Name: hotspot.local (ชื่อ DNS ของ Hotspot)
```

คลิก **Next**

**ขั้นที่ 7: สร้าง User ตั้งต้น**

```
Name of Local Hotspot User: admin
Password: admin123
```

คลิก **Next** → **OK**

### ตรวจสอบการตั้งค่า Hotspot

```
/ip hotspot print
/ip hotspot user print
/ip hotspot profile print
```

### ดู Active Users

```
/ip hotspot active print
```

### ตั้งค่า DHCP Server (ถ้า Wizard ไม่สร้างให้)

```
/ip pool add name=hotspot-pool ranges=192.168.100.2-192.168.100.254
/ip dhcp-server add name=hotspot-dhcp interface=bridge-hotspot address-pool=hotspot-pool disabled=no
/ip dhcp-server network add address=192.168.100.0/24 gateway=192.168.100.1 dns-server=8.8.8.8
```

---

## 9. VM Windows 7 กับการเข้าใช้งานบริการ Hotspot VM Mikrotik CHR

### เปลี่ยน Windows 7 ให้รับ IP อัตโนมัติ

1. คลิกขวา **Network** → **Properties**
2. คลิก **Local Area Connection** → **Properties**
3. เลือก **Internet Protocol Version 4 (TCP/IPv4)** → **Properties**
4. เลือก **Obtain an IP address automatically**
5. เลือก **Obtain DNS server address automatically**
6. คลิก **OK**

### ตรวจสอบ IP ที่ได้รับ

เปิด Command Prompt:

```cmd
ipconfig /all
```

ควรได้รับ IP ในช่วง `192.168.100.2 - 192.168.100.254`

### ทดสอบ Ping

```cmd
ping 192.168.100.1
ping 8.8.8.8
```

### เข้าใช้งาน Hotspot Login Page

1. เปิด **Internet Explorer** หรือ Browser อื่นบน Windows 7
2. พิมพ์ URL ใดก็ได้ เช่น `http://www.google.com`
3. ระบบจะ Redirect ไปยังหน้า Login ของ Hotspot:

```
http://hotspot.local/login
หรือ
http://192.168.100.1/login
```

4. ใส่ข้อมูล Login:
   - Username: `admin`
   - Password: `admin123`
5. คลิก **Log In**

### หน้า Login Page สำเร็จ

หลัง Login สำเร็จ ระบบจะ Redirect ไปยังเว็บที่ต้องการ และสามารถใช้งานอินเทอร์เน็ตได้

### ตรวจสอบสถานะ Active Session

บน Winbox → **IP** → **Hotspot** → Tab **Active**

จะเห็นรายชื่อ User ที่กำลังใช้งาน พร้อม IP Address และเวลาที่ใช้งาน

### Logout

ผู้ใช้สามารถ Logout ได้โดย:
- เปิด Browser พิมพ์: `http://hotspot.local/logout`
- หรือรอ Session หมดอายุ

---

## 10. การปรับแต่งระบบของ Hotspot ของ Mikrotik

### การจัดการ User Profiles

**สร้าง Profile สำหรับจำกัด Bandwidth:**

ผ่าน Winbox → **IP** → **Hotspot** → Tab **User Profiles** → คลิก **+**

```
Name:           student
Rate Limit (rx/tx): 1M/1M
Session Timeout:    08:00:00
Idle Timeout:       00:30:00
Shared Users:       1
```

หรือผ่าน Terminal:

```
/ip hotspot user profile add name=student rate-limit=1M/1M session-timeout=8h idle-timeout=30m shared-users=1
/ip hotspot user profile add name=teacher rate-limit=5M/5M session-timeout=8h idle-timeout=1h shared-users=2
/ip hotspot user profile add name=guest rate-limit=512k/512k session-timeout=2h idle-timeout=15m
```

### การสร้าง User

```
/ip hotspot user add name=student01 password=pass01 profile=student
/ip hotspot user add name=teacher01 password=pass02 profile=teacher
/ip hotspot user add name=guest01 password=pass03 profile=guest
```

### การปรับแต่งหน้า Login Page

ไฟล์หน้า Login อยู่ใน Flash ของ Mikrotik สามารถดาวน์โหลดและแก้ไขได้:

**ดาวน์โหลดไฟล์ผ่าน FTP:**

1. เปิด FTP Client เช่น **FileZilla** บน Windows 7
2. เชื่อมต่อไปที่: `192.168.100.1` port `21`
3. Login ด้วย `admin` และรหัสผ่าน
4. ไปที่โฟลเดอร์ `/hotspot/`
5. ดาวน์โหลดไฟล์ `login.html`

**ไฟล์สำคัญใน Hotspot:**

| ไฟล์ | หน้าที่ |
|------|---------|
| `login.html` | หน้า Login |
| `logout.html` | หน้า Logout |
| `status.html` | หน้าแสดงสถานะการใช้งาน |
| `alogin.html` | หน้า Login สำหรับ Auto-login |
| `errors.html` | หน้าแสดง Error |

**ตัวแปรที่ใช้ใน Template:**

```html
$(username)    - ชื่อผู้ใช้
$(password)    - รหัสผ่าน
$(error)       - ข้อความ Error
$(link-login)  - URL สำหรับ Login
$(link-logout) - URL สำหรับ Logout
```

### ตั้งค่า Walled Garden (เว็บที่เข้าได้โดยไม่ต้อง Login)

```
/ip hotspot walled-garden add dst-host=*.google.com action=allow
/ip hotspot walled-garden add dst-host=*.microsoft.com action=allow
/ip hotspot walled-garden ip add dst-address=203.0.113.0/24 action=allow
```

### ตั้งค่า Advertisement (โฆษณา)

```
/ip hotspot user profile set [find name=guest] advertise=yes advertise-url=http://192.168.100.1/ads.html
```

### กำหนด Session Limit ด้วย Time-based Policy

```
/ip hotspot user set [find name=guest01] limit-uptime=2h
```

### ดู Log การ Login/Logout

```
/log print where topics~"hotspot"
```

### Backup ค่าตั้งค่า

```
/export file=hotspot-config
/system backup save name=hotspot-backup
```

---

## 11. การใช้งาน Radius Server เป็น Linux Ubuntu 22.04

### Radius คืออะไร?

**RADIUS (Remote Authentication Dial-In User Service)** คือโปรโตคอลสำหรับจัดการ Authentication แบบ Centralized ช่วยให้ Mikrotik ส่งข้อมูล Login ไปยัง Server กลาง แทนที่จะเก็บ User ไว้ในตัว Mikrotik

### สถาปัตยกรรมระบบ

```
[Windows 7 Client]
       |
[Mikrotik CHR Hotspot]
       |  (RADIUS Request: port 1812)
[Ubuntu 22.04 - FreeRADIUS Server]
       |
[MySQL Database - เก็บข้อมูล User]
```

### ติดตั้ง Ubuntu 22.04 VM

1. สร้าง VM ใน VMware สำหรับ Ubuntu Server 22.04
2. RAM: `1024 MB`, Disk: `20 GB`
3. Network: **VMnet2** (วงเดียวกับ Mikrotik LAN)
4. ติดตั้ง Ubuntu 22.04 LTS Server
5. ตั้ง IP แบบ Static: `192.168.100.200/24`, Gateway: `192.168.100.1`

### ติดตั้ง FreeRADIUS และ MySQL บน Ubuntu

**อัปเดตระบบ:**

```bash
sudo apt update && sudo apt upgrade -y
```

**ติดตั้ง FreeRADIUS และ MySQL:**

```bash
sudo apt install freeradius freeradius-mysql mysql-server -y
```

**ตรวจสอบสถานะ FreeRADIUS:**

```bash
sudo systemctl status freeradius
```

### ตั้งค่า MySQL

**เข้า MySQL:**

```bash
sudo mysql -u root -p
```

**สร้าง Database และ User:**

```sql
CREATE DATABASE radius;
CREATE USER 'radius'@'localhost' IDENTIFIED BY 'RadiusPass123';
GRANT ALL PRIVILEGES ON radius.* TO 'radius'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

**นำเข้า Schema ของ FreeRADIUS:**

```bash
sudo mysql -u root -p radius < /etc/freeradius/3.0/mods-config/sql/main/mysql/schema.sql
```

### ตั้งค่า FreeRADIUS ให้ใช้ MySQL

**เปิดใช้งาน SQL Module:**

```bash
sudo ln -s /etc/freeradius/3.0/mods-available/sql /etc/freeradius/3.0/mods-enabled/
```

**แก้ไขไฟล์ `/etc/freeradius/3.0/mods-enabled/sql`:**

```bash
sudo nano /etc/freeradius/3.0/mods-enabled/sql
```

แก้ไขค่าต่อไปนี้:

```
dialect = "mysql"
driver = "rlm_sql_mysql"

server = "localhost"
port = 3306
login = "radius"
password = "RadiusPass123"

radius_db = "radius"
```

**แก้ไขไฟล์ `/etc/freeradius/3.0/sites-enabled/default`:**

หา section `authorize` แล้วเพิ่มหรือ Uncomment:

```
authorize {
    ...
    sql
    ...
}
```

หา section `accounting` แล้วเพิ่ม:

```
accounting {
    ...
    sql
    ...
}
```

### เพิ่ม NAS Client (Mikrotik) ใน FreeRADIUS

แก้ไขไฟล์ `/etc/freeradius/3.0/clients.conf`:

```bash
sudo nano /etc/freeradius/3.0/clients.conf
```

เพิ่ม Block ต่อไปนี้:

```
client mikrotik-hotspot {
    ipaddr          = 192.168.100.1
    secret          = radiussecret123
    shortname       = mikrotik
    nastype         = other
}
```

### เพิ่ม User ใน Database

```bash
sudo mysql -u radius -p radius
```

```sql
INSERT INTO radcheck (username, attribute, op, value)
VALUES ('user01', 'Cleartext-Password', ':=', 'password01');

INSERT INTO radcheck (username, attribute, op, value)
VALUES ('user02', 'Cleartext-Password', ':=', 'password02');

EXIT;
```

**ตั้งค่า Reply Attributes (Bandwidth Limit):**

```sql
INSERT INTO radreply (username, attribute, op, value)
VALUES ('user01', 'Mikrotik-Rate-Limit', '=', '2M/2M');
```

### Restart และทดสอบ FreeRADIUS

```bash
sudo systemctl restart freeradius
sudo systemctl enable freeradius
```

**ทดสอบด้วย radtest:**

```bash
radtest user01 password01 localhost 0 testing123
```

ถ้าสำเร็จจะเห็น:

```
Received Access-Accept Id 0 from 127.0.0.1:1812 to 0.0.0.0:0 length 20
```

### ตั้งค่า Mikrotik ให้ใช้ RADIUS

**ผ่าน Terminal ของ Mikrotik:**

**เพิ่ม RADIUS Server:**

```
/radius add service=hotspot address=192.168.100.200 secret=radiussecret123 authentication-port=1812 accounting-port=1813
```

**ตั้งค่า Hotspot ให้ใช้ RADIUS:**

```
/ip hotspot profile set [find name=default] use-radius=yes
```

**ตั้งค่า RADIUS Accounting:**

```
/radius incoming set accept=yes port=3799
```

### ตรวจสอบการเชื่อมต่อ RADIUS

บน Mikrotik:

```
/radius print
/log print where topics~"radius"
```

บน Ubuntu ดู Log:

```bash
sudo tail -f /var/log/freeradius/radius.log
```

### ติดตั้ง daloRADIUS (Web Interface)

**ติดตั้ง Apache, PHP และ daloRADIUS:**

```bash
sudo apt install apache2 php php-mysql php-curl php-gd php-mbstring unzip -y
```

```bash
cd /tmp
wget https://github.com/lirantal/daloradius/archive/master.zip
unzip master.zip
sudo mv daloradius-master /var/www/html/daloradius
```

**ตั้งค่า daloRADIUS:**

```bash
cd /var/www/html/daloradius/app/common/includes/
sudo cp daloradius.conf.php.sample daloradius.conf.php
sudo nano daloradius.conf.php
```

แก้ไขค่า:

```php
$configValues['CONFIG_DB_HOST'] = 'localhost';
$configValues['CONFIG_DB_USER'] = 'radius';
$configValues['CONFIG_DB_PASS'] = 'RadiusPass123';
$configValues['CONFIG_DB_NAME'] = 'radius';
```

**นำเข้า Tables ของ daloRADIUS:**

```bash
sudo mysql -u radius -p radius < /var/www/html/daloradius/contrib/db/fr2-mysql-daloradius-and-freeradius.sql
```

**ตั้งสิทธิ์:**

```bash
sudo chown -R www-data:www-data /var/www/html/daloradius
sudo chmod -R 755 /var/www/html/daloradius
sudo systemctl restart apache2
```

**เข้าถึง daloRADIUS:**

เปิด Browser บน Windows 7:

```
http://192.168.100.200/daloradius/app/users/
```

Login เริ่มต้น:
- Username: `administrator`
- Password: `radius`

### สรุปการทดสอบระบบทั้งหมด

1. Windows 7 เปิด Browser → ระบบ Redirect ไปหน้า Login Hotspot
2. กรอก Username/Password → Mikrotik ส่ง RADIUS Request ไปยัง Ubuntu
3. FreeRADIUS ตรวจสอบข้อมูลใน MySQL → ส่ง Access-Accept กลับ
4. Mikrotik อนุญาตให้ผู้ใช้เข้าอินเทอร์เน็ต
5. ดู Log และสถิติผ่าน daloRADIUS Web Interface

---

## ภาคผนวก: คำสั่ง Mikrotik ที่ใช้บ่อย

```bash
# ดู Interface ทั้งหมด
/interface print

# ดู IP Address
/ip address print

# ดู Routing Table
/ip route print

# ดู Active Hotspot Users
/ip hotspot active print

# ดู Log
/log print

# Reboot
/system reboot

# Export Config
/export file=backup

# ดู System Resources
/system resource print
```

---

## ภาคผนวก: การแก้ปัญหาที่พบบ่อย

| ปัญหา | สาเหตุ | วิธีแก้ |
|-------|--------|---------|
| Client ไม่ได้รับ IP | DHCP ไม่ทำงาน | ตรวจสอบ `/ip dhcp-server print` |
| ไม่เห็นหน้า Login | DNS ไม่ถูกต้อง | ตั้ง DNS เป็น `8.8.8.8` บน Client |
| Login แล้วไม่มีอินเทอร์เน็ต | NAT ไม่ทำงาน | ตรวจสอบ Masquerade Rule |
| RADIUS ไม่ตอบสนอง | Firewall บน Ubuntu | `sudo ufw allow 1812,1813/udp` |
| Winbox เชื่อมต่อไม่ได้ | Firewall Mikrotik | ตรวจสอบ `/ip firewall filter print` |

---

*คู่มือนี้จัดทำเพื่อการศึกษาและทดสอบใน Lab Environment*  
*สำหรับการใช้งานจริง กรุณาปรึกษาผู้เชี่ยวชาญด้าน Network Security*
