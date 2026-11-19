# คู่มือแก้ปัญหาพื้นฐาน (Troubleshooting)

🏠 [หน้าหลัก](../README.md) · 📖 [คำศัพท์](glossary.md) · 🐙 [Git, GitHub และ Markdown](git-github-markdown.md)

> **หน้านี้รวมอะไร:** ปัญหาพื้นฐานเรื่อง Shell, สิทธิ์การรันสคริปต์, สาย USB / Serial Port, การตั้งค่า `PATH` และ Environment ที่พบบ่อยซ้ำกันทั้งใน **ND1**, **ND2** และ **ND3** เพื่อลดความซ้ำซ้อนและให้ค้นหาวิธีแก้ได้จากจุดเดียว

## 1. Shell และสิทธิ์การรันสคริปต์ (Windows / macOS / Linux)

| อาการ | สาเหตุ | วิธีแก้ |
|---|---|---|
| Windows: `not digitally signed` หรือ `.ps1` ถูกบล็อก | PowerShell ป้องกันการรันสคริปต์ | รัน `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` ทุกครั้งที่เปิดหน้าต่างใหม่ |
| Windows: เครื่องมือ ESP-IDF หรือ `robotos` error แปลก ๆ / path เพี้ยน | รันใน Git Bash (MSYS/MinGW แปลง path ที่ขึ้นต้นด้วย `/` ผิด) | เปลี่ยนไปใช้ **PowerShell 7** |
| macOS: Gatekeeper บล็อก binary เช่น `rbdev` | ระบบความปลอดภัยของ macOS บล็อกไฟล์ที่โหลดจากเน็ต | คลิกขวาที่ไฟล์ → กด **Open** หนึ่งครั้ง (หรืออนุญาตใน System Settings → Privacy & Security) |
| Linux / macOS: `permission denied` เวลารันสคริปต์ | ไฟล์ยังไม่มีสิทธิ์ execute | รัน `chmod +x <ชื่อไฟล์>` (เช่น `chmod +x run setup.sh`) |

## 2. USB, Serial Port และสิทธิ์การเชื่อมต่อ

| อาการ | สาเหตุ | วิธีแก้ |
|---|---|---|
| เสียบสายแล้วไม่เห็นพอร์ต (`COM` / `/dev/cu.*` / `/dev/tty*`) | ใช้สายชาร์จอย่างเดียว (ไม่มีสายข้อมูล) หรือพอร์ตหลวม | เปลี่ยนเป็นสาย USB ที่รับส่งข้อมูลได้ และลองสลับช่อง USB |
| Flash ไม่ได้ / เปิดพอร์ตไม่ขึ้น (`Resource busy` / Access denied) | มีโปรแกรมอื่นเปิดพอร์ตค้างอยู่ (serial monitor หรือแท็บ Web Serial) | ปิด serial monitor หรือแท็บเบราว์เซอร์ที่ต่อพอร์ตอยู่ทั้งหมด แล้วลองใหม่ |
| Linux: `permission denied` ที่ `/dev/ttyUSB0` หรือ `/dev/ttyACM0` | ผู้ใช้ยังไม่อยู่ในกลุ่ม `dialout` | รัน `sudo usermod -aG dialout $USER` แล้ว logout/login (หรือ reboot) หนึ่งครั้ง |
| เปิด serial port แล้วบอร์ด ESP32-S3 รีบูตเอง | วงจร auto-reset บนบอร์ดตอบสนองต่อสัญญาณ DTR/RTS | **เป็นพฤติกรรมปกติของบอร์ด** ไม่ใช่บั๊ก (รายละเอียดใน [ND1 บทที่ 2](../nd1/02-freertos-ai-agents.md#5-usb-บนบอร์ดนี้-และกับดัก-เปิดพอร์ตแล้วบอร์ดรีเซ็ต)) |

## 3. Environment, PATH และโฟลเดอร์โปรเจกต์

| อาการ | สาเหตุ | วิธีแก้ |
|---|---|---|
| `idf.py` หรือ `robotos` ขึ้น `command not found` | เปิด terminal ใหม่แล้วยังไม่ได้ activate environment | **ESP-IDF:** รัน `export.ps1` / `export.sh` ที่ `IDF_PATH` · **robotos:** ตั้ง `PATH` และ activate `.venv` ใหม่ทุกหน้าต่าง |
| `.venv does not exist` | ยังไม่ได้รันสคริปต์ติดตั้ง Python environment ครั้งแรก | รันสคริปต์ setup ของ workshop นั้น ๆ (เช่น `start-here` หรือ `setup.sh`) |
| `vp install` / compile พังแบบไม่มีสาเหตุชัดเจน | โฟลเดอร์โปรเจกต์อยู่ในพื้นที่ที่ sync กับ OneDrive / Google Drive / Dropbox | ย้ายโฟลเดอร์งานออกมาไว้ใน path ธรรมดา เช่น `D:\dev\` หรือ `~/dev/` |
| เปิดโปรเจกต์ใน VS Code แล้วตั้งค่าเดิมใช้ไม่ได้ | ไฟล์ `.vscode/settings.json` มี path จากเครื่องอื่น | ลบหรือแก้ path ใน `.vscode/settings.json` ให้ตรงกับเครื่องตัวเอง |

## 4. คำสั่งตรวจสุขภาพเครื่องมือ (รันก่อนถาม AI)

เมื่อติดปัญหาติดตั้งไม่สมบูรณ์ ให้รันคำสั่งตรวจสุขภาพของเครื่องมือนั้น ๆ แล้วนำผลลัพธ์ไปตรวจสอบหรือถาม AI จะได้คำตอบที่แม่นยำขึ้น:

- **ND1 (ESP-IDF & Workshop):** `idf.py --version` · `rb doctor` · คำสั่ง `ESP-IDF: Doctor Command` ใน VS Code
- **ND2 (Vite+ & Web):** `node -v` · `vp doctor`
- **ND3 (robotos):** `robotos doctor`

---
🏠 [หน้าหลัก](../README.md) · [ND1](../nd1/README.md) · [ND2](../nd2/README.md) · [ND3](../nd3/README.md)
