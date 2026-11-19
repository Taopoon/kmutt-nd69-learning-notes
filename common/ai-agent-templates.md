# ตัวอย่าง `AGENTS.md` และ Prompt Template สำหรับทำงานกับ AI Agent

🏠 [หน้าหลัก](../README.md) · 📖 [คำศัพท์](glossary.md) · 🐙 [Git, GitHub และ Markdown](git-github-markdown.md)

> **หน้านี้ใช้ทำอะไร:** รวมต้นแบบไฟล์ `AGENTS.md` และ Prompt Template ที่ระบุข้อกำหนดทางวิศวกรรมครบถ้วน (พลังงาน, หน่วยความจำ, ความปลอดภัย, สถาปัตยกรรม 4 ชั้น และหลักฐานการทดสอบ) เพื่อให้ AI coding agent ทำงานได้ตรงสเปกและลดการแก้โค้ดวนซ้ำ

---

## 1. ตัวอย่างไฟล์ `AGENTS.md` (วางที่ root ของโปรเจกต์)

ไฟล์ `AGENTS.md` ช่วยให้ AI agent (เช่น Claude Code, Codex, Gemini, Copilot, Cursor) รู้กติกาของโปรเจกต์ตั้งแต่เริ่มโดยไม่ต้องพิมพ์คำสั่งซ้ำทุกแชท

```markdown
# AGENTS.md — Project Guidelines

## Build & Verify Commands
- Build: `idf.py build` (หรือ `vp build` สำหรับเว็บ / `robotos doctor` สำหรับหุ่นยนต์)
- Flash & Monitor: `idf.py -p <PORT> flash monitor`
- Run Tests: `<ระบุคำสั่งรัน test หรือ simulator>`

## Architecture & Hard Constraints
1. **ห้ามเดาสเปกฮาร์ดแวร์:** ตรวจสอบ pinout, caps mask และ datasheet ในโฟลเดอร์ `docs/` ก่อนแก้ไขโค้ดไดรเวอร์ทุกครั้ง
2. **วินัยหน่วยความจำ (Embedded):**
   - ใช้ static allocation เป็นหลัก ห้ามใช้ `malloc`/`free` ใน ISR
   - DMA buffer ต้องจองด้วย `heap_caps_malloc(..., MALLOC_CAP_DMA)` เท่านั้น
   - ค่าที่ต้องเก็บถาวรข้ามการปิดเครื่องให้เก็บใน **NVS**
3. **ความปลอดภัยและเสถียรภาพ:**
   - การสื่อสารเครือข่ายต้องใช้ **TLS** ห้ามส่งรหัสผ่านหรือ API key แบบ plain text
   - ห้าม hardcode credential ลงในโค้ด
4. **รูปแบบการทำงานของ Agent:**
   - เสนอแผนแก้ไขสั้น ๆ ก่อนลงมือเมื่อต้องแก้มากกว่า 2 ไฟล์
   - หลังแก้โค้ด ต้องรันคำสั่ง build/test เพื่อยืนยันว่าผ่านจริงก่อนสรุปผล
```

---

## 2. Prompt Template — งาน Firmware & Edge AI (ND1)

ใช้เมื่อสั่ง AI เขียนหรือปรับแก้ firmware บน ESP32-S3 + FreeRTOS:

```text
บริบท: บอร์ด <es3c28p / ws28p> (ESP32-S3R8, Flash 16 MB, PSRAM 8 MB) ใช้ ESP-IDF v6.1 และ FreeRTOS

งานที่ต้องการ:
- <อธิบายฟีเจอร์ เช่น อ่านค่าจากเซ็นเซอร์ I2C แล้วส่งขึ้น MQTT broker>

ข้อกำหนดที่ต้องปฏิบัติตาม (ห้ามละเว้น):
1. Memory:
   - DMA buffer ต้องอยู่ใน internal SRAM (`MALLOC_CAP_DMA`)
   - Buffer ขนาดใหญ่ที่ไม่ใช่ DMA ให้วางใน PSRAM (`MALLOC_CAP_SPIRAM`)
   - ค่าตั้งค่าถาวร (เช่น WiFi, device ID) ต้องเก็บใน NVS
2. Concurrency & ISR:
   - แยกงานเป็น FreeRTOS tasks และสื่อสารผ่าน Queue
   - บัส I2C ที่ใช้ร่วมกันต้องป้องกันด้วย Mutex และตรวจค่า return ของ `xSemaphoreTake` ทุกครั้ง
   - ISR ต้องใส่ `IRAM_ATTR` และส่งงานต่อให้ task เท่านั้น (ห้าม `printf` หรือ `vTaskDelay`)
3. Power & Security:
   - ใช้การเชื่อมต่อแบบ TLS (พอร์ต 8883) และจัดการ reconnect เมื่อเครือข่ายหลุด
   - <ระบุเงื่อนไขประหยัดพลังงาน เช่น light sleep / wake on event>

การตรวจสอบ:
- ระบุคำสั่ง `idf.py build` และเงื่อนไขการทดสอบกรณีเครือข่ายหลุดหรือข้อมูลผิดพลาด
```

---

## 3. Prompt Template — งาน Web App คุยกับฮาร์ดแวร์ (ND2)

ใช้เมื่อสั่ง AI สร้างเว็บแอปด้วย TypeScript / React ผ่าน Web Serial หรือ Web Bluetooth:

```text
บริบท: เว็บแอป TypeScript (รันบน Vite+ / Chromium) คุยกับบอร์ดผ่าน <Web Serial / Web Bluetooth> ด้วยโปรโตคอล <JSON lines / protobuf + COBS>

งานที่ต้องการ:
- <อธิบายหน้าจอหรือฟีเจอร์ เช่น dashboard แสดงระดับเสียงแบบ real-time และปุ่มปรับสี LED>

ข้อกำหนดสถาปัตยกรรม 4 ชั้น (ต้องแยกไฟล์/โมดูลชัดเจน):
1. Transport: จัดการเฉพาะ byte stream ของ Web Serial / BLE
2. Connection Manager: รองรับ state `disconnected → connecting → connected` พร้อมจัดการกรณีสายหลุดและ auto-reconnect
3. Binding / Dispatch: แปลงข้อความเข้า-ออก และตรวจสอบ schema (discard ข้อความที่ parse ไม่ผ่าน)
4. UI / State:
   - แยกค่าที่ผู้ใช้สั่ง (requested) กับค่าที่บอร์ดยืนยันแล้ว (applied) ออกจากกันชัดเจน
   - ใช้ capability gating จาก `BoardInfo` (caps mask) เปิดเฉพาะฟีเจอร์ที่บอร์ดรองรับ

ข้อห้าม:
- ห้ามใช้ polling ถามค่าซ้ำทุกวินาที ให้ใช้ event/notification จากอุปกรณ์
```

---

## 4. Prompt Template — งาน Middleware & Digital Twin (ND3)

ใช้เมื่อสั่ง AI เขียน Python node บน ROS 2 หรือ `robotos`:

```text
บริบท: ระบบ <ROS 2 / robotos> ภาษา Python ใช้ topic `/robot/01/cmd_vel` (Twist), `/robot/01/odom` (Odometry) และ `/robot/01/scan` (LaserScan: index 0=ขวา, 1=หน้า, 2=ซ้าย)

งานที่ต้องการ:
- <อธิบายพฤติกรรม เช่น เขียน state machine CRUISE / AVOID / RECOVER เดินตามผนังขวา>

ข้อกำหนดการออกแบบ:
1. Communication:
   - เลือกใช้ Topic (stream), Service (ถาม-ตอบสั้น) หรือ Action (งานยาวที่ยกเลิกได้) ให้เหมาะกับหน้าที่
   - ส่ง `cmd_vel` ที่ความถี่ 20 Hz (ทุก 50 ms) และส่ง Twist เป็นศูนย์เมื่อจบการทำงาน
2. Control & Stability:
   - ออกแบบเป็น State Machine ที่ชัดเจน และใส่ hysteresis เพื่อป้องกันการสลับสถานะรัว ๆ ที่ขอบเกณฑ์
   - ตรวจ `range_max` และค่าผิดปกติจากเซ็นเซอร์ก่อนนำไปคำนวณ
3. Verification Evidence:
   - บันทึก log เฉพาะเมื่อมีการเปลี่ยนสถานะ พร้อมระยะ R/F/L
   - ระบุคำสั่งบันทึก `.rbag` (`robotos record`) และคำสั่งตรวจสอบผลลัพธ์ (`robotos bag info`, `topic echo`)
```

---
🏠 [หน้าหลัก](../README.md) · [ND1](../nd1/README.md) · [ND2](../nd2/README.md) · [ND3](../nd3/README.md)
