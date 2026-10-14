# SpotSync (SmartPark CUEE)

ระบบที่จอดรถอัจฉริยะสำหรับติดตามสถานะช่องจอดแบบเรียลไทม์, ผูกผู้ใช้ผ่าน LINE, แจ้งเตือนค่าจอดอัตโนมัติ และเชื่อมต่ออุปกรณ์ ESP32 สำหรับเซ็นเซอร์/ไม้กั้น

## ภาพรวมโปรเจกต์

SpotSync เป็นแอปเว็บด้วย **Next.js 14 (App Router)** ที่ทำงานร่วมกับ:
- **MongoDB** เก็บสถานะช่องจอดและประวัติการจอด
- **LINE Messaging API + LINE Login/LIFF** สำหรับยืนยันตัวตนและส่งแจ้งเตือน
- **ESP32 Devices** สำหรับอัปเดตสถานะเซ็นเซอร์และควบคุมไม้กั้น

## ความสามารถหลัก

- แสดงสถานะช่องจอดว่าง/ไม่ว่างแบบเรียลไทม์
- เลือกช่องจอดและยืนยันการเข้าจอดผ่านหน้าเว็บ
- ส่ง E-Ticket และแจ้งเตือนค่าจอดผ่าน LINE
- คิดค่าจอดตามช่วงเวลาอัตโนมัติจากเซิร์ฟเวอร์
- เปิดไม้กั้นอัตโนมัติช่วงสั้น ๆ เมื่อมีเหตุการณ์เข้า/ออก
- มีหน้า Admin สำหรับดูสถานะ, ผู้ใช้, history และจัดการช่องจอด
- รองรับการอัปเดตสถานะจากอุปกรณ์ (API Key/HMAC)

## โครงสร้างโดยย่อ

```text
app/
  api/
    update/            อัปเดตสถานะช่องจอด (web/sensor/checkout)
    get-status/        อ่านสถานะช่องจอดทั้งหมด
    gate/status/       API ให้ ESP32 ไม้กั้น poll สถานะเปิด/ปิด
    billing-tick/      คำนวณ/ส่งแจ้งเตือนค่าจอดตามรอบ
    confirm/           ผูกผู้ใช้กับ session และส่งตั๋วจอด
    webhook/           LINE bot webhook
    auth/*             LINE login + user login
    admin/*            API ฝั่งผู้ดูแล
  page.tsx             หน้าใช้งานหลักผู้จอดรถ
  checkin/             หน้า flow ตอนเข้าจอด
  checkout/            หน้า flow ตอนออก + QR ขาออก
  qrcode/              หน้าจอ QR ทางเข้าแบบหมุน token
lib/
  mongodb.ts           MongoDB connection
  parking-repository.ts ธุรกรรมหลักของช่องจอด/เซสชัน
  line-client.ts       LINE push/reply helper
  device-auth.ts       ตรวจ API key + HMAC สำหรับอุปกรณ์
```

## การทำงานระดับสูง

1. ผู้ใช้สแกน QR ทางเข้า และล็อกอิน/เชื่อม LINE
2. ผู้ใช้เลือกช่องว่าง แล้วระบบเรียก `/api/update` เป็น `occupied`
3. ระบบสร้าง session จอดรถ, บันทึกฐานข้อมูล และส่งตั๋วผ่าน LINE
4. ฝั่ง client เรียก `/api/billing-tick` เป็นช่วง ๆ เพื่อให้เซิร์ฟเวอร์ส่งแจ้งเตือนค่าจอดตามรอบ
5. เมื่อ checkout ระบบเรียก `/api/update` เป็น `vacant` พร้อม source `checkout`
6. ระบบปิด session, ส่งใบเสร็จ/แจ้งเตือน และเปิดไม้กั้นชั่วคราวให้ ESP32 ตรวจพบผ่าน `/api/gate/status`

## API สำคัญ

- `POST /api/update` เปลี่ยนสถานะช่องจอด + จัดการ session
- `GET /api/get-status` ดึงสถานะช่องจอดทั้งหมด
- `POST /api/confirm` ผูกผู้ใช้กับ session และส่งตั๋ว LINE
- `POST /api/billing-tick` คำนวณรอบค่าจอดและ trigger แจ้งเตือน
- `GET /api/gate/status` ให้ controller ไม้กั้นอ่านคำสั่งเปิด/ปิด
- `POST /api/webhook` รับข้อความจาก LINE OA

## เริ่มต้นใช้งาน

```bash
npm install
npm run dev
```

เปิดที่ `http://localhost:3000`

## Environment Variables ที่ใช้บ่อย

- `MONGODB_URI`
- `MONGODB_DATABASE`
- `SMARTPARK_SLOTS` (เช่น `A1,A2,A3,A4`)
- `SESSION_SECRET`
- `LINE_CHANNEL_ACCESS_TOKEN`
- `LINE_LOGIN_CLIENT_ID`
- `LINE_LOGIN_CLIENT_SECRET`
- `LINE_LIFF_ID`
- `LINE_OA_ID`
- `DEVICE_API_KEY`
- `DEVICE_HMAC_WINDOW_SECONDS`
- `ADMIN_USERNAME`
- `ADMIN_PASSWORD`

## เอกสารเสริม

- คู่มือฝั่งอุปกรณ์/ESP32: [`README-ESP32.md`](./README-ESP32.md)

