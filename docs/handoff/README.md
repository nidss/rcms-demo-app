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
type EntStatus = 'PENDING' | 'INCREASE_PENDING' | 'PARTIAL' | 'COMPLETED' | 'OVERDUE';
// สถานะใบรายการชำระทุน (sheet Status workflow, tab Status)
type DeclStatus = 'DRAFT' | 'PENDING' | 'APPRAISED_PENDING' | 'APPRAISED_COMPLETED' | 'APPROVED'
  | 'RETURNED' | 'COMPLETED' | 'REJECTED';
type Role = 'owner' | 'maker' | 'checker' | 'approver' | 'valfirm' | 'external' | 'admin';

interface Enterprise {
  id: string;            // Registration no. เช่น ELN-2025-1001
  name: string;          // ชื่อภาษาลาว
  nameEn: string;
  rep: string;           // ผู้แทน
  type: I18nText;        // ประเภทวิสาหกิจ
  prov: I18nText;        // แขวง (ค่าที่ใช้กรองคือ prov.lo)
  district?: I18nText;   // เมือง (ไม่มี = demo คิดจาก PROV_META / DISTRICTS ด้วย entDistrict())
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

// คำขอสร้างบัญชี (Create Account) ที่รอเจ้าหน้าที่ตรวจเอกสาร
type SignupRole = 'owner' | 'appraiser' | 'moicMaker' | 'moicChecker' | 'moicApprover' | 'bankMaker' | 'bankChecker';
type EntityType = 'sole' | 'partnership' | 'company' | 'statejv' | 'coop';
interface SignupApplication {
  email: string;               // ต้องยืนยันด้วย OTP ก่อนถึงขั้นเอกสาร
  idType: 'cid' | 'pp';        // Citizen ID หรือ Passport (เลือกอย่างใดอย่างหนึ่ง)
  idNo: string;
  role: SignupRole;
  nameLo: { title: 'mr' | 'mrs' | 'ms'; first: string; last: string };
  nameEn: { title: 'mr' | 'mrs' | 'ms'; first: string; last: string };
  regNo?: string;              // เฉพาะ owner
  entityType?: EntityType;     // เฉพาะ owner
  docs: { k: string; name: string; size: number }[];   // k ตาม SU_DOC เช่น regCert, idCard, staffCard
  status: 'pending' | 'approved' | 'rejected';         // approved แล้วจึงส่งรหัสผ่านชั่วคราวทางอีเมล
}

interface Collateral {
  type: 'land' | 'building' | 'car' | 'machine';
  label: I18nText; detail: I18nText; value: number;
  fields: Record<string, string>;   // key ตาม ASSET_FIELDS[type]
}

interface Instalment { no: number; amt: number; state: 'paid' | 'due' | 'wait'; meta: I18nText }

interface HistoryEntry {
  d: string; a: I18nText;
  m: string;                 // ประเภทการชำระคั่นด้วยจุลภาค เช่น 'cash' หรือ 'cash,asset' ('' = ไม่มียอด)
  v: number;
  u: string;                 // ผู้ดำเนินการ
  f: string;                 // จำนวนไฟล์
  ch: string;                // ช่องทาง เช่น chCounter
  st: 'complete' | 'pending' | 'returned' | 'rejected';
}

// คำร้องแจ้งชำระทุนที่ Maker หรือผู้ประกอบการยื่น (หนึ่งงวด มีการชำระได้หลายรายการ)
interface Declaration {
  ref?: string;              // CAP-YYYYMM-{running 6 หลัก} เช่น CAP-202608-000912 ฉบับร่างยังไม่มี
  entId: string;             // demo เก็บทั้ง object (ent) แต่ API ควรส่ง id
  amt: number;               // ยอดรวมทุกรายการใน pays
  mode: string;              // ประเภทที่ใช้ คั่นด้วยจุลภาค เช่น 'cash,asset,labour'
  pays: Payment[];           // seed เก่าไม่มี demo สร้างให้ตอนเปิดดู (taskPays())
  ch?: 'counter' | 'online'; // ช่องทางยื่น (ไม่มี = ดูจาก trail)
  by: string; at: string;
  st: DeclStatus;
  stage: 'draft' | 'appraise' | 'checker' | 'approve' | 'returned' | 'closed';   // คิดจาก st (DST_STAGE)
  appraised?: boolean;       // ประเมินครบแล้ว ยื่นใหม่หลังถูกตีกลับจึงไปที่ APPRAISED_COMPLETED เลย
  payMode?: 'full' | 'partial';
  ret?: { by: 'appraiser' | 'checker' | 'approver'; reason: string | I18nText; internal?: string | I18nText; u: string; d: string };
  lastRet?: Declaration['ret'];   // การตีกลับครั้งก่อน แสดงให้เจ้าหน้าที่ที่ตรวจใบรายการที่ยื่นใหม่
  rejReq?: { reason: string; u: string; d: string };   // Checker ขอปฏิเสธ รอ Approver (stage = 'approve')
  rej?: { by: 'checker' | 'approver'; reason: string; u: string; d: string };
  files: DocFile[];          // ไฟล์ทุกรายการรวมกัน (ใช้นับจำนวน)
  docs?: DocFile[];          // เอกสารระดับคำร้อง (กลุ่ม CAPITAL_DECLARATION_DOCUMENT) seed เก่าไม่มี
  trail: { who: string; act: I18nText; at: string }[];
}

// k = param_code ของเอกสารในกลุ่มของมัน (ดู DOC_GROUPS) หนึ่ง k มีได้หลายไฟล์ (สูงสุด 5, ไฟล์ละ 15 MB)
// ชื่อเอกสารไม่เก็บในไฟล์แล้ว ให้ lookup จาก DOC_GROUPS ตามภาษาปัจจุบัน
interface DocFile { k: string; name: string; size?: string }

// master เอกสาร (จาก sheet Doc_Type): group_code -> รายการเอกสาร
// req: 1 = จำเป็น, 0 = ไม่บังคับ, 'foreign' = จำเป็นเฉพาะทุนต่างประเทศ
// Cash ที่ src = domestic ซ่อนช่องที่ req = 'foreign' (เหลือ Bank Statement กับ Other) ส่วน foreign แสดงทุกช่อง
// กลุ่ม: DOCUMENT_TYPE (ขอ user), CAPITAL_DECLARATION_DOCUMENT (placeholder - sheet ยังเว้น code ไว้),
// CAPITAL_CASH_DOCUMENT, CAPITAL_VEHICLE_DOCUMENT, CAPITAL_BUILDING_DOCUMENT, CAPITAL_LAND_DOCUMENT,
// CAPITAL_LABOUR_DOCUMENT, CAPITAL_MACHINE_DOCUMENT (ไม่มีใน sheet demo เพิ่มเอง)
type DocGroups = Record<string, { k: string; req: 0 | 1 | 'foreign'; lo: string; en: string }[]>;

// หนึ่งรายการใน "Capital being paid" (การ์ดหนึ่งใบ)
type Payment =
  | { type: 'cash'; src: 'domestic' | 'foreign'; amt: string; bank: string; acctNo: string; acctName: string; files: DocFile[] }
  | { type: 'asset'; cat: 'land' | 'building' | 'car' | 'machine'; firmType: 'moic' | 'in' | 'out'; firmName: string;   // moic = MOIC Checker ประเมินเอง ไม่ผ่าน appraiser
      vals: Record<string, string>;   // key ตาม ASSET_FIELDS[cat], ยอด = vals[ASSET_PRICE_KEY[cat]]
      files: DocFile[] }
  | { type: 'labour'; firmType: 'moic' | 'in' | 'out'; firmName: string;   // ไม่มี firmType (ข้อมูลเก่า) = moic
      lb: { name: string; idNo: string; skill: string; role: string; months: string; value: string; info: string };   // info = Additional information
      files: DocFile[] };

interface ValuationJob {
  id: string;                // VAL-2026-0341
  ent: string; asset: I18nText;
  type: 'land' | 'building' | 'car' | 'machine' | 'labour';   // ใช้แบ่งกลุ่มใน Dashboard ของบริษัทประเมิน
  fields?: Record<string, string>;   // ข้อมูลที่กรอกตอนแจ้ง (key ตาม ASSET_FIELDS[type])
  declGroup?: string; declFiles?: DocFile[];   // เอกสารของ block ที่แนบตอนแจ้ง ไม่มี = งาน seed แสดง valuation report ตัวอย่าง
  part?: [number, number];   // เช่น [2, 2] = ทรัพย์สินชิ้นที่ 2 จาก 2 ในการแจ้งเดียวกัน
  remark?: string;           // หมายเหตุที่บริษัทประเมินใส่ตอนส่งผล
  firm: string; net: 'in' | 'out';
  amt: number;
  status: 'doing' | 'wait' | 'outapprove' | 'accepted';
  files?: string[];
}

interface Invoice {
  id: string; ent: string;
  kind: 'fee' | 'penalty';
  title: I18nText; amt: number; due: string;
  status: 'PENDING' | 'PAID' | 'WAIVED';   // WAIVED: role ที่ยกเว้นได้ยังรอ CF (demo ยังไม่มีปุ่ม)
  paidAt: string | null; receipt: string | null; channel: string;
}

interface StaffUser { nm: string; org: string; role: string; active: 0 | 1; pwd: string; online: 0 | 1 }
```

## 4. Role, เมนู และ route

ใน demo ใช้ `S.view` + `S.dashTab` / `S.finTab` / `S.setTab` แทน URL ระบบจริงควรแปลงเป็น route
เพื่อให้กด back, refresh และแชร์ลิงก์ได้

| Route (เสนอ) | View ใน demo | Role ที่เข้าได้ |
|---|---|---|
| `/login` | `vLogin` (step `cred` / `otp`) | ทุกคน (ยังไม่ login) |
| `/create-account` | `vLogin` (step `signup` ซึ่งมี `S.signup.step` 1-3 และ `signupDone`) | ทุกคน (ยังไม่ login) |
| `/my-capital` | `vOwnerList` | owner, newuser |
| `/my-capital/:id` | `vOwnerDash` | owner |
| `/workspace/:tab` | `vMakerDash` (`ret`, `queue`, `due`) | maker |
| `/dashboard?prov=&dist=` | `vCentralDash` (tab `overview`) | admin (Super Admin), maker, checker, approver |
| `/workspace/:tab` | `vCheckerDash` (`queue`, `atAp`, `ret`) | checker |
| `/workspace/:tab` | `vApproverDash` (`queue`, `risk`, `signed`) | approver |
| `/system` | `vAdminDash` (tab `sys`) | admin |
| `/enterprises` | `vList` | maker, checker, approver, admin |
| `/enterprises/:id` | `vDetail` | maker, checker, approver, admin |
| `/enterprises/:id/declare` | `vDeclare` | maker, owner |
| `/enterprises/:id/certificate` | `vCert` | owner และ staff ที่เปิดรายละเอียดวิสาหกิจได้ (เมื่อชำระครบ) |
| `/transactions/:id` | `vTxDetail` | owner, staff |
| `/tasks/:ref` | `vTask` | checker, approver |
| `/valuation-queue` | `vValQueue` | checker, approver |
| `/valuation-dashboard` | `vValDash` | valfirm |
| `/valuation-jobs`, `/valuation-jobs/:id` | `vValFirm`, `vValJobDetail` | valfirm |
| `/finance/:tab` | `vFin` (`fee`, `pen`, `rec`) | checker, approver |
| `/reports` | `vRep` | checker, approver, admin |
| `/settings/:tab` | `vSet` (`fee`, `master`, `gl`, `user`, `api`) | admin |
| `/portal` | `vPortal` | external |

- เมนู sidebar ของแต่ละ role ดูได้จาก `menus()`
- หน้าแรกหลัง login ดูได้จาก `roleHome()` และ `firstTab()`
- ระบบจริงต้องกัน route ตาม role ทั้งฝั่ง frontend และ backend
- ห้ามพึ่งแค่การซ่อนเมนู
- **ขอบเขตข้อมูลของ Dashboard กลาง** มาจากระบบบริหารสิทธิ์อีกระบบ (demo: `ACCESS_SCOPE`)
  ```ts
  type AccessScope = { all: true }                        // Super Admin: ทุกแขวง ทุกเมือง
    | { all?: false; prov: string; districts: string[] }; // Checker / Approver: แขวงเดียว หลายเมือง
  ```
  - backend ต้องกรองข้อมูลตามขอบเขตนี้เอง (ไม่ส่งข้อมูลนอกขอบเขตมาให้ frontend) และ frontend ใช้แค่สร้าง dropdown
  - demo ใช้ขอบเขตกับ Dashboard กลางและคิวงานของ Checker / Approver (`queueOk()` = `inScope()` กับขอบเขตของคนที่ login)
    ระบบจริงต้องกรองที่ API ของคิวงานด้วย (`GET /declarations`, `GET /valuation-jobs`) ไม่ใช่แค่ซ่อนใน frontend
- **กราฟของ Dashboard กลาง** demo เขียนเป็น HTML/CSS เอง ตอนย้ายไป React / Next.js ใช้ library ได้ (เช่น Recharts)
  แต่ต้องเก็บกติกาเดิม: palette เดิม, แท่งบาง (<= 24px) มุมมน 4px ฝั่งปลาย, เส้น grid บาง ไม่เป็นเส้นประ,
  ตัวเลขที่ปลายแท่ง, tooltip ทั้ง hover และ focus, และทุกกราฟต้องสลับเป็นตารางได้
  ข้อมูลรายเดือนควรมาจาก API (`GET /dashboard/summary` คืน series 12 เดือน) แทนการคำนวณจากประวัติแบบ demo

## 5. Business rule ที่ต้องทำให้ครบ

ต้อง validate ฝั่ง frontend เพื่อ UX และ **ต้องบังคับซ้ำฝั่ง backend**

- **การชำระทุน** (handler `#submitDeclare` ใน `bind()`)
  - แบ่งชำระ 2 งวดได้เฉพาะเมื่อยังไม่เคยชำระ (`paid === 0`) ถ้าเคยชำระแล้วต้องชำระเต็มยอดคงเหลือ
  - ต้องมีการชำระอย่างน้อย 1 รายการ (`errNoPay`)
  - ยอดรวมทุกรายการ (`payTotal()`) ต้องไม่ต่ำกว่ายอดขั้นต่ำของงวด (`errMinInst`; งวดแรกแบบแบ่ง = 30%
    ของทุนจดทะเบียน, แบบ Single = ยอดคงเหลือทั้งหมด) และต้องไม่เกิน `cap - paid` (`errOver`)
  - ตรวจทีละรายการด้วย `payErr()` แล้วแสดง error ใต้การ์ดนั้น (การ์ดที่ย่ออยู่จะเปิดเอง)
    - Cash: จำนวนเงิน, Bank code, From Account No / Name และ Payment evidence
    - Asset: ช่องบังคับของประเภททรัพย์สิน (`ASSET_REQ`, `errAssetField`), Estimated price และ Valuation report
    - Labour capital: Contributor name, ID / Passport No., Skill type, Agreed value และ Labour contribution agreement
  - เปลี่ยนประเภทของการ์ดที่กรอกแล้วต้องยืนยันก่อน (modal `chgpay`) เพราะข้อมูลในการ์ดจะถูกล้าง
  - วิสาหกิจที่มีใบรายการระหว่างพิจารณา (`LIVE_ST`) ยื่นซ้ำไม่ได้ (`pendingOf()`, `hasPending`)
  - ฉบับร่างหรือใบรายการที่ถูกตีกลับ เปิดแก้ต่อจากหน้าวิสาหกิจ (`openOf()`, `data-editdecl`) แทนการสร้างใหม่
- **ลำดับงาน**
  - Owner / Maker ยื่น (`DRAFT` → `PENDING`, หรือ `APPRAISED_PENDING` ถ้ามีทรัพย์สิน)
  - Appraiser ส่งผลครบทุกงาน → `APPRAISED_COMPLETED` (`valDone()`) หรือตีกลับ → `RETURNED`
  - Checker: Return → `RETURNED`, Reject → ขอปฏิเสธ (`rejReq`) รอ Approver, Approve → `APPROVED` ส่งต่อ Approver
    - Checker ยกเลิกคำขอปฏิเสธได้จนกว่า Approver จะตัดสิน (`data-rejcancel`)
    - Approver อนุมัติคำขอ → `REJECTED` (`data-rejapprove`) หรือไม่อนุมัติ → กลับคิว Checker (`data-rejdecline`)
  - Approver: Return → `RETURNED`, Reject → `REJECTED` ทันที, Approve → `COMPLETED`
  - Internal note ของการตีกลับ แสดงเฉพาะเจ้าหน้าที่ (`retNotes()`, `intNote()`, `isStaff()`)
    ทุนครบ 100% ต้อง E-Sign (MFA ซ้ำ) และออกใบรับรอง ยังไม่ครบไม่ต้อง E-Sign และไม่ออกใบรับรอง (`approve()`)
  - ยื่นใหม่หลังถูกตีกลับ กลับเข้าคิว Checker ตามสถานะเดิม (`PENDING` หรือ `APPRAISED_COMPLETED`)
  - Approver จะไม่เห็นงานจนกว่า Checker จะ Approve
- **สถานะวิสาหกิจ**
  - ชำระครบแล้วเป็น `COMPLETED` ถ้ายังไม่ครบเป็น `PARTIAL` (ดู `approve()`)
  - เกินวันครบกำหนดเป็น `OVERDUE` และมีค่าปรับ (ดู `penOf()`)
- **ใบรับรอง**: ออกได้เมื่อทุนชำระครบเท่านั้น
- **Create Account** (handler `#doSignup`, `#doSuOtp`, `#doSuCreate` ใน `bind()`)
  - Step 1 Applicant details: ทุกช่องต้องกรอก (Email, Citizen ID หรือ Passport, Role applied for,
    ชื่อลาว และชื่ออังกฤษ อย่างละ คำนำหน้า / ชื่อ / นามสกุล), อีเมลต้องถูกรูปแบบ, เลขบัตรอย่างน้อย 6 ตัว,
    ชื่ออังกฤษใช้ได้แค่ A-Z
  - Step 2 Verify email: OTP 6 หลักที่ส่งไปอีเมล ปุ่ม Next กดได้เมื่อกรอกครบ มี Resend code
  - Step 3 Documents เปลี่ยนตาม role (`suDocsFor()`)
    - owner: ต้องกรอก Enterprise registration number + Enterprise type แล้วอัปโหลดเอกสารตามประเภท
      (`SU_ETYPES` / `SU_DOC`, บางรายการไม่บังคับ)
    - role อื่น: ไม่มี Enterprise details อัปโหลดบัตรประชาชน (หรือ Passport ตาม step 1) + บัตรพนักงาน
    - รับเฉพาะ PDF, JPG, PNG ไม่เกิน 10 MB ต่อไฟล์ (`suFileErr()`) ต้องตรวจซ้ำฝั่ง backend (ทั้งนามสกุลและ MIME)
  - ส่งแล้วเป็นสถานะรอตรวจเอกสาร เมื่อเอกสารผ่านจึงส่งรหัสผ่านชั่วคราวไปที่อีเมล
  - Role ที่สมัครยังไม่ได้ผูกกับ role ในระบบ (`Role`) ต้องตกลงกับลูกค้าว่า Bank Maker / Checker,
    MOIC Maker ฯลฯ ตรงกับ role ไหน
- **รหัสผ่านชั่วคราว**: บัญชีที่ใช้รหัสผ่านชั่วคราวต้องตั้งรหัสใหม่หลัง OTP โดยปิด modal ไม่ได้
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
| `Modal` + ชนิดต่างๆ | `vModal()` | `approve`, `verify`, `backreason`, `chgpay`, `receipt`, `qr`, `valjob`, `adduser`, `forgot`, `done`, `help`, `keys`, `clearcache` |
| `PaymentBlock` (แก้ไข / อ่านอย่างเดียว) | `payCardEdit()`, `payCardView()` | หัวการ์ด `payHead()`, ย่อ/ขยายด้วย `tglBtn()` |
| `PaymentSummary` | `paySummary()` | ยอดแยกตามประเภท + Total this instalment + Outstanding after |
| `EnterpriseBrief` | `entBrief()` | 4 ช่องย่อ + Show details (ไม่มีแผนที่) ใช้ในหน้า Declare และ Task |
| `OtpInput` | `otpWire()` | 6 ช่อง, เลื่อนช่องอัตโนมัติ, Backspace ย้อนกลับ (ใช้ทั้ง login และ Verify email) |
| `StepBar` | `stepBar` ใน `vLogin()` | แถบ 3 ขั้นของ Create Account (ขั้นที่ผ่านแล้วเป็น ✓ สีเขียว) |
| `FileDrop` | `docField` ใน `vLogin()` | กดเลือกหรือลากวาง, ตรวจชนิด/ขนาดไฟล์, แสดงชื่อ + ขนาด + ปุ่มลบ |
| `Toast` | `toast()` | แสดงกลางล่าง |
| `MapView` | Leaflet ใน `vDetail` | ต้องรันฝั่ง client เท่านั้น |
| `WorkSplit` layout | หน้า Transaction | ซ้ายข้อมูลอ้างอิง ขวาคำร้อง + เอกสารแบบ sticky (หน้า Declare, Task, Valuation job เป็นแบบเต็มความกว้าง `vstack`) |

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
- รหัสผ่าน / OTP อะไรก็ผ่าน (รวม OTP ยืนยันอีเมลตอน Create Account)
- Create Account ข้ามขั้นรอตรวจเอกสาร: บัญชี demo `new.user` เปลี่ยนเป็นผู้สมัครทันทีและ log in ได้เลย
  ระบบจริงต้องมีหน้าหลังบ้านให้เจ้าหน้าที่ตรวจเอกสารแล้วอนุมัติ / ปฏิเสธ (ยังไม่มีใน demo)
- วันที่ล็อก 04/08/2026
- mobile preview ของ owner (`Alt+M`, `vMobile`)
- โค้ดที่ไม่มีทางเข้า: `vStaffDash`, `vQueue` (view `inspect` / `approveQ`)

คีย์ลัดที่ควรพิจารณาเก็บไว้: `Alt+L` สลับภาษา, `Alt+B` ย่อ sidebar, `Alt+/` ดูคีย์ลัด

## 9. API ที่ frontend ต้องการ (ร่าง)

```
GET    /me/scope                   -> AccessScope (มาจากระบบบริหารสิทธิ์)
GET    /dashboard/summary          ?prov=&dist=  -> KPI ของ Dashboard กลาง (กรองตามขอบเขตที่ฝั่ง server)
GET    /dashboard/breakdown        ?prov=&dist=  -> แถวตามแขวง หรือตามเมือง
GET    /dashboard/attention        ?prov=&dist=&page=&size=
POST   /auth/login                 { user, password } -> { challengeId }
POST   /auth/otp                   { challengeId, otp } -> session (+ mustChangePassword)
POST   /auth/password              { newPassword }
POST   /auth/forgot                { email }
POST   /auth/signup                { email, idType, idNo, role, nameLo, nameEn } -> { applicationId } (ส่ง OTP ไปอีเมล)
POST   /auth/signup/:id/otp        { otp }  (ยืนยันอีเมล)
POST   /auth/signup/:id/otp/resend
POST   /auth/signup/:id/documents  multipart: k, file  (PDF/JPG/PNG ≤10 MB)
POST   /auth/signup/:id/submit     { regNo?, entityType? } -> status 'pending'
GET    /admin/signup-applications  ?status=&role=&page=&size=
POST   /admin/signup-applications/:id/approve   (ส่งรหัสผ่านชั่วคราวทางอีเมล)
POST   /admin/signup-applications/:id/reject    { reason }

GET    /enterprises                ?name=&regNo=&prov=&type=&isic=&regFrom=&regTo=&updFrom=&updTo=&status=&page=&size=
GET    /enterprises/:id
GET    /me/enterprises             (owner)
POST   /enterprises/:id/declarations   { plan, payments: Payment[] } + ไฟล์ของแต่ละรายการ (multipart)

GET    /declarations               ?status=PENDING,APPRAISED_COMPLETED&ref=&ent=&regNo=&channel=&by=&page=&size=
GET    /declarations/:ref
PUT    /declarations/:ref          (แก้ฉบับร่าง หรือใบรายการที่ถูกตีกลับ)
DELETE /declarations/:ref          (ลบได้เฉพาะ DRAFT)
POST   /declarations/:ref/submit   (DRAFT / RETURNED -> PENDING / APPRAISED_PENDING / APPRAISED_COMPLETED)
POST   /declarations/:ref/verify   (checker: -> APPROVED)
POST   /declarations/:ref/return   { reason, internal, external } (appraiser / checker / approver -> RETURNED)
POST   /declarations/:ref/reject   { reason } (approver -> REJECTED; checker -> คำขอปฏิเสธ รอ approver)
POST   /declarations/:ref/reject/cancel    (checker ยกเลิกคำขอปฏิเสธ)
POST   /declarations/:ref/reject/approve   (approver อนุมัติคำขอ -> REJECTED)
POST   /declarations/:ref/reject/decline   (approver ไม่อนุมัติ -> กลับคิว checker)
POST   /declarations/:ref/approve  { otp? } (approver -> COMPLETED; otp เมื่อทุนครบ 100% เพื่อ E-Sign)

GET    /valuation-jobs             ?ref=&ent=&regNo=&firm=&net=&status=&page=&size=
GET    /valuation-jobs/:id
POST   /valuation-jobs/:id/result  multipart: value, remark, files[] (แต่ละไฟล์ส่ง param_code ของเอกสารคู่กัน)

GET    /invoices                   ?kind=fee|penalty&...
GET    /receipts
GET    /reports/:type              ?from=&to=
GET    /certificates/:entId        (PDF)

GET    /lookups/{provinces,isic,entity-types,valuation-firms,doc-types,staff?role=,signup-roles}
                                   (entity-types ควรคืนรายการเอกสารที่ต้องอัปโหลดของแต่ละประเภทด้วย)
GET    /admin/{users,gl,api-log,audit,fees,master}
```

- list endpoint ทุกตัวควรคืน `{ items, total, page, size }`
- frontend จะได้ไม่ต้องดึงทั้งหมดมากรองเองแบบ demo

## 10. ลำดับงานที่แนะนำ

1. **Foundation**
   - ตั้งโปรเจกต์ ย้าย `tokens.css` ฟอนต์ และ i18n (lo / en)
   - ทำ mock API จาก seed data
2. **Shell + Auth**
   - login, OTP, forced password change, Create Account (3 ขั้น + อัปโหลดเอกสาร)
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
