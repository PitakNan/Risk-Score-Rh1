# Work Log — Risk Score Dashboard

## 2026-07-05 — เพิ่ม Sort คลิกหัวตาราง (ตาราง รพ. เงินบำรุงบวก/ลบ)

**ฟีเจอร์ใหม่ใน `renderBumrungStatus()`:** คลิกหัวตาราง (จังหวัด/โรงพยาบาล/ประเภท/เงินบำรุงสุทธิ/Cash Ratio/Risk) เพื่อเรียงลำดับได้เองทั้ง 2 ตาราง คลิกซ้ำสลับทิศทาง มาก↔น้อย มีลูกศร ▲▼ บอกคอลัมน์+ทิศทางที่ใช้อยู่ สถานะ sort ของ 2 ตารางแยกอิสระจากกัน (ตัวแปร `bumSort.pos` / `bumSort.neg`)

**ค่าเริ่มต้น (ก่อนคลิกอะไร):**
- ตาราง รพ. เงินบำรุงบวก (42 แห่ง): เรียงเงินบำรุงสุทธิ มาก→น้อย (เหมือนเดิม)
- ตาราง รพ. เงินบำรุงลบ (61 แห่ง): เรียง Risk มาก→น้อย (7→1) ก่อน แล้วตามด้วยเงินบำรุงสุทธิ น้อย→มาก เป็น tiebreak (**ตัดเงื่อนไข Cash Ratio ที่เคยเป็น tiebreak ชั้น 2 ออก** เพราะ Cash Ratio กลายเป็นคอลัมน์ sort เองได้แล้ว)

ทดสอบด้วย Playwright ยืนยันแล้วว่า default order + click-to-sort + toggle ทิศทาง ทำงานถูกต้องครบทุกกรณี ไม่มี JS error

**Deploy แล้ว:** commit `3959ac7` push ขึ้น https://github.com/PitakNan/Risk-Score-Rh1 เรียบร้อย (backup index เดิมก่อนแก้ไว้ที่ `index(2026-07-05).html` ในโฟลเดอร์ repo ตาม convention เดิม)

## 2026-07-05 — จัดระเบียบไฟล์ที่ไม่เกี่ยวข้อง/เลิกใช้แล้ว

ย้าย repo GitHub เก่า `D:\Github\Risk-Score` (ก่อนเปลี่ยนชื่อเป็น Risk-Score-Rh1), ไฟล์ backup เก่าในโฟลเดอร์นี้ (30 ไฟล์), ไฟล์ index วันที่เก่าใน deploy repo, สคริปต์ scratch/ ยุค Flask+SQLite, โฟลเดอร์ 9.Github ที่ซ้ำซ้อน, และโปรเจกต์ tps-dashboard ที่หลงมาอยู่ผิดที่ — ทั้งหมดย้ายไป `D:\OneDrive\Share Rh1-New\0 Claude Cowork\_Backup\` (ไม่ได้ลบทิ้ง ดูรายละเอียดที่ README.md ในนั้น) **GitHub repo เก่า `pitaknan/Risk-Score` บนเว็บยังไม่ได้ลบ**

## กระบวนการ Deploy ไปยัง GitHub Pages

**คำสั่ง trigger:** ผู้ใช้พูดว่า **"Deploy เลย"**

**ขั้นตอน:**
1. Rename `D:\Github\Risk-Score-Rh1\index.html` → `index(YYYY-MM-DD).html` (วันที่ปัจจุบัน เช่น `index(2026-06-10).html`)
2. Copy `D:\Dashboard AI\Risk Score\Risk_Score_Dashboard.html` → `D:\Github\Risk-Score-Rh1\index.html`
3. Commit + Push ไปยัง GitHub

**หมายเหตุ:** GitHub Pages ต้องใช้ชื่อ `index.html` เท่านั้นถึงจะเปิดได้โดยตรงจาก URL

---

## 2026-06-10 — นำเข้าข้อมูล เม.ย. 2569

**งาน:** นำเข้าข้อมูล Risk Score เดือนเมษายน 2569 เข้าระบบ Dashboard

**ไฟล์ที่ใช้:** `risk score เม.ย. 69.xlsx` (103 รพ.)

**สิ่งที่ทำ:**
1. นำเข้าข้อมูล 103 รายการเข้า `dashboard.db` (sort_key: 202604)
2. ตรวจสอบความถูกต้อง Excel vs DB — ตรงกัน 103/103 ไม่มี mismatch
3. อัพเดต `Risk_Score_Dashboard.html` ด้วย `fix_update.py` — rebuild periods list ครบ 55 งวด FY 2565–2569

---

## 2026-06-10 — เพิ่มหน้า "วิเคราะห์เชิงลึก"

**งาน:** สร้าง tab ใหม่ใน `Risk_Score_Dashboard.html`

**ฟีเจอร์:**
1. Tab navigation สลับหน้าหลัก / วิเคราะห์เชิงลึก
2. เลือกตัวแปร multi-select (11 ตัวแปร)
3. Sort ซ้อน Sort สูงสุด 4 ลำดับ พร้อมปุ่ม ▲▼ สลับลำดับ
4. ตาราง highlight สีเขียว/เหลือง/แดงตามเกณฑ์
5. กราฟแนวโน้มรายเดือน multi-select metric (สี = รพ., รูปแบบเส้น = metric)

**ปัญหาที่พบ:**
- Dashboard พังจาก `const HOSP_NAMES` inject ซ้ำ 2 ครั้ง → restore backup + apply ครั้งเดียว
- ชื่อ รพ. ไม่ตรงกันระหว่างงวด (เก่า: `ชื่อ,รพศ.` / ใหม่: `รพ. ชื่อ`) → normalize โดยใช้รหัสหน่วยบริการเป็น key อัพเดต DB 103 รพ. + HTML 5,653 จุด

**ค่าเริ่มต้นหน้าวิเคราะห์เชิงลึก:**
1. เงินบำรุงคงเหลือสุทธิ (น้อย→มาก)
2. EBITDA (น้อย→มาก)
3. NI (น้อย→มาก)
4. Risk Score (มาก→น้อย)

**สคริปต์สำคัญ:**
- `fix_update.py` — อัพเดตข้อมูลเดือนใหม่เข้า HTML (ใช้แทน update_dashboard.py)
- `build_canonical_names.py` — normalize ชื่อ รพ. ทั้ง DB และ HTML (รันหลังนำเข้าข้อมูลใหม่)

---

## 2026-06-11 — เพิ่ม 2 ฟีเจอร์ใหม่ + ยืนยัน Focus Table

**งาน:** แก้ไข `Risk_Score_Dashboard.html` ก่อน Deploy

### ฟีเจอร์ที่ 1: ตาราง Province Summary คลิกดูรายชื่อ รพ. ได้
- **ที่แก้:** `renderRiskSummary()` — cell ที่มีตัวเลข (`v > 0`) เพิ่ม `onclick`, `cursor:pointer`, `title`
- **เพิ่ม function:** `openRiskProvPanel(prov, riskScore)` — filter รพ. ของจังหวัด + risk level นั้นจากงวดล่าสุด แล้วเรียก `openPanel()` เดิม
- **ผลลัพธ์:** คลิกตัวเลขในตาราง 📊 สรุปสถานการณ์ Risk Score รายจังหวัด → เปิด panel รายชื่อ รพ. เหมือน 🎯 Risk tile

### ฟีเจอร์ที่ 2: บล็อกประวัติ Risk Score ใน บทวิเคราะห์ รพ.
- **ที่แก้:** `renderHospAnalysis()` — แทรกบล็อกใหม่ต่อจาก exec summary
- **ข้อมูล:** ดึงจาก `RAW.records` ทั้งหมดของ รพ. นั้น (ไม่ผ่าน filter) นับงวดที่เคยอยู่แต่ละ Risk 0–7
- **แสดง:** ช่วงข้อมูล (จากงวดแรก–ล่าสุด), จำนวนงวดรวม, highlight ระดับปัจจุบัน (◀)

### ยืนยัน: ตาราง Focus Page
- `focus-tbl-wrap` ยังคง `max-height:320px` + `overflow-y:auto` (แก้ไขเมื่อวานยังอยู่ครบ)
