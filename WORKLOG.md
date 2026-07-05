# Work Log — Risk Score Dashboard

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
