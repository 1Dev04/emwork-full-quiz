# แบบทดสอบ Fullstack Developer (Specialized HR/Payroll)

```git version 2.45.2.windows.1```

## ข้อ 1: The Buggy Payroll Logic
URL: https://github.com/1Dev04/emwork-full-code.git

## ข้อ 2: The Midnight Shift SQL
#### Query:
```
SELECT
    e.emp_id,
    e.name,
    a.clock_in,
    TIMESTAMPDIFF(MINUTE, '2026-03-19 00:05:00', a.clock_in) AS late_minutes

FROM attendance a
JOIN employees e ON e.emp_id = a.emp_id
JOIN shifts    s ON s.emp_id = a.emp_id
               AND s.shift_date = '2026-03-19'
               AND s.shift_type = 'NIGHT'

WHERE
    -- รองรับสแกนข้ามคืน 23:45 วันที่ 18
    a.clock_in BETWEEN '2026-03-18 23:45:00' AND '2026-03-19 08:00:00'
    -- มาสายคือหลัง 00:05
    AND a.clock_in > '2026-03-19 00:05:00'

ORDER BY late_minutes DESC;
```

## ข้อ 3: System Design & Audit Trail
#### Query:
```
-- ตารางหลัก
CREATE TABLE salaries (
    emp_id      INT PRIMARY KEY,
    base_salary DECIMAL(15,2) NOT NULL,
    updated_at  DATETIME
);

-- Audit Log (ลบไม่ได้, Immutable)
CREATE TABLE salary_audit_logs (
    id           BIGINT PRIMARY KEY AUTO_INCREMENT,
    emp_id       INT NOT NULL,
    changed_by   INT NOT NULL,
    old_salary   DECIMAL(15,2) NOT NULL,
    new_salary   DECIMAL(15,2) NOT NULL,
    reason       TEXT NOT NULL,
    ip_address   VARCHAR(45),
    created_at   DATETIME DEFAULT CURRENT_TIMESTAMP
);
```
#### API
```
POST   /api/salary-adjustments          ยื่นขอแก้
PATCH  /api/salary-adjustments/{id}/approve   HR อนุมัติ
GET    /api/employees/{empId}/audit-log        ดู history
```
#### ป้องกัน IT แก้เงินเดือนตัวเอง
```
1. App Layer  → if (actor.emp_id === empId) throw 403
2. DB Layer   → REVOKE UPDATE ON salaries FROM 'it_user'
3. Procedure  → IF p_emp_id = p_changed_by → SIGNAL ERROR
4. Alert      → ทุก direct DB access แจ้ง Security ทันที
```

---
## ข้อ 4: ManufacturePlus Scenario
#### Functional Requirements 3 ข้อ:
```
- FR-01: Audit Log บันทึกทุกธุรกรรมวันลา — ใครทำ, เวลาไหน, เปลี่ยนอะไร
- FR-02: Point Tracking สะสม Point จากการมางาน, ร่วมกิจกรรม, กรอก Daily Report → เชื่อมเกณฑ์ขึ้นเงินเดือน
- FR-03: Leave Self-Service + Alert พนักงานแจ้งลาเอง → ระบบหักวันอัตโนมัติ → แจ้งเตือนเมื่อวันลาเหลือ ≤ 2 วัน
```
## ข้อ 5: Database Modeling for Shifts
#### ER Diagram:
```
(Flow ตาม Module ที่วางไว้)
EMPLOYEE (1) ──< SHIFT_SWAP_REQUEST >── (1) EMPLOYEE
                    │
                    ├──> AUDIT_LOG
                    │
                    ├──> SHIFT_SCHEDULE
                    │
                    └──> OT_CALCULATION ──> PAYROLL_RESULT
```
#### Tables
```
employees        -- employee_id (PK) | name | base_salary
shifts           -- shift_id (PK) | shift_name | start_time | end_time | is_night | allowance
shift_schedules  -- schedule_id (PK) | employee_id | shift_id | work_date | status
shift_swap_requests -- request_id (PK) | requester_id | receiver_id | status(pending/approved/rejected) | approved_by
audit_logs       -- log_id (PK) | request_id | action | actor_id | old_status | new_status | timestamp
ot_calculations  -- ot_id (PK) | employee_id | request_id | is_night | night_allowance | ot_amount
payroll_results  -- payroll_id (PK) | employee_id | base_salary | night_allowance | ot_amount | total_pay
```
#### OT Logic:
```
swap approved → ดึง actual_shift หลัง swap → is_night = true → บวก night_allowance → วันหยุด → คำนวณ ot_amount → บันทึก payroll_results
```

### ส่วนที่ 3 — AI Integration & Supervision
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
