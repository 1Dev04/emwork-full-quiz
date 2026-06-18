# แบบทดสอบ Fullstack Developer (Specialized HR/Payroll)

### 3. AI Integration & Supervision
---
## ข้อ 6: Smart Policy Bot
#### Prompt:
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
#### Logic Pesudo Code: 
---
```
Start
# Guard
if not leave_keywords in input:        return "❌ ไม่ใช่คำถามเรื่องลา"
if missing required_fields:            return "⚠️ ขาดข้อมูล กรุณากรอกเพิ่ม"
if out_of_scope in input:              return "❌ ถาม HR โดยตรง"

# Calculate
accrued   = min(carried_over + 10, 15)
remaining = accrued - used_this_year
if remaining != method_b:             return "🚨 ผลไม่ตรง → ส่ง HR"

# Policy
if remaining > 15:                    return "❌ R1: สะสมเกิน 15 วัน"
if notice_days < 3:                   return "❌ R2: แจ้งล่วงหน้าไม่ถึง 3 วัน"
if used_this_year > 10:               return "❌ R3: เกิน 10 วัน/ปี"

# Output
return f"✅ วันลาคงเหลือ {remaining} วัน + ⚠️ ยืนยันกับ HR ก่อนยื่นจริง"
End
```
---
## ข้อ 7: AI Feature Design
#### Diagram:
---
![Diagram7.1](https://res.cloudinary.com/dag73dhpl/image/upload/v1781755959/Screenshot_2026-06-18_110544_rypmxf.png)
![Diagram7.2](https://res.cloudinary.com/dag73dhpl/image/upload/v1781756028/Screenshot_2026-06-18_110739_ujcawv.png)
