# RCMS Demo App

Prototype สำหรับนำเสนอโครงการ **Registration and Capital Monitoring System (RCMS)**
ความร่วมมือระหว่างกระทรวงอุตสาหกรรมและการค้า (MOIC) กับธนาคารส่งเสริมการเกษตร (APB)

เป็นเว็บแอปแบบ single-page ที่เขียนด้วย vanilla JS ล้วน (ไม่มี framework) รองรับ
2 ภาษา ลาว / อังกฤษ สลับได้ทันที และมีข้อมูลจำลอง (mock data) ที่ผูกตัวเลขให้
สอดคล้องกัน เพื่อให้สาธิต flow การทำงานได้จริงตั้งแต่ต้นจนจบ

> ข้อมูลทั้งหมดเป็น mock data สำหรับสาธิตเท่านั้น ไม่ได้เชื่อมฐานข้อมูลจริง

> ส่งต่อให้ทีม frontend: ดู [`docs/handoff/`](docs/handoff/README.md) (แนวทางย้ายไป React หรือ Next.js)

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
| `khamla.p` | เจ้าหน้าที่เคาน์เตอร์ APB (Maker) | Counter Workspace (Dashboard กลาง) | Dashboard, Search Enterprises |
| `sivilay.a` | เจ้าหน้าที่ตรวจสอบ (Checker) | Verification Workspace (Dashboard กลาง) | Dashboard, Search Enterprises, Valuation Queue, Finance, Reports |
| `head.moic` | หัวหน้าพะแนก (Approver + E-Sign) | Approval Workspace (Dashboard) | Approval Workspace, Search Enterprises, Valuation Queue, Finance, Reports |
| `lao.valuation` | บริษัทประเมิน (In-network firm) | Dashboard | Dashboard, My Valuation Jobs |
| `mof.gov` | หน่วยงานภายนอก (External agency) | External Portal | พอร์ทัลภายนอก |
| `bounmy.m` | ผู้ดูแลระบบสูงสุด (Super Admin) | System Overview (Dashboard กลาง) | Dashboard, Search Enterprises, Reports, System Settings |
| `new.user` | ผู้ใช้ใหม่ (New user) | My Registered Capital (ว่าง) | ทุนของฉัน |

---

## ฟีเจอร์หลักตามหน้าจอ

### หน้า Login
- โลโก้ MOIC ใหม่ (เฟือง + รวงข้าว) บนวงกลมสีขาว ฝังอยู่ในไฟล์ (`src/moic-logo.png`)
  ไม่ต้องโหลดจากเว็บ; จอเล็กซ่อนภาพฝั่งซ้ายแล้วแสดงโลโก้เหนือฟอร์มแทน
- ยืนยันตัวตนแบบ MFA (OTP), สลับภาษา ลาว / อังกฤษ, กู้รหัสผ่าน
- ข้อความภาษาอังกฤษใช้ **Log in / Log out** (ไม่ใช้ Sign in / Sign out) ภาษาลาวใช้ ເຂົ້າສູ່ລະບົບ / ອອກຈາກລະບົບ
- ชิป **Demo accounts** ใต้ปุ่ม Next กดแล้วกรอก username ให้ทันที (สะดวกตอนนำเสนอ)
- **สร้างบัญชี (Create Account)**: ลิงก์ใต้ปุ่ม Next เปิดฟอร์ม 3 ขั้น มีแถบบอกขั้นด้านบน
  1. **Applicant details**: Email, Citizen ID / Passport Number (เลือกประเภทได้อย่างใดอย่างหนึ่ง แล้วกรอกเลข),
     Role applied for (dropdown: Owner, Appraiser, MOIC Maker, MOIC Checker, MOIC Approver, Bank Maker, Bank Checker),
     ชื่อภาษาลาว (คำนำหน้า, ชื่อ, นามสกุล), ชื่อภาษาอังกฤษ (คำนำหน้า, ชื่อ, นามสกุล)
     (ตรวจครบทุกช่อง, รูปแบบอีเมล, เลขบัตรอย่างน้อย 6 ตัว, ชื่ออังกฤษใช้ได้แค่ A-Z) กด Submit แล้วระบบส่ง OTP ไปที่อีเมล
  2. **Verify email**: แจ้งว่าส่ง OTP ไปที่อีเมลที่ระบุแล้ว ให้นำรหัสมาใส่ (หากไม่พบให้ดูในโฟลเดอร์ Spam / Junk)
     ใส่ OTP 6 หลัก (demo ใส่เลขอะไรก็ได้) กด Next, มีลิงก์ Resend code และ Back กลับไปแก้อีเมล
  3. **Documents**: เนื้อหาเปลี่ยนตาม Role ที่เลือกใน step 1 (`suDocsFor()`)
     - **Owner**: Enterprise details คือ Enterprise registration number และ Enterprise type (Sole proprietorship,
       Partnership, Company, State / Joint venture, Cooperative) แล้วอัปโหลดเอกสารกลุ่ม `DOCUMENT_TYPE`
       แยกช่องละเอกสาร: Copy of Enterprise Registration Documents (จำเป็น), Power of Attorney (จำเป็น),
       Copy of Share Certificate (ไม่บังคับ) - ชุดเอกสารเดียวกันทุกประเภทวิสาหกิจ
     - **Role อื่น**: ไม่มี Enterprise details ให้อัปโหลดบัตรประชาชน (หรือพาสปอร์ต ถ้า step 1 เลือก Passport)
       และบัตรพนักงาน
     - รับเฉพาะ PDF, JPG, PNG ไม่เกิน 15 MB ต่อไฟล์ และไม่เกิน 5 ไฟล์ต่อเอกสาร (ตรวจจากไฟล์จริง ทั้งกดเลือกและลากวาง
       เลือกทีละหลายไฟล์ได้)
       ต้องอัปโหลดเอกสารที่จำเป็นให้ครบก่อนกด Create Account
- หลังกด Create Account ขึ้นหน้า **Application submitted** แจ้งว่าเอกสารรอตรวจสอบ (Owner แสดงเลขทะเบียน + ประเภทวิสาหกิจ, Role อื่นแสดง Role)
  และเมื่อเอกสารผ่านจะส่งรหัสผ่านชั่วคราวไปที่อีเมล → กลับหน้า login (กรอกอีเมลให้) → (demo: ข้ามขั้นรอตรวจ) ใส่รหัสผ่าน
  → OTP → ขึ้น modal **ตั้งรหัสผ่านใหม่** (modal เดียวกับลืมรหัสผ่าน ปิดไม่ได้จนกว่าจะตั้ง) → เข้า
  My Registered Capital ที่ยังไม่มีวิสาหกิจ บัญชี demo **New user** (`new.user`) ใช้ flow ครั้งแรกนี้
  ได้เลยโดยไม่ต้องสมัคร และหลังสร้างบัญชีนี้จะเปลี่ยนเป็นชื่อ/อีเมลที่กรอก (ชื่อตามภาษาที่เลือกอยู่)

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
- **Maker (Counter Workspace)** เมนูย่อยใต้ Dashboard (เปิดที่ Dashboard กลางก่อน): **Dashboard**, **Returned** ใบรายการสถานะ `RETURNED` (ค้นหา Enterprise, Registration no., Returned by; ปุ่ม Fix
  เข้าหน้าวิสาหกิจ แล้วกด Edit and resubmit), **Awaiting result** ใบรายการที่อยู่ระหว่างพิจารณา
  (Reference, Enterprise, Status แบบ dropdown, Submit by)
  และ **Due / Overdue** วิสาหกิจใกล้ครบ/เกินกำหนด (Enterprise, Status) ทุกรายการมีปุ่ม Search /
  Clear และ pagination
- **Dashboard กลาง** (`vCentralDash()`) ใช้ร่วมกันของ Super Admin, Maker, Checker และ Approver
  (เมนูย่อยแรกใต้ Dashboard และเป็นหน้าแรกหลัง login) ดู KPI ระดับแขวงและเมือง
  - **ขอบเขตสิทธิ์** (`ACCESS_SCOPE`): ระบบจริงรับมาจากระบบบริหารสิทธิ์อีกระบบ demo เก็บสำเนา payload ไว้ต่อ role
    - Super Admin (`bounmy.m`) เห็นทุกแขวงทุกเมือง เลือกแขวงได้ แล้วเลือกเมืองในแขวงนั้น
    - Checker / Approver เห็นได้หลายเมือง แต่เฉพาะแขวงที่ตัวเองสังกัด (แขวงล็อกไว้ เลือกได้แค่เมืองในขอบเขต)
      demo: Checker = นครหลวงเวียงจันทน์ 4 เมือง, Approver = นครหลวงเวียงจันทน์ 7 เมือง
    - Maker เห็นเฉพาะแขวงที่สังกัดเหมือนกัน (demo: นครหลวงเวียงจันทน์ 4 เมือง) แต่ขอบเขตใช้กับ Dashboard เท่านั้น
      คิวงานของ Maker ไม่ถูกกรอง เพราะเคาน์เตอร์รับเรื่องของวิสาหกิจได้ทุกแขวง
    - Dashboard ของ Maker มีแถบแจ้งใบรายการที่ถูกตีกลับ พร้อมปุ่ม Fix ไปเมนูย่อย Returned
    - แถบบนสุดบอกขอบเขตและแหล่งที่มา (Sent by the access management system)
  - **การ์ดหลัก**: ตัวเลขเด่น % ทุนจดทะเบียนที่ยืนยันแล้ว + แถบแบ่งส่วน (Confirmed / Under Review /
    Not yet submitted) พร้อม legend แล้วตามด้วย stat tile 6 ช่อง: Total Registered Capital, Confirmed Capital,
    Capital Under Review, Outstanding Capital, Overdue, Unpaid penalties (ตัวเลขย่อ เช่น 12B, ตัวเต็มอยู่ใน tooltip)
    และแถบแจ้งเตือน: งานรอตัวเอง (ปุ่มไปคิว) และวิสาหกิจเกินกำหนด + ค่าปรับ
  - **กราฟ** (ทุกกราฟมีปุ่ม Table สลับเป็นตาราง และมี tooltip ตอนชี้หรือ focus ด้วยคีย์บอร์ด)
    - *Capital confirmed per month*: กราฟแท่งแนวตั้ง 12 เดือนล่าสุด (ก.ย. 2025 - ส.ค. 2026)
      จากการชำระที่ยืนยันแล้วในประวัติ ป้ายตัวเลขเฉพาะเดือนที่สูงสุด
    - *Enterprises by payment status*: แท่งแนวนอนจำนวนวิสาหกิจต่อสถานะ + ยอดคงเหลือ
    - *Declarations in progress*: แท่งตามขั้น (Appraiser / Checker / Approver) แล้วแยกส่วนที่กลับไปหาผู้ยื่น
      (ถูกตีกลับ, คำขอปฏิเสธ) + จำนวนที่อนุมัติวันนี้
    - *Time to deadline*: วิสาหกิจที่ยังชำระไม่ครบ แบ่งเป็น เกินกำหนด (สีแดง + ไอคอน), 0-30, 31-90, 91-180,
      > 180 วัน (ใกล้กำหนดสีเข้มกว่า)
    - *By province / By district*: แท่งซ้อนต่อแขวงหรือเมือง (ความยาว = ทุนจดทะเบียน แบ่งเป็น Confirmed /
      Under Review / Not yet submitted) 8 อันดับที่ทุนคงเหลือสูงสุด กดแถวเพื่อเจาะลงระดับถัดไป
      ปุ่ม Table แสดงตารางเต็ม (จำนวนวิสาหกิจ, ทุนจดทะเบียน, % ยืนยันแล้ว, ทุนคงเหลือ, รอพิจารณา, เกินกำหนด,
      ครบกำหนดใน 30 วัน) แบบมี pagination
  - **สีกราฟ** ใช้ palette ที่ผ่านการตรวจ colorblind (ฟ้า #2a78d6 / ส้ม #eb6834 และไล่โทนฟ้าสำหรับลำดับ)
    สีแดงใช้เฉพาะสถานะเกินกำหนด และมีไอคอน + ข้อความกำกับเสมอ
  - **Enterprises needing attention**: วิสาหกิจเกินกำหนดก่อน แล้วเรียงตามวันครบกำหนดที่ใกล้สุด กดเพื่อเปิดหน้าวิสาหกิจ
  - ขอบเขตเดียวกันนี้กรองคิวงานของ Checker / Approver ด้วย: Verification queue, Awaiting approval,
    Returned by me, Overdue / penalties, Valuation Queue และตัวเลขบนเมนูด้านข้าง แต่ละหน้ามีแถบบอกว่า
    "แสดงเฉพาะงานในขอบเขตสิทธิ์" + แขวง / เมือง ส่วน Maker, Owner และบริษัทประเมินไม่ถูกกรอง
    (demo: ใบรายการที่ Maker ยื่นให้วิสาหกิจนอกนครหลวงเวียงจันทน์ จะไปอยู่ในคิวของ Checker แขวงนั้น ซึ่ง demo ไม่มีบัญชี)
  - ข้อมูล demo ในขอบเขต: ใบรายการที่ถูกตีกลับรอแก้อยู่ที่ ELN-2025-1126 (จันทะบูลี) และมีใบรายการรอตรวจ
    ที่ ELN-2025-1128, 1130 และรออนุมัติที่ ELN-2025-1129 (งวดแรก ไม่ต้อง E-Sign), 1133 (ครบ 100% ต้อง E-Sign)
- **Checker (Verification Workspace)** เมนูย่อยใต้ Dashboard: Dashboard (Dashboard กลาง), Verification
  queue, Awaiting approval, Returned by me หน้าแสดงการ์ดค้นหาแบบเดียวกับหน้า Finance &
  Penalties (Reference, Enterprise, Channel แบบ dropdown, Submit by; Returned by me มีแค่
  Enterprise ค้นได้ทั้งชื่อลาว/อังกฤษ/เลขทะเบียน) กรอกแล้วกดปุ่ม Search หรือ Enter เพื่อค้นหา
  ปุ่ม Clear ล้างเงื่อนไข แล้วตามด้วยตาราง
  ที่มี pagination เลือกได้ 5/10/20/50 แถวต่อหน้า
- **Approver (Approval Workspace)** เมนูหลักชื่อ Approval Workspace มีเมนูย่อย 4 รายการ (เปิดที่
  Dashboard ก่อน):
  - **Dashboard** คือ Dashboard กลาง (ดูด้านบน) แถบแจ้งงานถัดไปมีปุ่ม Open to sign พาไปเมนูย่อย Awaiting approval
  - **Awaiting approval** (ค้นหา Reference, Enterprise, Channel, Verified by), **Overdue / penalties**
    (Enterprise, Status) และ **Recently signed (E-Sign)** (Enterprise, Registration no.) แสดงเฉพาะ
    ตาราง มีปุ่ม Search / Clear และ pagination
- **บริษัทประเมิน (Dashboard)** การ์ด Overview: แถบ % งานที่ยืนยันแล้ว (x / 10 jobs) แล้วตามด้วย
  3 แถว: *Jobs and declared value* (งานทั้งหมด, มูลค่าที่แจ้ง, มูลค่าที่ยืนยันแล้ว, ยังไม่ยืนยัน),
  *Jobs by status* (In progress, Awaiting acceptance, Confirmed) และ *Jobs by asset type* (Land,
  Building, Car, Machine) แถบล่างบอกงานถัดไป ปุ่ม Open jobs ไปหน้า My Valuation Jobs
- **Super Admin (System Overview)** เมนูย่อยใต้ Dashboard: Dashboard (Dashboard กลาง เห็นทุกแขวงทุกเมือง) และ
  **System health** (หน้าเดิมของผู้ดูแลระบบ) ซึ่งแยกจากงานติดตามทุน มาเน้นงานดูแล
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
  สถานะ, แถบ % ที่ชำระ, ตัวเลขตามหัวข้อ Capital Summary ใน sheet Status: Total Registered Capital /
  Confirmed Capital / Capital Under Review (ยอดของใบรายการสถานะ PENDING, APPRAISED_PENDING,
  APPRAISED_COMPLETED, APPROVED) / Outstanding Capital (ทุนทั้งหมด - ทุนที่ยืนยันแล้ว) + ค่าปรับ +
  วันครบกำหนด, ปุ่มหลักมุมขวาบน และแถบแจ้งเตือนท้ายการ์ดที่มีปุ่มแก้ไขอยู่ข้างข้อความ
  (เช่น ใบแจ้งหนี้ค้าง + Pay by QR, งานที่รออนุมัติอยู่, ฉบับร่าง + Delete draft, ใบรายการที่ถูกตีกลับ + เหตุผล)
  ถ้ามีฉบับร่างหรือใบรายการที่ถูกตีกลับ ปุ่มหลักจะเป็น Continue draft / Edit and resubmit แทนการแจ้งชำระใหม่
- **Enterprise Registration Details** แบ่งเป็นกลุ่มมีหัวข้อ: Identity / Registration /
  Tax & finance / Location (+ แผนที่ Leaflet) ใช้รูปแบบ summary list คือ label ซ้ายกว้างคงที่
  แล้ว value อยู่ติดกัน (ไม่ดันไปชิดขอบขวา) เพื่อให้อ่านคู่ label/value ได้ในครั้งเดียว
- รายละเอียดธุรกรรม ใช้ layout `worksplit`: ข้อมูลอ้างอิงซ้าย, คำร้อง + เอกสารขวา
- หน้าตรวจทาน (Task ของ Checker / Approver) เต็มความกว้าง เรียงจากบนลงล่าง:
  ขั้นตอน (ช่องทางยื่น → Appraisal ถ้ามีทรัพย์สิน → Checker → Approver) → Enterprise Registration Details แบบย่อ (Registration no., Enterprise name,
  Capital account, Status + ปุ่ม Show details เปิดดูข้อมูลเต็ม 4 กลุ่ม ไม่มีแผนที่ มีลิงก์พิกัดแทน)
  → **Case detail** 6 ช่อง + สถานะใบรายการ (Reference, ทุนจดทะเบียน, ชำระแล้ว %, ยอดที่ยื่น, Channel = APB Counter /
  Online, Payment method) → **Capital being paid** การ์ดการชำระแบบอ่านอย่างเดียวทีละรายการ
  (ย่อ/ขยายได้ด้วย Show / Hide details) + แถบยอดรวม → **Application documents** → **Decision**
  (Return for correction / Reject / Approve and forward to the approver ของ Checker;
  Approver เป็น Approve and sign digitally เมื่อทุนครบ 100% หรือ Approve เมื่อยังไม่ครบ)

### บันทึกการชำระทุน (Declare Capital Payment / Add payment)
ผู้ประกอบการ (Declare capital payment online) และ Maker (Add payment) ใช้หน้าเดียวกัน เต็มความกว้าง:

- **Enterprise Registration Details แบบย่อ** ด้านบน (ปุ่ม Show details / Hide details)
- **1. Registered Capital Contribution Notification Application \*** (กล่องแรก): ลิงก์ **Download form**
  ต่อท้ายชื่อ ดาวน์โหลดแบบฟอร์มเปล่าเป็น PDF (demo เป็นไฟล์ตัวอย่าง) แล้วแนบแบบฟอร์มที่ลงลายเซ็นแล้ว
  ในช่อง Signed application form (ต้องแนบ ไม่งั้นกด Submit ไม่ได้)
- **2. Capital being paid** (รวมการเลือกงวดกับการ์ดการชำระไว้กล่องเดียว): เลือกแผนการชำระก่อน 2 แบบ
  - **Pay once 100%**: "Pay the full X LAK in one go." (X = ทุนคงเหลือ)
  - **Split into 2 instalments**: "Enter A - X LAK for instalment 1 (minimum 30%). The rest is paid in
    instalment 2." (A = 30% ของทุนจดทะเบียน) เลือกได้เฉพาะวิสาหกิจที่ยังไม่เคยชำระ ถ้าชำระงวดที่ 1 ไปแล้ว
    การ์ดนี้กดไม่ได้ และข้อความในการ์ดเปลี่ยนเป็น "Not available: instalment 1 (P LAK) is already paid,
    so this payment settles the remaining X LAK."
  - แล้วกดปุ่ม **+ Payment** เพื่อเพิ่มการ์ดการชำระ ใส่ได้หลายใบในงวดเดียว แต่ละใบมีหัว (ไอคอน + ประเภท,
    ป้าย Payment N, Remove เมื่อมีมากกว่า 1 ใบ, ยอดเงิน, Show / Hide details) และเลือก **Payment method** ได้ 3 แบบ
  - **Cash**: Amount paid, Bank code, From Account No, From Account Name, Payment evidence
  - **Asset**: Asset category, Valuation firm (In-network / Out-of-network), Valuation firm name,
    ช่องตามประเภททรัพย์สิน (`ASSET_FIELDS`), Supporting documents + ไฟล์
  - **Labour capital**: Contributor name, ID / Passport No., Skill type, Role / position,
    Contribution period (months), Agreed value, Supporting documents (Labour contribution agreement ฯลฯ)
  - การ์ดใหม่เติมยอดที่ยังค้างให้อัตโนมัติ ยอดของการ์ด = Amount paid / Estimated price / Agreed value
  - เปลี่ยน Payment method ของการ์ดที่กรอกข้อมูลแล้ว จะถามยืนยันก่อน (Change payment method?
    ข้อมูลที่กรอกจะหาย)
- **แถบยอดรวม**: ยอดแยกตามประเภทที่ใช้ (Cash / Asset (estimated) / Labour capital),
  Total this instalment, Outstanding after
- ปุ่ม Cancel / Submit for verification ชิดขวา คำร้องที่ส่งเก็บการ์ดทั้งหมดไว้ใน `task.pays`
  และ `task.mode` เป็นรายการประเภทคั่นด้วยจุลภาค (เช่น `cash,asset`)

### ประเมินทรัพย์สิน (Valuation)
- บริษัทประเมิน In-network เห็น Dashboard และ My Valuation Jobs (10 งาน รวม VAL-2026-0392
  รถ Toyota Hilux Revo ซึ่งเป็นทรัพย์สินชิ้นที่ 2 ของการแจ้งเดียวกับ VAL-2026-0341)
- หน้ารายละเอียดงานเต็มความกว้าง เรียงบนลงล่าง: แถบสรุปงาน (วิสาหกิจ, ทรัพย์สิน
  + "Asset 2 of 2 in this declaration" ถ้ามี, ประเภทเครือข่าย, มูลค่าที่แจ้ง) → การ์ด
  "Asset under valuation" (Collateral items กับ Fields submitted อยู่คู่กัน + Documents submitted
  at capital declaration) → การ์ด **Submit valuation result** (Valuation price, Remark,
  Supporting documents + ไฟล์, ปุ่มส่งชิดขวา)
- Checker/Approver มี Asset Valuation Queue พร้อม KPI, การ์ดค้นหา (Reference, Enterprise / ทรัพย์สิน,
  Valuation firm, Network type, Status แบบ dropdown + Search / Clear) และ pagination

### ตรวจทาน → อนุมัติ → ใบรับรอง
ตาม flow (User Journey - Role lanes) และ tab Status ของ sheet Status workflow

- **สถานะใบรายการชำระทุน** (`DST`, `setSt()`): `DRAFT` → `PENDING` (Cash / Labour) หรือ `APPRAISED_PENDING`
  (มีทรัพย์สิน) → `APPRAISED_COMPLETED` → `APPROVED` (Checker ผ่าน) → `COMPLETED` (Approver อนุมัติ)
  และ `RETURNED` / `REJECTED`
- **Reference ID** `CAP-YYYYMM-{running 6 หลัก}` เช่น CAP-202608-000921 ฉบับร่างยังไม่มีเลข
- **ฉบับร่าง**: ฟอร์มแจ้งชำระมีปุ่ม Save draft (แก้ / ลบได้) กลับมาแก้ต่อจากหน้าวิสาหกิจ
- **ทรัพย์สิน**: ยื่นแล้วสร้างงานประเมินให้บริษัทประเมิน 1 งานต่อทรัพย์สิน 1 รายการ
  ส่งผลครบทุกงานแล้วใบรายการจึงเป็น `APPRAISED_COMPLETED` และเข้าคิว Checker
- **Return** (Appraiser / Checker / Approver ต้องใส่เหตุผล): ใบรายการเป็น `RETURNED` กลับไปที่ Owner / Maker
  แก้แล้วยื่นใหม่ เข้าคิว Checker ตามสถานะเดิม (`PENDING` หรือ `APPRAISED_COMPLETED` ถ้าประเมินแล้ว)
  ถ้า Appraiser เป็นคนตีกลับ งานประเมินเดิมปิด และยื่นใหม่จะได้งานประเมินใหม่
- **Reject** (ต้องใส่เหตุผล): ใบรายการเป็น `REJECTED` ปิดถาวร ยื่นใหม่ไม่ได้ (ต้องแจ้งชำระเป็นใบรายการใหม่)
  - Approver กด Reject แล้วปิดทันที
  - Checker กด Reject เป็นแค่ "ขอปฏิเสธ" (Reject requested) ส่งไปให้หัวหน้ากรม (Approver) อนุมัติก่อน
    ระหว่างรอ Checker กดยกเลิกการปฏิเสธได้จากเมนู Awaiting approval แล้วงานกลับเข้าคิวตรวจสอบ
  - Approver เปิดงานที่ขอปฏิเสธ: Approve rejection → `REJECTED`, Decline rejection → กลับไปคิว Checker
- **หมายเหตุตอนตีกลับ**: Return for correction มี Internal note (เจ้าหน้าที่เห็นเท่านั้น) และ External note
  (แจ้งผู้ประกอบการ) ฝั่งเจ้าหน้าที่ (Maker / Checker / Approver) เห็นทั้ง 2 ข้อความ ในแถบแจ้งเตือนหน้าวิสาหกิจ
  หน้าแจ้งชำระที่เปิดแก้ รายการ Returned / Returned by me และประวัติ ส่วน Owner เห็นเฉพาะ External note
  หน้าตรวจทานของใบรายการที่ยื่นใหม่ มีแถบ Previously returned แสดงทั้ง 2 ข้อความของการตีกลับครั้งก่อน
- **Approve ของ Approver**: ทุนครบ 100% → ลงลายเซ็นดิจิทัล (ต้อง MFA ซ้ำ) + ออกใบรับรองการฝากทุน
  (Print Preview + ฉบับจริง/สำเนาพร้อม watermark) ถ้ายังไม่ครบ → อนุมัติงวดนั้นเป็น `COMPLETED`
  โดยไม่ต้อง E-Sign และไม่ออกใบรับรอง
- ข้อความประวัติใช้ตาม sheet เช่น Returned for revision by Checker, Rejected by Approver,
  Full Capital Payment Submitted, First / Second Capital Payment Installment Submitted

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
  (ประเภทเอกสาร + ชื่อไฟล์), ปุ่ม Open ด้านขวา (หน้าดูอย่างเดียว)
- **ช่องแนบไฟล์** (`upSlot()`) แบบ compact: ทุกช่องเรียงจากบนลงล่างในกรอบเดียว (`upList()`) แต่ละช่องเป็น 1 แถว
  ชื่อเอกสาร + จำนวนไฟล์ (n/5) + ปุ่ม Attach ด้านขวา ลากไฟล์มาวางที่แถวไหนก็ได้ ไฟล์ที่แนบแล้วแสดงเป็น chip
  ต่อกันไปในบรรทัดเดียว (ล้นแล้วขึ้นบรรทัดใหม่) พร้อมขนาดไฟล์และปุ่มลบ ✕ ชื่อไฟล์แสดงเต็ม ไม่ย่อ
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

- งวดแรกต้องไม่ต่ำกว่า 30% ของทุนจดทะเบียน (ยอดรวมทุกการ์ดต้องไม่ต่ำกว่า Minimum for this instalment)
- ยอดรวมทุกการ์ดต้องไม่เกินยอดคงเหลือ
- ต้องมีการ์ดการชำระอย่างน้อย 1 ใบ และแต่ละใบต้องกรอกช่องที่มี * ครบ
- การอัปโหลดเอกสารแยกช่องละ 1 ประเภทเอกสาร (`upSlot()`) ตาม master `DOC_GROUPS` (จาก sheet Doc_Type)
  ไม่มี dropdown เลือกประเภทเอกสารแล้ว แต่ละช่องแนบได้สูงสุด 5 ไฟล์ ไฟล์ละไม่เกิน 15 MB (PDF / JPG / PNG)
  ช่องที่มี * ต้องมีอย่างน้อย 1 ไฟล์ก่อนกด Submit
  - ทุกคำร้อง (กล่องที่ 1 ของหน้าแจ้งชำระ): Registered Capital Contribution Notification Application *
  - Cash (`CAPITAL_CASH_DOCUMENT`): Bank Statement *, Certificate of Capital Importation และ Letter from the Bank of the Lao PDR
    (จำเป็นเมื่อ Source of capital = Foreign capital เท่านั้น), Other
  - Asset - Land (`CAPITAL_LAND_DOCUMENT`): Land Title Deed *, Mortgage-Free Certificate *, Cadastral Map,
    Ownership Transfer Document, Other
  - Asset - Building (`CAPITAL_BUILDING_DOCUMENT`): Ownership Transfer Document *, Valuation Report,
    Land Title Document for the Building Site, Permit to Construct, Other
  - Asset - Car (`CAPITAL_VEHICLE_DOCUMENT`): Vehicle Registration Certificate *, Ownership Transfer Document from the
    Department of Transport *, Valuation Report, Certificate of No Encumbrance, Other
  - Asset - Machinery: ไม่มีใน sheet demo ใช้ Ownership Transfer Document *, Valuation Report, Other ไปก่อน
  - Labour capital (`CAPITAL_LABOUR_DOCUMENT`): Partnership Establishment Agreement *, Other
  - เปลี่ยน Asset category แล้วไฟล์ที่แนบไว้ในการ์ดนั้นจะถูกล้าง เพราะรายการเอกสารต่างกัน
- บริษัทประเมินส่งผล: ต้องใส่ Valuation price ก่อน แล้วต้องแนบเอกสารอย่างน้อย 1 ไฟล์
  (ช่องแยก: Valuation report, Asset photos, Site inspection form)
- ปุ่มออกใบรับรองกดไม่ได้จนกว่าทุนจะชำระครบ
- ผู้อนุมัติจะไม่เห็นงานจนกว่าจะผ่านการตรวจทานก่อน และ Checker จะไม่เห็นใบรายการที่มีทรัพย์สินจนกว่าจะประเมินครบ
- Reject ต้องใส่เหตุผล และใบรายการที่ถูก Reject ยื่นใหม่ไม่ได้
- Reject ของ Checker ต้องให้ Approver อนุมัติก่อน และ Checker ยกเลิกได้จนกว่า Approver จะตัดสิน
- Internal note ของการตีกลับ Owner มองไม่เห็น
- Create Account: ทุกช่องใน step 1 ต้องกรอก (รวม Role applied for), อีเมลต้องถูกรูปแบบ,
  เลขบัตรอย่างน้อย 6 ตัว, ชื่ออังกฤษใช้ได้แค่ A-Z, ปุ่ม Next ของ OTP กดได้เมื่อครบ 6 หลัก,
  ไฟล์ต้องเป็น PDF / JPG / PNG ไม่เกิน 15 MB (สูงสุด 5 ไฟล์ต่อเอกสาร) และต้องอัปโหลดเอกสารที่จำเป็นให้ครบ

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
- ทุกตารางที่มีวิสาหกิจแสดง Enterprise กับ Registration no. คนละคอลัมน์ และการ์ดค้นหามีช่อง
  Enterprise (ชื่อลาว / อังกฤษ) แยกจาก Registration no.
- ช่อง Submit by / Verified by / Returned by เป็น dropdown ที่ดึงรายชื่อจากข้อมูลในรายการนั้น (`optsOf()`)
- My Valuation Jobs (บริษัทประเมิน) มีการ์ดค้นหา: Reference, Enterprise / Asset, Registration no.,
  Network type และ Status (dropdown)
- ตารางที่มีการ์ดค้นหาทั้งหมดใช้ `listCard()` ตัวเดียวกัน (กรองจาก state ไม่ได้ซ่อนแถวใน DOM)
  ช่องค้นหารองรับข้อความ, dropdown, วันที่ และเงื่อนไขเฉพาะ (เช่นช่วงจำนวนวันเกินกำหนด)
- External Portal: ตารางแยกคอลัมน์ Enterprise กับ Registration no. ค้นหาด้วย Enterprise,
  Registration no. และ Province (dropdown)
- ช่อง dropdown ในการ์ดค้นหาจับคู่ค่าแบบตรงทั้งคำ (เช่นเลือก "Vientiane Province" ไม่ปน
  "Vientiane Capital") ส่วนช่องข้อความค้นหาจากบางส่วนของคำ
- ชื่อสถานะบนหน้าจอใช้ภาษาลาว/ไทยเป็นหลัก และมีรหัสระบบ
  (PENDING / INCREASE_PENDING / PARTIAL / COMPLETED / OVERDUE) กำกับไว้เพื่อเทียบกับเอกสาร requirement
- โค้ดทั้งหมดอยู่ในไฟล์เดียว (`src/index.html`) แบ่งส่วนด้วยคอมเมนต์:
  dictionary 2 ภาษา, ข้อมูล mock, helper (โดนัท / KPI / ไอคอน), ฟังก์ชัน view
  ต่อหน้าจอ (`vDash`, `vSet`, `vFin` ...), และตัว `render()` / `bind()` ที่วาดใหม่ทุกครั้ง

---

## Deploy

push ขึ้น branch `main` แล้ว GitHub Pages (Deploy from a branch, main / root) จะเสิร์ฟ
`index.html` ที่ root ให้อัตโนมัติ
