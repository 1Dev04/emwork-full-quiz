# แบบทดสอบ Fullstack Developer (Specialized HR/Payroll)

### AI Integration & Supervision
---
Prompt:
```
ออกแบบ flow diagram ระบบ AI ตอบคำถามสิทธิ์การลาพนักงาน
มี 5 ชั้น: INPUT → GUARD → LOGIC → POLICY → OUTPUT

กฎบริษัท: ลาพักร้อน 10 วัน/ปี, สะสมได้ไม่เกิน 15 วัน,
แจ้งล่วงหน้า 3 วัน

แสดง Guard Layer 3 ตัว (Intent/Data/Scope),
Logic คำนวณ deterministic + cross-check 2 วิธี,
Policy Validator 3 rules, reject path สีแดง,
safe output + audit log
```
Diagram: 
---
![Diagram1](https://res.cloudinary.com/dag73dhpl/image/upload/v1781755159/Screenshot_2026-06-18_104657_fbqt7m.png)
![Diagram2](https://res.cloudinary.com/dag73dhpl/image/upload/v1781755159/Screenshot_2026-06-18_104711_vnof96.png)
