# RCMS Frontend Handoff

เอกสารชุดนี้ใช้ส่งต่อ demo (`src/index.html`) ให้ทีม frontend เอาไปทำเป็นระบบจริง

| ไฟล์ | เนื้อหา |
|---|---|
| `README.md` (ไฟล์นี้) | ส่วนที่ใช้ร่วมกันไม่ว่าจะเลือก framework ไหน: สิ่งที่ต้องยึดเป็นต้นแบบ, data model, route, component, business rule, ลำดับการย้าย |
| [`react.md`](./react.md) | แนวทางสำหรับ React SPA (Vite + React Router) |
| [`nextjs.md`](./nextjs.md) | แนวทางสำหรับ Next.js (App Router) |

---

## 1. สถานะของ demo ตอนนี้

- เป็น **prototype ที่คลิกได้** ไม่ใช่โค้ด production
  - ไฟล์เดียว (`src/index.html`) เขียนด้วย vanilla JS แบบ ES5
  - สร้าง HTML ด้วยการต่อ string แล้ว `render()` วาดใหม่ทั้งหน้าทุกครั้งที่ state เปลี่ยน
- **ไม่มี backend**
  - ข้อมูลทั้งหมดเป็น mock อยู่ใน memory และกลับค่าเดิมเมื่อ reload
  - login ใส่รหัสผ่านและ OTP อะไรก็ผ่าน
- **อย่าแปลงโค้ดทีละบรรทัด** ให้ใช้ demo เป็นต้นแบบของ:
  - หน้าตาและพฤติกรรมของแต่ละหน้าจอ (ดู `README.md` ที่ root)
  - design token และ CSS
  - ข้อความ 2 ภาษา
  - รูปร่างข้อมูล (ใช้ร่าง API contract)
  - business rule

## 2. ของที่ต้องหยิบไปใช้ (source of truth)

| ของ | อยู่ที่ | เอาไปทำอะไร |
|---|---|---|
| คำอธิบายหน้าจอทั้งหมด | `README.md` (root) | ใช้เป็น acceptance criteria รายหน้า |
| Requirement | `RCMS_Phase1_Spec.md`, `SPEC-phase1.md`, `SPEC-payment-fix.md` | ขอบเขตงานจริง |
| Design token | `src/index.html` บรรทัดต้นไฟล์ (`:root{...}`) | ย้ายเป็น `tokens.css` ได้ตรงๆ |
| CSS ของ component | `src/index.html` ส่วน `<style>` (~570 บรรทัด) | แยกตาม component (ดูข้อ 6) |
| ข้อความ UI 2 ภาษา | `var D={lo:{...},en:{...}}` | export เป็น `locales/lo.json`, `locales/en.json` |
| Mock data | ส่วน `seed data` (`seedEnts`, `MOCK_ENTS`, `seedTasks`, `VALJOBS`, `USERS_SEED`, `seedInv`, `FEES`, `MASTER`) | ทำเป็น fixture ของ mock API (MSW หรือ JSON server) |
| ฟิลด์หลักประกัน | `ASSET_FIELDS`, `ASSET_REQ`, `ASSET_PRICE_KEY` | ใช้สร้าง schema ฟอร์ม (ตรงกับ `collateral_fields (1).xlsx`) |
| ไอคอน | ชุด Lucide (`var LU`) | ใช้ `lucide-react` ชื่อเดียวกัน |
| โลโก้ / ภาพพื้นหลัง login | `src/moic-logo.png`, `src/MOIC01.webp` | ใส่ใน `public/` |

ตัวอย่างสคริปต์ export ข้อความ (รันใน DevTools console ของหน้า demo ไม่ได้ เพราะ `D` อยู่ใน module scope ให้ copy object `D` ออกมาไปไว้ในไฟล์ `.mjs` แล้วรันด้วย Node):

```js
// extract-i18n.mjs: วาง object D จาก src/index.html ไว้ตรงนี้
const D = { lo: { /* ... */ }, en: { /* ... */ } };
import { writeFileSync } from 'node:fs';
for (const lng of ['lo', 'en']) writeFileSync(`locales/${lng}.json`, JSON.stringify(D[lng], null, 2));
```

## 3. Data model (ร่างจาก mock)

ร่างนี้ได้มาจาก mock ใน demo ต้องยืนยันกับทีม backend ก่อนใช้จริง
- ข้อความที่มี 2 ภาษาเก็บเป็น `{ lo, en }`
- วันที่ใน demo เป็น `dd/mm/yyyy` แต่ API ควรใช้ ISO `yyyy-mm-dd` แล้วค่อย format ตอนแสดงผล

```ts
type I18nText = { lo: string; en: string };
type EntStatus = 'PENDING' | 'PARTIAL' | 'COMPLETED' | 'OVERDUE';
type Role = 'owner' | 'maker' | 'checker' | 'approver' | 'valfirm' | 'external' | 'admin';

interface Enterprise {
  id: string;            // Registration no. เช่น ELN-2025-1001
  name: string;          // ชื่อภาษาลาว
  nameEn: string;
  rep: string;           // ผู้แทน
  type: I18nText;        // ประเภทวิสาหกิจ
  prov: I18nText;        // แขวง (ค่าที่ใช้กรองคือ prov.lo)
  village?: I18nText;
  lat?: number; lng?: number;
  isic: string;
  reg: string;           // วันจดทะเบียน
  due: string;           // วันครบกำหนดชำระทุน
  taxRegDate?: string;
  cap: number;           // ทุนจดทะเบียน (LAK)
  paid: number;          // ชำระแล้ว
  status: EntStatus;
  acct: string;          // บัญชี APB
  taxris: string;
  collateral: Collateral[];
  insts: Instalment[];
  hist: HistoryEntry[];
}

interface Collateral {
  type: 'land' | 'building' | 'car' | 'machine';
  label: I18nText; detail: I18nText; value: number;
  fields: Record<string, string>;   // key ตาม ASSET_FIELDS[type]
}

interface Instalment { no: number; amt: number; state: 'paid' | 'due' | 'wait'; meta: I18nText }

interface HistoryEntry {
  d: string; a: I18nText;
  m: 'cash' | 'asset' | '';  v: number;
  u: string;                 // ผู้ดำเนินการ
  f: string;                 // จำนวนไฟล์
  ch: string;                // ช่องทาง เช่น chCounter
  st: 'complete' | 'pending' | 'returned';
}

// คำร้องแจ้งชำระทุนที่ Maker ยื่น
interface Declaration {
  ref: string;               // APR-2026-0912
  entId: string;             // demo เก็บทั้ง object (ent) แต่ API ควรส่ง id
  amt: number; mode: 'cash' | 'asset';
  by: string; at: string;
  stage: 'checker' | 'approve';
  files: { k: string; lbl: string; name: string }[];
  trail: { who: string; act: I18nText; at: string }[];
}

interface ValuationJob {
  id: string;                // VAL-2026-0341
  ent: string; asset: I18nText;
  firm: string; net: 'in' | 'out';
  amt: number;
  status: 'doing' | 'wait' | 'outapprove' | 'accepted';
  files?: string[];
}

interface Invoice {
  id: string; ent: string;
  kind: 'fee' | 'penalty';
  title: I18nText; amt: number; due: string;
  status: 'PAID' | 'UNPAID';
  paidAt: string | null; receipt: string | null; channel: string;
}

interface StaffUser { nm: string; org: string; role: string; active: 0 | 1; pwd: string; online: 0 | 1 }
```

## 4. Role, เมนู และ route

ใน demo ใช้ `S.view` + `S.dashTab` / `S.finTab` / `S.setTab` แทน URL ระบบจริงควรแปลงเป็น route
เพื่อให้กด back, refresh และแชร์ลิงก์ได้

| Route (เสนอ) | View ใน demo | Role ที่เข้าได้ |
|---|---|---|
| `/login`, `/signup` | `vLogin` (step `cred` / `otp` / `signup` / `signupDone`) | ทุกคน (ยังไม่ login) |
| `/my-capital` | `vOwnerList` | owner, newuser |
| `/my-capital/:id` | `vOwnerDash` | owner |
| `/workspace/:tab` | `vMakerDash` (`ret`, `queue`, `due`) | maker |
| `/workspace/:tab` | `vCheckerDash` (`queue`, `atAp`, `ret`) | checker |
| `/workspace/:tab` | `vApproverDash` (`queue`, `risk`, `signed`) | approver |
| `/overview` | `vAdminDash` | admin |
| `/enterprises` | `vList` | maker, checker, approver, admin |
| `/enterprises/:id` | `vDetail` | maker, checker, approver, admin |
| `/enterprises/:id/declare` | `vDeclare` | maker, owner |
| `/enterprises/:id/certificate` | `vCert` | owner และ staff ที่เปิดรายละเอียดวิสาหกิจได้ (เมื่อชำระครบ) |
| `/transactions/:id` | `vTxDetail` | owner, staff |
| `/tasks/:ref` | `vTask` | checker, approver |
| `/valuation-queue` | `vValQueue` | checker, approver |
| `/valuation-jobs`, `/valuation-jobs/:id` | `vValFirm`, `vValJobDetail` | valfirm |
| `/finance/:tab` | `vFin` (`fee`, `pen`, `rec`) | checker, approver |
| `/reports` | `vRep` | checker, approver, admin |
| `/settings/:tab` | `vSet` (`fee`, `master`, `gl`, `user`, `api`) | admin |
| `/portal` | `vPortal` | external |

- เมนู sidebar ของแต่ละ role ดูได้จาก `menus()`
- หน้าแรกหลัง login ดูได้จาก `roleHome()` และ `firstTab()`
- ระบบจริงต้องกัน route ตาม role ทั้งฝั่ง frontend และ backend
- ห้ามพึ่งแค่การซ่อนเมนู

## 5. Business rule ที่ต้องทำให้ครบ

ต้อง validate ฝั่ง frontend เพื่อ UX และ **ต้องบังคับซ้ำฝั่ง backend**

- **การชำระทุน** (handler `#submitDeclare` ใน `bind()`)
  - แบ่งชำระ 2 งวดได้เฉพาะเมื่อยังไม่เคยชำระ (`paid === 0`) ถ้าเคยชำระแล้วต้องชำระเต็มยอดคงเหลือ
  - งวดแรกต้องไม่ต่ำกว่า 30% ของทุนจดทะเบียน
    - **demo แสดงยอดขั้นต่ำ (`dcMinInst`) แต่ไม่ได้ตรวจตอนกดส่ง** ระบบจริงต้องตรวจ
  - ต้องใส่จำนวนเงิน (`errAmt`) และต้องไม่เกินยอดคงเหลือ `cap - paid` (`errOver`)
  - ชำระเป็นทรัพย์สิน: ต้องกรอกช่องบังคับของประเภททรัพย์สิน (`ASSET_REQ`, `errAssetField`)
  - ต้องแนบเอกสารหลักฐานอย่างน้อย 1 ไฟล์ (`errFile`)
    - ถ้าชำระเป็นทรัพย์สินต้องแนบรายงานประเมิน (Valuation report) (ดู `docsErrMsg()`)
  - วิสาหกิจที่มีคำร้องค้างอยู่ ยื่นซ้ำไม่ได้ (`pendingOf()`, `hasPending`)
- **ลำดับงาน**
  - Maker ยื่น → Checker ตรวจ (ส่งต่อ หรือตีกลับพร้อมเหตุผล) → Approver อนุมัติ + E-Sign (ต้อง MFA ซ้ำ)
  - Approver จะไม่เห็นงานจนกว่า Checker จะส่งต่อ
- **สถานะวิสาหกิจ**
  - ชำระครบแล้วเป็น `COMPLETED` ถ้ายังไม่ครบเป็น `PARTIAL` (ดู `approve()`)
  - เกินวันครบกำหนดเป็น `OVERDUE` และมีค่าปรับ (ดู `penOf()`)
- **ใบรับรอง**: ออกได้เมื่อทุนชำระครบเท่านั้น
- **Sign-up**
  - ทุกช่องต้องกรอก, อีเมลต้องถูกรูปแบบ, เลขบัตรอย่างน้อย 6 ตัว
  - บัญชีที่ใช้รหัสผ่านชั่วคราวต้องตั้งรหัสใหม่หลัง OTP โดยปิด modal ไม่ได้
- **ค่าธรรมเนียม**: ตาม `FEES` (ตามประเภทวิสาหกิจ และตามช่วงทุน)
  - อัตราค่าปรับต่อวันยังไม่ได้กำหนดใน TOR (ข้อความ `penRateTBD`) ต้องถามลูกค้า
- **"วันนี้"**: demo ล็อกไว้ที่ 04/08/2026 (`daysLeft()`) ระบบจริงต้องใช้วันที่จาก server

## 6. รายการ component ที่ควรแยก

| Component | มาจาก (demo) | หมายเหตุ |
|---|---|---|
| `AppShell` (Sidebar, Topbar) | `shell()` | sidebar ย่อได้, จอ ≤900px เป็น drawer |
| `SidebarMenu` | `menus()` | รายการเมนูย่อยมีตัวเลข badge |
| `NotificationBell`, `UserMenu`, `LangSwitch` | `bellPanel()`, `shell()` | |
| `FilterBar` | `filterBar()`, `colFilterBar()` | ปุ่ม Search (icon `search`) + Clear (icon `brush-cleaning`) |
| `DataTable` + `Pager` | `listCard()`, `paginate()` | 5/10/20/50 แถวต่อหน้า |
| `ListCard` | `listCard()` | รวม FilterBar + ตาราง + pager + จำนวน `hit / total` |
| `SummaryCard`, `Metric` | `sumCard()`, `metric()` | ค่า 0 เป็นสีเทา, ค่าที่ต้องทำเป็นสีสถานะ |
| `StatusPill`, `HistoryPill` | `pill()`, `histPill()` | มีจุดสีนำหน้าเสมอ |
| `DocRow` | `docRow()` | แถวเอกสารแบบเดียวกันทั้งระบบ |
| `Donut` | helper โดนัท | palette `#283579 #5B8DEF #137A41 #E6B92C #C8202E #8B6FD6` |
| `Modal` + ชนิดต่างๆ | `vModal()` | `approve`, `verify`, `backreason`, `receipt`, `qr`, `valjob`, `adduser`, `forgot`, `done`, `help`, `keys`, `clearcache` |
| `OtpInput` | `otpWire()` | 6 ช่อง, เลื่อนช่องอัตโนมัติ, Backspace ย้อนกลับ |
| `Toast` | `toast()` | แสดงกลางล่าง |
| `MapView` | Leaflet ใน `vDetail` | ต้องรันฝั่ง client เท่านั้น |
| `WorkSplit` layout | หน้า Task / Transaction | ซ้ายข้อมูลอ้างอิง ขวาส่วนตัดสินใจแบบ sticky |

## 7. พฤติกรรมที่ต้องเก็บไว้ให้เหมือน demo

- **การ์ดค้นหา**
  - พิมพ์แล้วยังไม่กรอง ต้องกด Search หรือ Enter ก่อน (draft แยกจาก applied)
  - ปุ่ม Clear ล้างทั้งสองอย่าง
- **การจับคู่**
  - ช่องข้อความหาแบบ substring ไม่สนตัวพิมพ์ใหญ่/เล็ก และไม่สนเครื่องหมาย `,`
  - dropdown จับคู่ตรงทั้งค่า
  - บางช่องมีเงื่อนไขเฉพาะ เช่นช่วงจำนวนวันเกินกำหนด
- **Pagination**
  - เปลี่ยนเงื่อนไขค้นหาแล้วกลับไปหน้า 1 เสมอ
  - ถ้าหน้าปัจจุบันเกินจำนวนหน้า ให้ดึงกลับมาหน้าสุดท้าย
- **ตารางที่มีวิสาหกิจ**
  - Enterprise กับ Registration no. อยู่คนละคอลัมน์
  - การ์ดค้นหาก็แยกสองช่องเช่นกัน
- **Dropdown Submit by / Verified by / Returned by**
  - ตัวเลือกดึงจากข้อมูลจริงของรายการนั้น (`optsOf()`)
  - ระบบจริงควรมี endpoint คืนรายชื่อให้
- **แขวง**: ค่าใน dropdown ไม่ขึ้นกับภาษา (ใช้ `prov.lo` เป็น value)
- **i18n**
  - ค่าเริ่มต้นคือภาษาลาว และเก็บ `<html lang>` ให้ตรงภาษา
  - font: ภาษาลาวใช้ Noto Sans Lao ภาษาอังกฤษใช้ IBM Plex Sans ตัวเลขใช้ IBM Plex Mono (`.num`)
- **Responsive breakpoint**
  - ≤1180px: KPI 2 คอลัมน์
  - ≤980px: work page คอลัมน์เดียว
  - ≤900px: sidebar เป็น drawer
  - ≤640px: summary card เรียงใหม่
- **ข้อความ**: ใช้ `-` ไม่ใช้ em-dash

## 8. ของที่เป็น demo-only (ไม่ต้องย้าย)

- ชิป Demo accounts และ datalist ชื่อผู้ใช้ที่หน้า login
- เมนูสลับ role ในเมนูผู้ใช้
- Reset demo data / Clear cache และคีย์ลัด `Alt+R`
- รหัสผ่าน / OTP อะไรก็ผ่าน
- วันที่ล็อก 04/08/2026
- mobile preview ของ owner (`Alt+M`, `vMobile`)
- โค้ดที่ไม่มีทางเข้า: `vStaffDash`, `vQueue` (view `inspect` / `approveQ`)

คีย์ลัดที่ควรพิจารณาเก็บไว้: `Alt+L` สลับภาษา, `Alt+B` ย่อ sidebar, `Alt+/` ดูคีย์ลัด

## 9. API ที่ frontend ต้องการ (ร่าง)

```
POST   /auth/login                 { user, password } -> { challengeId }
POST   /auth/otp                   { challengeId, otp } -> session (+ mustChangePassword)
POST   /auth/password              { newPassword }
POST   /auth/forgot                { email }
POST   /auth/signup                { email, citizenId, title, firstName, lastName }

GET    /enterprises                ?name=&regNo=&prov=&type=&isic=&regFrom=&regTo=&updFrom=&updTo=&status=&page=&size=
GET    /enterprises/:id
GET    /me/enterprises             (owner)
POST   /enterprises/:id/declarations   multipart: amt, mode, files[], collateral

GET    /declarations               ?stage=checker|approve&ref=&ent=&regNo=&channel=&by=&page=&size=
GET    /declarations/:ref
POST   /declarations/:ref/verify   (checker: ส่งต่อ)
POST   /declarations/:ref/return   { reason, internal, external }
POST   /declarations/:ref/approve  { otp } (approver: E-Sign)

GET    /valuation-jobs             ?ref=&ent=&regNo=&firm=&net=&status=&page=&size=
GET    /valuation-jobs/:id
POST   /valuation-jobs/:id/result  multipart: value, files[], docType

GET    /invoices                   ?kind=fee|penalty&...
GET    /receipts
GET    /reports/:type              ?from=&to=
GET    /certificates/:entId        (PDF)

GET    /lookups/{provinces,isic,entity-types,valuation-firms,doc-types,staff?role=}
GET    /admin/{users,gl,api-log,audit,fees,master}
```

- list endpoint ทุกตัวควรคืน `{ items, total, page, size }`
- frontend จะได้ไม่ต้องดึงทั้งหมดมากรองเองแบบ demo

## 10. ลำดับงานที่แนะนำ

1. **Foundation**
   - ตั้งโปรเจกต์ ย้าย `tokens.css` ฟอนต์ และ i18n (lo / en)
   - ทำ mock API จาก seed data
2. **Shell + Auth**
   - login, OTP, forced password change, sign-up
   - `AppShell`, เมนูตาม role, route guard
3. **Shared list**
   - `FilterBar`, `DataTable`, `Pager`, `ListCard`, `SummaryCard`, `StatusPill`
   - ทำเสร็จก่อนแล้วหน้ารายการทั้งหมดจะเร็วขึ้นมาก
4. **Flow หลัก (Maker → Checker → Approver)**
   - Search Enterprises, Enterprise detail, Declare, Task, Approve + E-Sign, Certificate
5. **Valuation**: Valuation queue (staff), My Valuation Jobs + submit result (firm)
6. **Owner**: My Registered Capital list, Capital details, invoice / QR
7. **ส่วนอื่น**: Finance, Reports, External Portal, Admin overview + Settings
8. **ทดสอบความเหมือน**: เปิด demo กับระบบใหม่คู่กันทีละหน้า ทั้ง 2 ภาษาและทุก breakpoint

## 11. เลือก React หรือ Next.js

| คำถาม | ถ้าตอบว่าใช่ |
|---|---|
| backend เป็นระบบแยก (เช่น Java / .NET / Go) และ frontend แค่เรียก API | React SPA พอ |
| ทุกหน้าอยู่หลัง login ไม่ต้องการ SEO | React SPA พอ |
| ต้อง deploy เป็นไฟล์ static (Nginx, S3, ภายในหน่วยงาน) โดยไม่มี Node server | React SPA |
| อยากมี BFF / API route ใน repo เดียวกับ frontend | Next.js |
| อยากซ่อน token / API key ไว้ฝั่ง server, ทำ session ด้วย httpOnly cookie ง่ายๆ | Next.js |
| มีหน้า public ที่ต้องโหลดเร็วหรือติด search engine (เช่นหน้าตรวจสอบใบรับรองผ่าน QR) | Next.js |
| ทีมมี infra รัน Node server หรือใช้ Vercel ได้ | Next.js |

**ข้อแนะนำ**
- ระบบนี้เป็นระบบภายในหลัง login เกือบทั้งหมด ถ้ามี backend API แยกอยู่แล้ว **React SPA (Vite) ง่ายและเบากว่า**
- เลือก Next.js เมื่อต้องการ BFF, session ฝั่ง server หรือหน้า public ที่ต้อง render ฝั่ง server
