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

- **Maker (Counter Workspace)** แถบ % ทุนที่ชำระแล้วทั้งพอร์ต + ตัวเลข 5 ช่อง (รอตรวจ,
  ถูกตีกลับ, ใกล้ครบกำหนด, เกินกำหนด, ชำระครบ x/6); แจ้งเตือนงานที่ถูกตีกลับ (ปุ่ม Fix)
  และวิสาหกิจที่ติดค่าปรับ; ต่อด้วยรายการที่ยื่นไปแล้ว และรายการใกล้ครบ/เกินกำหนด
- **Checker (Verification Workspace)** ตัวเลข 4 ช่อง + แจ้งเตือน "งานถัดไป" พร้อมปุ่ม
  Start review เข้างานแรกทันที (ไม่มีงาน = แถบสีเขียว All caught up), คิวตรวจสอบ, งานตีกลับ
- **Approver (Approval Workspace)** รูปแบบเดียวกัน (ปุ่ม Open to sign), คิวรออนุมัติ +
  ลงลายเซ็นดิจิทัล, รายการเสี่ยง / เกินกำหนด
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
- ช่องค้นหาเดียว (ชื่อลาว / อังกฤษ / เลขทะเบียน) + แขวง + ประเภทวิสาหกิจ กรองทันที
  ขณะพิมพ์, ปุ่ม **More filters** เปิดช่อง ISIC และช่วงวันที่ (มี badge บอกจำนวนตัวกรอง)
- การ์ดสรุป (Overview) แบบเดียวกับ Dashboard: แถบ % ทุนที่ชำระแล้วของผลลัพธ์ที่กรองอยู่
  + ช่องสถานะ 5 ช่องที่กดเพื่อกรองได้ (นับจากผลที่ผ่านตัวกรองอื่นแล้ว จำนวนจึงตรงกับแถวที่แสดง)
- การ์ดผลลัพธ์มี Export to Excel / Print ที่หัวการ์ด; บนมือถือแต่ละแถวแสดงเป็นการ์ด
  (ชื่อ + สถานะด้านบน, ข้อมูลอื่นเป็น label / value 2 คอลัมน์) ไม่ต้องเลื่อนแนวนอน

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
- Checker/Approver มี Asset Valuation Queue พร้อม KPI

### ตรวจทาน → อนุมัติ → ใบรับรอง
- Reviewer ตรวจแล้วส่งต่อหรือตีกลับ, Approver อนุมัติ + ลงลายเซ็นดิจิทัล (ต้อง MFA
  ซ้ำ), ออกใบรับรองการฝากทุน (Print Preview + ฉบับจริง/สำเนาพร้อม watermark)

### Finance
- 3 แท็บภายในเมนูเดียว: **Fees / Penalties / E-Receipts**
- แต่ละแท็บมี search filter แยกตาม criteria ของตัวเอง (เช่น Enterprise,
  Reference, Payment status, Days overdue, Receipt no, ช่วงวันที่) กรองทันที
  ขณะพิมพ์ แท็บเป็น segmented control และปุ่ม Export / Print อยู่แถวเดียวกับแท็บ

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
  ≤900px sidebar กลายเป็น drawer (ปุ่ม ☰ บน topbar), ≤640px ปุ่มเปลี่ยนภาษา ລາວ / EN ย้ายจาก topbar
  ไปอยู่ใน drawer (ใต้โลโก้) และในเมนูผู้ใช้, ≤640px summary card จัดลำดับใหม่
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
- ชื่อสถานะบนหน้าจอใช้ภาษาลาว/ไทยเป็นหลัก และมีรหัสระบบ
  (PENDING / PARTIAL / COMPLETED / OVERDUE) กำกับไว้เพื่อเทียบกับเอกสาร requirement
- โค้ดทั้งหมดอยู่ในไฟล์เดียว (`src/index.html`) แบ่งส่วนด้วยคอมเมนต์:
  dictionary 2 ภาษา, ข้อมูล mock, helper (โดนัท / KPI / ไอคอน), ฟังก์ชัน view
  ต่อหน้าจอ (`vDash`, `vSet`, `vFin` ...), และตัว `render()` / `bind()` ที่วาดใหม่ทุกครั้ง

---

## Deploy

push ขึ้น branch `main` แล้ว GitHub Pages (Deploy from a branch, main / root) จะเสิร์ฟ
`index.html` ที่ root ให้อัตโนมัติ
