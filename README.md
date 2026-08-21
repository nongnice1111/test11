วิธีนำโฟลเดอร์ `final-pack` ไปวางและรันตั้งแต่ Proxmox จนจบ Flow หน้างาน:

---

**Step 1: ดึงไฟล์ลงเครื่อง Proxmox (Host Server)**
หลังจากลง Proxmox จาก ISO เสร็จ ให้เปิด Shell ของ Proxmox (ผ่านหน้าเว็บพอร์ต 8006 หรือเสียบจอต่อตรง):

* **กรณีต่อเน็ตได้ (วิธีที่เร็วที่สุด):**
```bash
cd /root
git clone <URL_REPO_GITHUB_ของคุณ> final-pack
cd final-pack

```


* **กรณีไม่มีเน็ต (เสียบ Flash Drive):**
```bash
# เสียบ USB แล้วเช็คชื่อไดรฟ์ (เช่น /dev/sdb1)
lsblk
mkdir -p /mnt/usb && mount /dev/sdb1 /mnt/usb
cp -r /mnt/usb/final-pack /root/
cd /root/final-pack

```



---

**Step 2: รันเครื่องที่ 1 (Proxmox)**

1. เช็ค ID Template:
```bash
qm list

```


2. แก้ไขไฟล์กลาง `config.env`:
```bash
nano config.env
# แก้ TPL_ID ให้ตรงกับเลขที่เห็น (เช่น 9000) -> กด Ctrl+O -> Enter -> Ctrl+X

```


3. สั่งรันสคริปต์ Provisioning:
```bash
chmod +x 01-proxmox/*.sh
./01-proxmox/01_provision.sh

```


4. จดเลข IP ของ VM 101 และ 102 ที่แสดงบนหน้าจอ แล้วเปิดแก้ `config.env` ใส่ `IP_GITLAB` กับ `IP_DEPLOY` ให้เรียบร้อย

---

**Step 3: ส่งไฟล์ข้ามไปเครื่อง 2 (GitLab VM) & เครื่อง 3 (Deploy VM)**
ใช้คำสั่ง `scp` จาก Proxmox ยิงโฟลเดอร์ข้ามไปแต่ละ VM ได้ทันทีโดยไม่ต้องไปนั่งโหลดใหม่:

```bash
# ส่งไปเครื่อง GitLab (VM 101)
scp -r /root/final-pack root@<IP_GITLAB>:/root/

# ส่งไปเครื่อง Deploy Target (VM 102)
scp -r /root/final-pack deploy@<IP_DEPLOY>:/home/deploy/

```

---

**Step 4: รันเครื่องที่เหลือตามลำดับ**

* **บนเครื่อง 2 (GitLab VM):**
```bash
ssh root@<IP_GITLAB>
cd /root/final-pack
chmod +x 02-gitlab-server/*.sh
./02-gitlab-server/02_install_gitlab.sh

```


* **บนโน้ตบุ๊กตัวเอง (Windows Client):**
* เปิดโฟลเดอร์ `04-windows-client` แล้วดับเบิ้ลคลิกไฟล์ `.bat` เพื่อสร้าง SSH Key
* ก๊อปปี้ Public Key ไปวางบนหน้าเว็บ GitLab แล้วนำ Private Key ไปแปะใน GitLab CI/CD Variables


* **บนเครื่อง 3 (Deploy VM):**
* สั่งรันสคริปต์ใน `03-deploy-target` เพื่อเช็คความพร้อมของ Docker และ Port ลำดับสุดท้าย
