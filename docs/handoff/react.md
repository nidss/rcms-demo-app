# Handoff: React SPA (Vite + React Router)

อ่าน [`README.md`](./README.md) ก่อน ไฟล์นี้อธิบายเฉพาะส่วนที่ขึ้นกับ React

## 1. Stack ที่แนะนำ

| เรื่อง | เลือก | เหตุผล |
|---|---|---|
| Build | Vite + React + TypeScript | repo ใช้ Vite อยู่แล้ว |
| Routing | React Router (data router: `createBrowserRouter`) | รองรับ loader / nested layout / guard |
| Server state | TanStack Query | cache, loading, refetch หลัง approve / return |
| ฟอร์ม | react-hook-form + zod | ใส่ business rule (30%, ไม่เกินคงเหลือ, ต้องแนบไฟล์) เป็น schema |
| i18n | i18next + react-i18next | ใช้ `locales/lo.json`, `locales/en.json` ที่ export จาก `D` |
| CSS | CSS Modules + `tokens.css` (CSS variables) | ย้าย CSS จาก demo ได้เกือบตรงตัว |
| ไอคอน | `lucide-react` | ชื่อเดียวกับที่ demo ใช้ (`Search`, `BrushCleaning`, ...) |
| แผนที่ | `react-leaflet` | demo ใช้ Leaflet อยู่แล้ว |
| Mock API | MSW (Mock Service Worker) | ใช้ seed data จาก demo เป็น fixture และสลับไป API จริงได้โดยไม่แก้ component |
| ตาราง | เขียนเอง หรือ TanStack Table | ตารางใน demo เรียบง่าย เขียนเองก็พอ |

ถ้าทีมถนัด Tailwind ให้ map token ใน `tailwind.config` (`colors.m`, `colors.ok`, ...) แทน CSS Modules

## 2. โครงสร้างโฟลเดอร์

```
src/
  main.tsx                 # QueryClientProvider, I18nextProvider, RouterProvider
  router.tsx               # route ทั้งหมด + guard ตาม role
  styles/
    tokens.css             # :root{...} จาก demo
    base.css               # reset, font, .num
  i18n/
    index.ts
    locales/lo.json
    locales/en.json
  api/
    client.ts              # fetch wrapper + auth header
    enterprises.ts         # useEnterprises(), useEnterprise(id)
    declarations.ts
    valuation.ts
    ...
  mocks/                   # MSW handlers + fixtures (จาก seed data)
  auth/
    AuthProvider.tsx       # user, role, mustChangePassword
    RequireRole.tsx
  components/
    shell/AppShell.tsx, Sidebar.tsx, Topbar.tsx, NotificationBell.tsx, UserMenu.tsx
    list/FilterBar.tsx, DataTable.tsx, Pager.tsx, ListCard.tsx
    SummaryCard.tsx, Metric.tsx, StatusPill.tsx, DocRow.tsx, Donut.tsx, OtpInput.tsx
    modal/Modal.tsx, ...
  features/
    auth/LoginPage.tsx, CreateAccountPage.tsx, ChangePasswordModal.tsx
    owner/MyCapitalList.tsx, CapitalDetails.tsx
    maker/MakerWorkspace.tsx
    checker/CheckerWorkspace.tsx
    approver/ApproverWorkspace.tsx
    enterprises/EnterpriseList.tsx, EnterpriseDetail.tsx, DeclarePayment.tsx, Certificate.tsx
    tasks/TaskReview.tsx
    valuation/ValuationQueue.tsx, MyValuationJobs.tsx, ValuationJobDetail.tsx
    finance/FinancePage.tsx
    reports/ReportsPage.tsx
    settings/SettingsPage.tsx
    portal/ExternalPortal.tsx
    admin/AdminOverview.tsx
  config/menus.ts          # แปลงจาก menus() ใน demo
```

## 3. Mapping จาก demo ไป React

| Demo | React |
|---|---|
| `S` (global state) + `render()` | แยกตามชนิด: server state → TanStack Query, ตัวกรอง/หน้า → URL search params, UI ชั่วคราว → `useState` |
| `S.view`, `S.dashTab`, `S.finTab`, `S.setTab` | route path (`/workspace/:tab`, `/finance/:tab`, `/settings/:tab`) |
| `S.colf[key]` (ตัวกรองที่กดค้นหาแล้ว) | URL search params (`?ent=...&regNo=...`) |
| `S.colDraft[key]` (ที่กำลังพิมพ์) | state ภายใน `FilterBar` |
| `S.pg[key]` | `?page=&size=` ใน URL |
| `S.modal` | state ของหน้า หรือ modal store เล็กๆ (Zustand / context) |
| `S.role`, `ACCTS` | `AuthProvider` (ข้อมูลจาก session) |
| `bind()` + `data-*` | `onClick` / `onChange` ใน JSX |
| `t('key')` | `const { t } = useTranslation(); t('key')` |
| `L({lo,en})` | hook `useL()` คืน `obj[i18n.language]` |
| `esc()` | ไม่ต้องใช้ React escape ให้เอง (ห้ามใช้ `dangerouslySetInnerHTML` กับข้อมูลผู้ใช้) |
| `luSvg('search')` | `<Search size={16} />` จาก `lucide-react` |
| `toast()` | ไลบรารี toast (sonner / react-hot-toast) หรือ component เอง |

## 4. ตัวอย่างโค้ดหลัก

### 4.1 Hook ตัวกรอง + หน้า ผูกกับ URL

ทำให้กด back / refresh แล้วได้ผลเดิม และเปลี่ยนตัวกรองแล้วกลับหน้า 1 เหมือน `paginate()`

```tsx
// src/components/list/useListParams.ts
import { useSearchParams } from 'react-router-dom';

export function useListParams(fields: string[]) {
  const [sp, setSp] = useSearchParams();
  const applied = Object.fromEntries(fields.map(f => [f, sp.get(f) ?? '']));
  const page = Number(sp.get('page') ?? 1);
  const size = Number(sp.get('size') ?? 10);

  function apply(values: Record<string, string>) {
    const next = new URLSearchParams();
    Object.entries(values).forEach(([k, v]) => { if (v.trim()) next.set(k, v.trim()); });
    if (size !== 10) next.set('size', String(size));
    setSp(next);                         // ไม่ใส่ page = กลับหน้า 1
  }
  function setPage(p: number) { sp.set('page', String(p)); setSp(sp); }
  function setSize(n: number) { sp.set('size', String(n)); sp.delete('page'); setSp(sp); }

  return { applied, page, size, apply, clear: () => apply({}), setPage, setSize };
}
```

### 4.2 FilterBar (draft แยกจาก applied)

```tsx
// src/components/list/FilterBar.tsx
import { useEffect, useState } from 'react';
import { Search, BrushCleaning } from 'lucide-react';
import { useTranslation } from 'react-i18next';

export type FieldSpec = {
  name: string;
  label: string;
  options?: [string, string][];   // มี = dropdown (จับคู่ตรงทั้งค่า)
  type?: 'text' | 'date';
};

export function FilterBar({ fields, applied, onSearch, onClear }: {
  fields: FieldSpec[];
  applied: Record<string, string>;
  onSearch: (v: Record<string, string>) => void;
  onClear: () => void;
}) {
  const { t } = useTranslation();
  const [draft, setDraft] = useState(applied);
  useEffect(() => setDraft(applied), [JSON.stringify(applied)]);

  return (
    <form className="fbar" onSubmit={e => { e.preventDefault(); onSearch(draft); }}>
      <div className="frow">
        {fields.map(f => (
          <div className="field" key={f.name}>
            <label htmlFor={`cf-${f.name}`}>{f.label}</label>
            {f.options ? (
              <select id={`cf-${f.name}`} value={draft[f.name] ?? ''}
                onChange={e => setDraft({ ...draft, [f.name]: e.target.value })}>
                <option value="">{t('finAll')}</option>
                {f.options.map(([v, l]) => <option key={v} value={v}>{l}</option>)}
              </select>
            ) : (
              <input id={`cf-${f.name}`} type={f.type ?? 'text'} value={draft[f.name] ?? ''}
                onChange={e => setDraft({ ...draft, [f.name]: e.target.value })} />
            )}
          </div>
        ))}
        <div className="fx">
          <button className="btn" type="submit"><Search size={16} />{t('searchBtn')}</button>
          <button className="btn ghost" type="button" onClick={() => { setDraft({}); onClear(); }}>
            <BrushCleaning size={16} />{t('clearBtn')}
          </button>
        </div>
      </div>
    </form>
  );
}
```

- ใช้ `<form onSubmit>` ทำให้กด Enter แล้วค้นหาได้เหมือน demo โดยไม่ต้องเขียน keydown เอง

### 4.3 ใช้งานในหน้า (ตัวอย่าง Checker: Verification queue)

```tsx
// src/features/checker/VerificationQueue.tsx
const FIELDS = ['ref', 'ent', 'regNo', 'channel', 'by'];

export function VerificationQueue() {
  const { t } = useTranslation();
  const lp = useListParams(FIELDS);
  const q = useDeclarations({ stage: 'checker', ...lp.applied, page: lp.page, size: lp.size });
  const staff = useStaffOptions('maker');          // dropdown Submit by

  return (
    <>
      <FilterBar applied={lp.applied} onSearch={lp.apply} onClear={lp.clear} fields={[
        { name: 'ref', label: t('cRef') },
        { name: 'ent', label: t('cEnt') },
        { name: 'regNo', label: t('regNo') },
        { name: 'channel', label: t('cChannel'), options: [['counter', t('chCounter')], ['qr', t('chQR')], ['mobile', t('chMobile')]] },
        { name: 'by', label: t('cBy'), options: staff.data ?? [] },
      ]} />
      <ListCard title={t('ckMQueue')} query={q} page={lp.page} size={lp.size}
        onPage={lp.setPage} onSize={lp.setSize}
        columns={[ /* Reference, Enterprise, Registration no., Amount, Channel, Submit by, ... */ ]} />
    </>
  );
}
```

- ชื่อ key ภาษา (`cRef`, `cEnt`, `regNo`, ...) ให้เปิดดูใน `D` ของ demo ก่อน แล้วใช้ชื่อเดิม
- ถ้า backend ยังไม่รองรับการกรอง ให้ทำ `filterRows()` ฝั่ง client ตามกติกาใน `listMatch()` ชั่วคราว
  - ช่องข้อความ: substring ไม่สนตัวพิมพ์และ `,`
  - dropdown: ตรงทั้งค่า

### 4.4 Route + guard ตาม role

```tsx
// src/router.tsx
export const router = createBrowserRouter([
  { path: '/login', element: <LoginPage /> },
  { path: '/create-account', element: <CreateAccountPage /> },
  {
    element: <RequireAuth><AppShell /></RequireAuth>,
    children: [
      { index: true, element: <RoleHome /> },     // redirect ตาม roleHome()
      { path: 'my-capital', element: <RequireRole roles={['owner']}><MyCapitalList /></RequireRole> },
      { path: 'my-capital/:id', element: <RequireRole roles={['owner']}><CapitalDetails /></RequireRole> },
      { path: 'workspace/:tab', element: <RequireRole roles={['maker','checker','approver']}><Workspace /></RequireRole> },
      { path: 'enterprises', element: <RequireRole roles={['maker','checker','approver','admin']}><EnterpriseList /></RequireRole> },
      { path: 'enterprises/:id', element: <EnterpriseDetail /> },
      { path: 'enterprises/:id/declare', element: <DeclarePayment /> },
      { path: 'tasks/:ref', element: <RequireRole roles={['checker','approver']}><TaskReview /></RequireRole> },
      { path: 'finance/:tab', element: <RequireRole roles={['checker','approver']}><FinancePage /></RequireRole> },
      { path: 'settings/:tab', element: <RequireRole roles={['admin']}><SettingsPage /></RequireRole> },
      // ... ตามตารางใน README.md ข้อ 4
    ],
  },
]);
```

- `Workspace` เลือก component ตาม role (เหมือน `vDash()`)
- เมนูย่อยใต้ Dashboard เป็น `NavLink` ไป `/workspace/ret`, `/workspace/queue`, ...
- ตัวเลข badge ข้างเมนูย่อยมาจาก endpoint นับจำนวน (เช่น `GET /me/counters`)

### 4.5 Login flow

```
/login (user + password) -> OTP -> ถ้า mustChangePassword: ChangePasswordModal (ปิดไม่ได้, Escape ไม่ปิด) -> RoleHome
/create-account
  1 Applicant details -> 2 Verify email (OTP) -> 3 Documents (ตาม role)
  -> หน้า "Application submitted" (รอตรวจเอกสาร ผ่านแล้วส่งรหัสผ่านชั่วคราวทางอีเมล) -> /login?user=<email>
```

- `CreateAccountPage`: เก็บข้อมูลทั้ง 3 ขั้นใน state เดียว (หรือ `useReducer`) กด Back แล้วข้อมูลไม่หาย
  - ขั้น 3 ใช้ `FileDrop` (`<input type="file" accept=".pdf,.jpg,.jpeg,.png">` + drag and drop) ตรวจชนิดและขนาด ≤10 MB ก่อนอัปโหลด
  - เปลี่ยน Enterprise type แล้วล้างไฟล์ที่เลือกไว้ เพราะรายการเอกสารต่างกัน

- `OtpInput`: 6 ช่อง, `inputMode="numeric"`, พิมพ์แล้วเลื่อนไปช่องถัดไป, Backspace ในช่องว่างย้อนกลับ, รองรับ paste ทั้ง 6 หลัก
- เก็บ token ใน httpOnly cookie (ให้ backend ตั้ง) ดีกว่า `localStorage`

### 4.6 i18n

```ts
// src/i18n/index.ts
i18n.use(initReactI18next).init({
  lng: localStorage.getItem('lang') ?? 'lo',
  fallbackLng: 'en',                         // เหมือน t() ใน demo: ไม่เจอใน lo ใช้ en
  resources: { lo: { translation: lo }, en: { translation: en } },
  interpolation: { escapeValue: false },
});
i18n.on('languageChanged', l => {
  document.documentElement.lang = l;
  document.body.classList.toggle('lo', l === 'lo');   // CSS เลือก font ตาม class นี้
  localStorage.setItem('lang', l);
});
```

### 4.7 แผนที่

```tsx
import { MapContainer, TileLayer, Marker } from 'react-leaflet';
import 'leaflet/dist/leaflet.css';
import L from 'leaflet';
import icon from 'leaflet/dist/images/marker-icon.png';
import icon2x from 'leaflet/dist/images/marker-icon-2x.png';
import shadow from 'leaflet/dist/images/marker-shadow.png';
// bundler เปลี่ยน path รูป marker ทำให้ icon ปกติหาย ต้องตั้งเอง
L.Icon.Default.mergeOptions({ iconUrl: icon, iconRetinaUrl: icon2x, shadowUrl: shadow });

// demo: mountRegMap() ใช้ OSM tile, zoom 15, marker ที่ lat/lng ของวิสาหกิจ
```

## 5. Deploy

- `npm run build` ได้ `dist/` เป็นไฟล์ static วางบน Nginx / S3 / server ภายในได้
- **ต้องตั้ง fallback ให้ทุก path ส่ง `index.html`** (SPA routing) เช่น Nginx:
  ```
  location / { try_files $uri /index.html; }
  ```
- ถ้าจะวางบน GitHub Pages ต่อ ให้ใช้ `createHashRouter` หรือ copy `index.html` เป็น `404.html`
- ไม่ต้องใช้ `vite-plugin-singlefile` แล้ว (ใช้แค่ใน demo เพื่อให้เปิดไฟล์เดียวได้)

## 6. Checklist ส่งงาน

- [ ] ทุกหน้าในตาราง route (README ข้อ 4) เปิดได้ และ guard ตาม role ถูก
- [ ] ทุกรายการมี FilterBar (Search / Clear), กรองเมื่อกด Search หรือ Enter เท่านั้น
- [ ] เปลี่ยนตัวกรองแล้วกลับหน้า 1, เลือกได้ 5/10/20/50 แถว
- [ ] Enterprise / Registration no. แยกคอลัมน์และแยกช่องค้นหา
- [ ] Business rule ครบตาม README ข้อ 5 และ error message ตรงกับ demo
- [ ] ภาษาลาว / อังกฤษครบทุกข้อความ, font ถูกตามภาษา
- [ ] Responsive ตาม breakpoint ใน README ข้อ 7
- [ ] เทียบหน้าจอกับ demo ทีละหน้า
