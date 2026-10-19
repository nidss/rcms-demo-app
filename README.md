# RCMS Demo App

Prototype สำหรับนำเสนอโครงการ **Registration and Capital Monitoring System (RCMS)**
ความร่วมมือระหว่างกระทรวงอุตสาหกรรมและการค้า (MOIC) กับธนาคารส่งเสริมการเกษตร (APB)

เป็นเว็บแอปแบบ single-page ที่เขียนด้วย vanilla JS ล้วน (ไม่มี framework) รองรับ
2 ภาษา ลาว / อังกฤษ สลับได้ทันที และมีข้อมูลจำลอง (mock data) ที่ผูกตัวเลขให้
สอดคล้องกัน เพื่อให้สาธิต flow การทำงานได้จริงตั้งแต่ต้นจนจบ

> ข้อมูลทั้งหมดเป็น mock data สำหรับสาธิตเท่านั้น ไม่ได้เชื่อมฐานข้อมูลจริง

---

## โครงสร้างโปรเจกต์

รันผ่าน Vite เพื่อให้ import npm package จริงได้ (เช่น Leaflet สำหรับแผนที่)
แต่ build ออกมาเป็น **ไฟล์ HTML เดียวจบ** ที่ self-contained

- `src/index.html` คือ **source** ที่แก้ไข (ใช้ ES module + `import`)
- `index.html` ที่ root คือ **ไฟล์ build** (สร้างจาก `npm run build`)
  GitHub Pages เสิร์ฟจาก root นี้ และเปิดตรงๆ ด้วย `file://` ก็ได้

```
npm install
npm run dev      # dev server ที่ http://localhost:5173 (เสิร์ฟ src/)
npm run build    # build เป็นไฟล์เดียวไปทับ index.html ที่ root
npm run preview  # เสิร์ฟตัว build ไว้เช็คก่อน deploy
```

> ⚠️ แก้โค้ดที่ `src/index.html` เท่านั้น แล้ว `npm run build` จะทับ `index.html`
> ที่ root อย่าแก้ `index.html` ที่ root ตรงๆ เพราะมันคือ output ที่ถูก generate
> ทับทุกครั้งที่ build

**เหตุผลที่ต้อง build เป็นไฟล์เดียว:** แอปใช้ ES module + Leaflet ที่ browser จะรัน
ได้เฉพาะผ่าน http(s) ปลั๊กอิน `vite-plugin-singlefile` จะ inline JS/CSS ทั้งหมด และ
ฝังรูป (โลโก้ / ภาพพื้นหลัง) เป็น data URI ทำให้เหลือไฟล์ HTML เดียวที่เปิดที่ไหนก็ได้

ต้องมีเน็ตตอน `npm install` ครั้งแรก ส่วนตอนเปิดใช้งานถ้าไม่มีเน็ต ฟอนต์จะ fallback
เป็นฟอนต์ระบบ และแผนที่ / โลโก้จาก URL ภายนอกจะไม่ขึ้น (โลโก้ login มี fallback
เป็นโลโก้ที่ฝังในไฟล์อยู่แล้ว)

---

## บทบาท (Roles)

หน้า login ใส่ **ชื่อผู้ใช้ + รหัสผ่านอะไรก็ได้ + OTP 6 หลักอะไรก็ได้** (เป็น demo)
หรือเข้าแล้วสลับบทบาทได้จากเมนู avatar มุมขวาบน แต่ละบทบาทมีเมนูและหน้า Dashboard
ของตัวเอง

| ชื่อผู้ใช้ (demo) | บทบาท | หน้าแรก | เมนูที่เห็น |
|---|---|---|---|
| `lao.agri` | ผู้ประกอบการ (Business owner) | My Registered Capital | ทุนของฉัน |
| `khamla.p` | เจ้าหน้าที่เคาน์เตอร์ APB (Maker) | Counter Workspace | Dashboard, Search Enterprises |
| `sivilay.a` | เจ้าหน้าที่ตรวจสอบ (Checker) | Verification Workspace | Dashboard, Search Enterprises, Valuation Queue, Finance, Reports |
| `head.moic` | หัวหน้าพะแนก (Approver + E-Sign) | Approval Workspace | Dashboard, Search Enterprises, Valuation Queue, Finance, Reports |
| `lao.valuation` | บริษัทประเมิน (In-network firm) | My Valuation Jobs | งานประเมินของฉัน |
| `mof.gov` | หน่วยงานภายนอก (External agency) | External Portal | พอร์ทัลภายนอก |
| `bounmy.m` | ผู้ดูแลระบบ (System administrator) | System Overview | Dashboard, Search Enterprises, Reports, System Settings |

---

## ฟีเจอร์หลักตามหน้าจอ

### หน้า Login
- โลโก้ MOIC ใหม่ (เฟือง + รวงข้าว) บนวงกลมสีขาว ฝังอยู่ในไฟล์ (`src/moic-logo.png`)
  ไม่ต้องโหลดจากเว็บ; จอเล็กซ่อนภาพฝั่งซ้ายแล้วแสดงโลโก้เหนือฟอร์มแทน
- ยืนยันตัวตนแบบ MFA (OTP), สลับภาษา ลาว / อังกฤษ, กู้รหัสผ่าน
- ชิป **Demo accounts** ใต้ปุ่ม Next กดแล้วกรอก username ให้ทันที (สะดวกตอนนำเสนอ)

### Dashboard แยกตามบทบาท
แต่ละบทบาทเห็น Dashboard ที่ออกแบบเฉพาะงานของตัวเอง

ทุก Dashboard เริ่มด้วย **การ์ดสรุป (Overview)** แบบเดียวกับหน้ารายละเอียดวิสาหกิจ:
แถวตัวเลขแบ่งช่องเท่ากัน แล้วตามด้วยแถบแจ้งเตือนที่มีปุ่มทำงานอยู่ข้างข้อความ

- **ผู้ประกอบการ (My Registered Capital)** บัญชี `lao.agri` ถือวิสาหกิจ 5 แห่ง (`OWNER_ENTS`:
  ELN-2025-1001, ELN-2024-1004, ELN-2025-1005, ELN-2025-1115, ELN-2025-1122 สถานะคละกัน) หน้าแรกเป็น
  ภาพรวม (แถบ % ทุนที่ชำระรวม, จำนวนวิสาหกิจ, ชำระครบ, ยังต้องชำระ, เกินกำหนด) + การ์ดค้นหา
  (Enterprise, Registration no., Status) + ตารางวิสาหกิจ กดแถวเพื่อเข้า **Capital Details** ของ
  วิสาหกิจนั้น (ลิงก์ย้อนกลับไปหน้ารายการ; แจ้งชำระ, ใบรับรอง, รายละเอียดรายการ กลับมาที่หน้านี้)
  ถ้าผู้ประกอบการมีวิสาหกิจเดียวจะเปิด Capital Details ทันที
- **Maker (Counter Workspace)** แถบ % ทุนที่ชำระแล้วทั้งพอร์ต + ตัวเลข 5 ช่อง (รอตรวจ,
  ถูกตีกลับ, ใกล้ครบกำหนด, เกินกำหนด, ชำระครบ x/30); แจ้งเตือนงานที่ถูกตีกลับ (ปุ่ม Fix)
  และวิสาหกิจที่ติดค่าปรับ; ต่อด้วยการ์ดค้นหา (Reference, Enterprise, Stage แบบ dropdown,
  Submit by + ปุ่ม Search / Clear) ของรายการที่ยื่นไปแล้ว และรายการใกล้ครบ/เกินกำหนด
- **Checker (Verification Workspace)** เลือกรายการจากเมนูย่อยใต้ Dashboard: Verification
  queue, Awaiting approval, Returned by me หน้าแสดงการ์ดค้นหาแบบเดียวกับหน้า Finance &
  Penalties (Reference, Enterprise, Channel แบบ dropdown, Submit by; Returned by me มีแค่
  Enterprise ค้นได้ทั้งชื่อลาว/อังกฤษ/เลขทะเบียน) กรอกแล้วกดปุ่ม Search หรือ Enter เพื่อค้นหา
  ปุ่ม Clear ล้างเงื่อนไข แล้วตามด้วยตาราง
  ที่มี pagination เลือกได้ 5/10/20/50 แถวต่อหน้า
- **Approver (Approval Workspace)** Overview รวม pipeline ไว้ในการ์ดเดียว: แถบอัตราการชำระ
  รวม + ตัวเลข 5 ช่องเรียงตามขั้นตอน (อยู่ขั้นตรวจสอบ, รออนุมัติ + มูลค่า, ลงนามวันนี้, ชำระครบ x/30,
  เกินกำหนด) และแจ้งเตือนงานถัดไป (ปุ่ม Open to sign) ด้านล่างเป็นรายการจากเมนูย่อยใต้ Dashboard:
  Awaiting approval (ค้นหา Reference, Enterprise, Channel, Verified by), Overdue / penalties
  (Enterprise, Status) และ Recently signed (E-Sign) (Enterprise, Date) แต่ละรายการมีปุ่ม
  Search / Clear และ pagination
- **System administrator (System Overview)** แยกจากงานติดตามทุน มาเน้นงานดูแล
  ระบบแทน ได้แก่
  - **System health** (การ์ดสรุป): ผู้ใช้ทั้งหมด / ออนไลน์ / ถูกล็อก, สถานะการเชื่อมต่อ,
    จำนวน API ที่ล้มเหลว + แจ้งเตือน endpoint ที่ช้า (ปุ่ม API Log) และบัญชีที่ถูกล็อก (ปุ่ม Users)
  - **Users & RBAC**: โดนัทผู้ใช้ตามหน่วยงาน / ตามสิทธิ์ + ตารางจำแนก
  - **Integrations & API**: สถานะ endpoint (eBRS, TaxRIS, FinLink, GovPay)
  - **System configuration**: ตารางค่าธรรมเนียม, Master data, Virtual GL
  - **Audit & security log**: บันทึกการเข้าใช้ / เปลี่ยนสิทธิ์ / เหตุการณ์ระบบ

ตัวเลขในการ์ดสรุปเปลี่ยนน้ำหนักตามค่า: ค่า 0 เป็นสีเทาจาง, ค่าที่ต้องจัดการแสดงเป็นสีสถานะ
(ทอง/แดง/เขียว) เพื่อให้เห็นทันทีว่าอะไรต้องทำ ทั้งหมดมาจาก helper เดียว `sumCard()` + `metric()`

### ทะเบียนวิสาหกิจ (Search Enterprises)
- การ์ดค้นหาแสดงทุกช่องตลอด: ชื่อวิสาหกิจ (ลาว / อังกฤษ), เลขทะเบียน, แขวง, ประเภทวิสาหกิจ,
  ISIC, ช่วงวันที่จดทะเบียน และช่วงวันที่อัปเดต กรอกแล้วกด **Search** หรือ Enter, **Clear**
  ล้างเงื่อนไข (รายการแขวง / ISIC ดึงจากข้อมูลจริง ค่าใน dropdown ไม่ขึ้นกับภาษา)
- ตารางแยกคอลัมน์ Enterprise กับ Registration no.
- สถานะเป็นชิปกดได้พร้อมจำนวน, Export to Excel / Print อยู่ที่หัวตาราง

### รายละเอียดวิสาหกิจ / Case detail
อ่านจากบนลงล่างตามลำดับที่ใช้งานจริง: ตัวเลขทุน → สิ่งที่ต้องทำ → ข้อมูลอ้างอิง → ประวัติ

- **Capital Details (summary card)** แทน action bar + การ์ดโดนัทเดิมที่แสดงตัวเลขซ้ำกัน:
  สถานะ, แถบ % ที่ชำระ, ตัวเลข 5 ช่อง (ทุนจดทะเบียน / ชำระแล้ว / คงเหลือ / ค่าปรับ /
  วันครบกำหนด + จำนวนวัน), ปุ่มหลักมุมขวาบน และแถบแจ้งเตือนท้ายการ์ดที่มีปุ่มแก้ไขอยู่ข้างข้อความ
  (เช่น ใบแจ้งหนี้ค้าง + Pay by QR, งานที่รออนุมัติอยู่)
- **Enterprise Registration Details** แบ่งเป็นกลุ่มมีหัวข้อ: Identity / Registration /
  Tax & finance / Location (+ แผนที่ Leaflet) ใช้รูปแบบ summary list คือ label ซ้ายกว้างคงที่
  แล้ว value อยู่ติดกัน (ไม่ดันไปชิดขอบขวา) เพื่อให้อ่านคู่ label/value ได้ในครั้งเดียว
- หน้าตรวจทาน (Task) และรายละเอียดธุรกรรม ใช้ layout `worksplit`: ข้อมูลอ้างอิงซ้าย,
  คำร้อง + เอกสาร + การตัดสินใจขวา (ติดหน้าจอเวลาเลื่อน); จอเล็กสลับให้ส่วนตัดสินใจขึ้นก่อน

### บันทึกการชำระทุน (Declare Capital Payment)
- ฟอร์มกรอกข้อมูลจัดเรียงอยู่แถวเดียวกับ Enterprise Registration Details
- รองรับ Single payment และแบบแบ่งชำระ 2 งวด (Instalment), แนบเอกสารหลักฐาน

### ประเมินทรัพย์สิน (Valuation)
- บริษัทประเมิน In-network เห็น My Valuation Jobs; หน้ารายละเอียดงานมีแถบสรุปงาน
  (วิสาหกิจ, ทรัพย์สิน, ประเภทเครือข่าย, มูลค่าที่แจ้ง) → การ์ด "Asset under valuation"
  (Collateral + Fields ที่ยื่นมา) ซ้าย → ฟอร์ม Submit result ขวา (เอกสารเดิมรวมอยู่ในฟอร์ม
  ไม่มีกล่องว่างแยก)
- Checker/Approver มี Asset Valuation Queue พร้อม KPI, การ์ดค้นหา (Reference, Enterprise / ทรัพย์สิน,
  Valuation firm, Network type, Status แบบ dropdown + Search / Clear) และ pagination

### ตรวจทาน → อนุมัติ → ใบรับรอง
- Reviewer ตรวจแล้วส่งต่อหรือตีกลับ, Approver อนุมัติ + ลงลายเซ็นดิจิทัล (ต้อง MFA
  ซ้ำ), ออกใบรับรองการฝากทุน (Print Preview + ฉบับจริง/สำเนาพร้อม watermark)

### Finance
- 3 แท็บภายในเมนูเดียว: **Fees / Penalties / E-Receipts**
- แต่ละแท็บมี search filter แยกตาม criteria ของตัวเอง (เช่น Enterprise,
  Reference, Payment status, Days overdue, Receipt no, ช่วงวันที่) กรอกแล้วกดปุ่ม
  Search หรือ Enter เพื่อกรอง ปุ่ม Clear ล้างเงื่อนไข (แบบเดียวกับหน้า Checker)
  แท็บเป็น segmented control และปุ่ม Export / Print อยู่แถวเดียวกับแท็บ

### Reports
- เลือกรายงานตามประเภท + ช่วงวันที่ แล้ว generate / export (ตัด chart ออก เหลือ
  ส่วนรายงาน)

### System Settings (ผู้ดูแลระบบ)
เมนูย่อยถูกจัดเป็น **แท็บภายในหน้า**: Fees / Master Data / Virtual GL / Users / API Log

- **Fees** 3 คอลัมน์ความสูงเท่ากัน: Fee by enterprise type, Fee by registered
  capital, Penalty rate per day
- **Master Data** grid 3 คอลัมน์ 2 แถว: (ISIC Code, Province, Entity type) /
  (Files, Valuation firm) ความสูงเท่ากันในแต่ละแถว
- **Virtual GL** ตารางลงบัญชีทุกการประกอบทุนที่อนุมัติแล้ว + search filter
  (Enterprise, Capital account, ช่วงวันที่, Channel, Submit by)
- **Users** จัดการผู้ใช้ / RBAC + search filter (User, Organisation, Role, Status)
- **API Log** บันทึกการเชื่อมต่อระบบภายนอก
- การ์ดค้นหาของ Virtual GL และ Users ใช้ปุ่ม **Search** / **Clear** แบบเดียวกับหน้าอื่น
  (กรองเมื่อกด Search หรือ Enter)

ทุกจุดที่มีการค้นหาใช้รูปแบบเดียวกัน: ช่องค้นหา + ปุ่ม Search (icon `search`) + ปุ่ม Clear
(icon `brush-cleaning`) ได้แก่ Checker workspace, Search Enterprises, Finance, System Settings,
External Portal และกล่อง "ตรวจสอบสถานะวิสาหกิจ" (QR verify) ที่หน้า login

---

## แนวทางดีไซน์ (UI conventions)

- **โทนสี** ดึงจากโลโก้ MOIC ใหม่: น้ำเงิน `#283579` (primary / sidebar), เขียว
  `#137A41` (สำเร็จ), ทอง `#E6B92C` (accent: เมนูที่เลือก, แถบการ์ดที่มีงาน); สีแดงใช้เป็น
  สีสถานะเท่านั้น (เกินกำหนด / error / badge จำนวน) ทั้งหมดเป็น CSS token ใน `:root`
- **ลำดับความสำคัญ** หน้าละ 1 action หลัก (ปุ่มสีน้ำเงินทึบ) ที่เหลือเป็นปุ่ม ghost;
  การ์ดที่มีงานรอใช้ class `hot` (แถบ gradient ด้านบน) แทนกรอบหมุน
- **Topbar** ปุ่มช่วยเหลือ / แจ้งเตือน / ผู้ใช้ เป็นปุ่มเดียวกันขนาดเท่ากัน,
  **Reset demo data** ย้ายไปอยู่ในเมนูผู้ใช้ (ยังใช้ `Alt + R` ได้), toast แสดงกลางล่าง
  ไม่บังปุ่ม และไม่ขึ้น toast ตอน login แล้ว
- **ไอคอนเมนู** ทุกบทบาทใช้ Lucide SVG (dashboard, search, clipboard, globe,
  banknote, chart, settings, wallet)
- **สถานะ (pill)** มีจุดสีนำหน้าทุกอัน อ่านสถานะได้แม้ไม่ดูสี
- **เอกสาร** ทุกรายการใช้ `docRow()` เดียวกัน: สูงเท่ากัน, ไอคอนเดียวกัน, 2 บรรทัด
  (ประเภทเอกสาร + ชื่อไฟล์), ปุ่ม Open ด้านขวา; ไฟล์ที่เพิ่งแนบเป็นไอคอนสีเขียว + ปุ่มลบ ✕
- **หัวข้อ** ไม่ใส่หัวข้อย่อยซ้ำกับชื่อการ์ด
- **กราฟโดนัท** วงหนา ปลาย segment มน เว้นช่องว่างบางๆ, legend เป็นจุดกลม +
  ชื่อ + ตัวเลข + เปอร์เซ็นต์, palette ตาม brand
  (`#283579 #5B8DEF #137A41 #E6B92C #C8202E #8B6FD6`)
- **ปุ่มใน popup** กรณี 1 ปุ่ม ชิดขวา, กรณี 2 ปุ่มวาง secondary (Close/Cancel)
  ซ้าย และ primary ขวา
- **Responsive** ≤1180px KPI/ตัวกรองเหลือ 2 คอลัมน์, ≤980px หน้า work page เป็นคอลัมน์เดียว,
  ≤900px sidebar กลายเป็น drawer (ปุ่ม ☰ บน topbar), ≤640px summary card จัดลำดับใหม่
  (ตัวเลข → แจ้งเตือน → ปุ่ม เต็มความกว้าง), summary list เป็น label บน / value ล่าง,
  ตารางเลื่อนแนวนอนภายในการ์ด
- **ตัวอักษร** หลีกเลี่ยงการใช้เครื่องหมาย em-dash ทั้งใน UI และเอกสาร ใช้ `-` แทน

---

## Business rule ที่ฝังไว้จริง (ลองกดให้ error ดูได้)

- งวดแรกต้องไม่ต่ำกว่า 30% ของทุนจดทะเบียน
- ยอดที่บันทึกต้องไม่เกินยอดคงเหลือ
- ต้องแนบเอกสารหลักฐานอย่างน้อย 1 ไฟล์
- ปุ่มออกใบรับรองกดไม่ได้จนกว่าทุนจะชำระครบ
- ผู้อนุมัติจะไม่เห็นงานจนกว่าจะผ่านการตรวจทานก่อน

---

## หมายเหตุ

- ข้อมูลทั้งหมดเป็น **mock data** ตัวเลขในระบบผูกกันให้สอดคล้อง
  (เช่น 1,284 = 742 + 311 + 133 + 98) เพื่อให้ตรวจสอบระหว่างนำเสนอได้
  ส่วนตัวเลขรายเดือนในกราฟและอัตราต่างๆ เป็นค่าสมมติสำหรับแสดงโครงสร้างรายงาน
- วิสาหกิจหลัก 6 แห่ง (ELN-2025-1001 ถึง 1006) ใช้สาธิต flow ของ Owner/Maker ส่วนอีก 24 แห่ง
  (ELN-2025-1101 ถึง 1124, `MOCK_ENTS` ใน `src/index.html`) มีไว้ให้รายการของ Checker/Approver
  ยาวพอเห็น pagination: Verification queue 15, Awaiting approval 11, Returned by me 13 รายการ
- งานประเมิน mock อีก 14 งาน (VAL-2026-0342 ถึง 0355, `MOCK_VAL`) บนวิสาหกิจ mock พร้อมข้อมูล
  หลักประกันของแต่ละงาน ทำให้ Asset Valuation Queue มี 18 งาน (2 หน้า) และ My Valuation Jobs
  ของ Lao Valuation มี 9 งาน (สถานะคละ: กำลังประเมิน / รอรับผล / รออนุมัติบริษัท / ยืนยันแล้ว)
- Pagination (`paginate()` ใน `src/index.html`, เลือกได้ 5/10/20/50 แถวต่อหน้า) ใช้กับทุกตาราง:
  รายการของ Checker / Approver / Maker (ทั้งรายการที่ยื่นไปแล้วและรายการใกล้ครบ/เกินกำหนด),
  หน้าค้นหาวิสาหกิจ, Asset Valuation Queue, งานของบริษัทประเมิน, External Portal, Finance ทั้ง
  3 แท็บ, System Settings ทุกแท็บ (การ์ดเล็กใน Fees / Master Data เริ่มที่ 5 แถว), ตารางใน
  Dashboard ของผู้ดูแลระบบ, ใบแจ้งหนี้ของผู้ประกอบการ และประวัติการดำเนินการของวิสาหกิจ
  เปลี่ยนเงื่อนไขค้นหาแล้วกลับไปหน้า 1 เสมอ
- ตารางที่มีการ์ดค้นหาทั้งหมดใช้ `listCard()` ตัวเดียวกัน (กรองจาก state ไม่ได้ซ่อนแถวใน DOM)
  ช่องค้นหารองรับข้อความ, dropdown, วันที่ และเงื่อนไขเฉพาะ (เช่นช่วงจำนวนวันเกินกำหนด)
- External Portal: ตารางแยกคอลัมน์ Enterprise กับ Registration no. ค้นหาด้วย Enterprise,
  Registration no. และ Province (dropdown)
- ช่อง dropdown ในการ์ดค้นหาจับคู่ค่าแบบตรงทั้งคำ (เช่นเลือก "Vientiane Province" ไม่ปน
  "Vientiane Capital") ส่วนช่องข้อความค้นหาจากบางส่วนของคำ
- ชื่อสถานะบนหน้าจอใช้ภาษาลาว/ไทยเป็นหลัก และมีรหัสระบบ
  (PENDING / PARTIAL / COMPLETED / OVERDUE) กำกับไว้เพื่อเทียบกับเอกสาร requirement
- โค้ดทั้งหมดอยู่ในไฟล์เดียว (`src/index.html`) แบ่งส่วนด้วยคอมเมนต์:
  dictionary 2 ภาษา, ข้อมูล mock, helper (โดนัท / KPI / ไอคอน), ฟังก์ชัน view
  ต่อหน้าจอ (`vDash`, `vSet`, `vFin` ...), และตัว `render()` / `bind()` ที่วาดใหม่ทุกครั้ง

---

## Deploy

push ขึ้น branch `main` แล้ว GitHub Pages (Deploy from a branch, main / root) จะเสิร์ฟ
`index.html` ที่ root ให้อัตโนมัติ
