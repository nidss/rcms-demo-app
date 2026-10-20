# Handoff: Next.js (App Router)

อ่าน [`README.md`](./README.md) ก่อน ไฟล์นี้อธิบายเฉพาะส่วนที่ขึ้นกับ Next.js
- component ส่วนใหญ่ (FilterBar, ListCard, SummaryCard, OtpInput ...) เขียนแบบเดียวกับ [`react.md`](./react.md)
- ไฟล์นี้เน้นส่วนที่ต่าง: server / client component, routing, auth, i18n และ deploy

## 1. Stack ที่แนะนำ

| เรื่อง | เลือก | เหตุผล |
|---|---|---|
| Framework | Next.js (App Router) + TypeScript | layout ซ้อนกันได้ตรงกับ AppShell / เมนูย่อย |
| Data | Server Components เรียก backend ตรง, TanStack Query เฉพาะส่วนที่ต้อง refetch ฝั่ง client | หน้ารายการ render ฝั่ง server จาก `searchParams` |
| Mutation | Server Actions หรือ Route Handlers (`app/api/...`) เป็น BFF | ซ่อน token ของ backend ไว้ฝั่ง server |
| Auth | Auth.js (NextAuth) หรือ session cookie ที่ backend ออกให้ + `middleware.ts` | กัน route ตาม role ก่อน render |
| i18n | `next-intl` | ใช้ `messages/lo.json`, `messages/en.json` ที่ export จาก `D` |
| ฟอร์ม | react-hook-form + zod (schema เดียวใช้ทั้ง client และ Server Action) | business rule ไม่ต้องเขียนซ้ำ |
| CSS | CSS Modules + `tokens.css` ใน `app/globals.css` | |
| Font | `next/font/google` (Noto Sans Lao, IBM Plex Sans, IBM Plex Mono) | self-host อัตโนมัติ ไม่ต้องโหลดจาก Google ตอนใช้งาน |
| ไอคอน | `lucide-react` | |
| แผนที่ | `react-leaflet` ผ่าน `next/dynamic` แบบ `ssr: false` | Leaflet ใช้ `window` |

## 2. โครงสร้างโฟลเดอร์

```
app/
  layout.tsx                      # <html lang>, font, NextIntlClientProvider
  globals.css                     # tokens.css + base
  (auth)/
    login/page.tsx
    create-account/page.tsx       # 3 ขั้นเป็น client component เดียว
  (app)/
    layout.tsx                    # AppShell (Sidebar + Topbar) อ่าน session ฝั่ง server
    page.tsx                      # redirect ตาม roleHome()
    my-capital/page.tsx
    my-capital/[id]/page.tsx
    workspace/[tab]/page.tsx      # เลือก Maker / Checker / Approver ตาม role
    overview/page.tsx             # admin
    enterprises/page.tsx
    enterprises/[id]/page.tsx
    enterprises/[id]/declare/page.tsx
    enterprises/[id]/certificate/page.tsx
    transactions/[id]/page.tsx
    tasks/[ref]/page.tsx
    valuation-queue/page.tsx
    valuation-jobs/page.tsx
    valuation-jobs/[id]/page.tsx
    finance/[tab]/page.tsx
    reports/page.tsx
    settings/[tab]/page.tsx
    portal/page.tsx
  api/                            # (ถ้าต้องการ BFF แบบ REST)
middleware.ts                     # ตรวจ session + role ต่อ path
lib/
  api.ts                          # เรียก backend จริง (server only)
  auth.ts
  menus.ts                        # แปลงจาก menus() ใน demo
  rules.ts                        # zod schema: 30%, ไม่เกินคงเหลือ, ต้องแนบไฟล์
components/                       # เหมือน react.md
messages/lo.json, messages/en.json
mocks/                            # fixture จาก seed data (ใช้ใน lib/api.ts ตอนยังไม่มี backend)
```

- route group `(auth)` กับ `(app)` แยก layout: หน้า login ไม่มี sidebar
- `[tab]` ใช้แทน `S.dashTab` / `S.finTab` / `S.setTab` ของ demo

## 3. Server หรือ Client component

| ส่วน | ชนิด | เหตุผล |
|---|---|---|
| `(app)/layout.tsx` (AppShell) | Server | อ่าน session, สร้างเมนูตาม role, ดึงตัวเลข badge |
| Sidebar ย่อ / drawer, NotificationBell, UserMenu, LangSwitch | Client | มี state เปิด/ปิด |
| หน้ารายการ (`page.tsx`) | Server | อ่าน `searchParams` แล้วดึงข้อมูลหน้านั้นจาก backend |
| `FilterBar`, `Pager` | Client | draft ของตัวกรอง และเปลี่ยน URL ด้วย `useRouter` |
| ตาราง | Server (render แถว) | ไม่ต้องส่ง JS ของตารางไปที่ browser |
| Modal (approve, verify, backreason, ...) | Client | |
| ฟอร์ม Declare / Submit valuation | Client + Server Action | validate ทั้งสองฝั่งด้วย schema เดียว |
| แผนที่ | Client (`dynamic(..., { ssr: false })`) | |

## 4. ตัวอย่างโค้ดหลัก

### 4.1 หน้ารายการแบบ server-side filter + paging

```tsx
// app/(app)/workspace/[tab]/page.tsx (ส่วน Checker: Verification queue)
import { getTranslations } from 'next-intl/server';
import { getDeclarations, getStaffOptions } from '@/lib/api';
import { FilterBar } from '@/components/list/FilterBar';
import { ListCard } from '@/components/list/ListCard';

export default async function Page({ params, searchParams }: {
  params: Promise<{ tab: string }>;
  searchParams: Promise<Record<string, string | undefined>>;
}) {
  const { tab } = await params;
  const sp = await searchParams;
  const t = await getTranslations();
  const page = Number(sp.page ?? 1), size = Number(sp.size ?? 10);

  const [data, makers] = await Promise.all([
    getDeclarations({ stage: 'checker', ref: sp.ref, ent: sp.ent, regNo: sp.regNo,
                      channel: sp.channel, by: sp.by, page, size }),
    getStaffOptions('maker'),
  ]);

  return (
    <>
      <FilterBar fields={[
        { name: 'ref', label: t('cRef') },
        { name: 'ent', label: t('cEnt') },
        { name: 'regNo', label: t('regNo') },
        { name: 'channel', label: t('cChannel'), options: [['counter', t('chCounter')], ['qr', t('chQR')], ['mobile', t('chMobile')]] },
        { name: 'by', label: t('cBy'), options: makers },
      ]} />
      <ListCard title={t('ckMQueue')} data={data} page={page} size={size} />
    </>
  );
}
```

- Next.js 15 ขึ้นไป `params` / `searchParams` เป็น Promise ต้อง `await`

### 4.2 FilterBar ฝั่ง client เขียน URL

```tsx
// components/list/FilterBar.tsx
'use client';
import { useRouter, usePathname, useSearchParams } from 'next/navigation';
import { useState } from 'react';
import { Search, BrushCleaning } from 'lucide-react';
import { useTranslations } from 'next-intl';

export function FilterBar({ fields }: { fields: FieldSpec[] }) {
  const t = useTranslations();
  const router = useRouter(), pathname = usePathname(), sp = useSearchParams();
  const [draft, setDraft] = useState<Record<string, string>>(
    Object.fromEntries(fields.map(f => [f.name, sp.get(f.name) ?? ''])));

  function push(values: Record<string, string>) {
    const next = new URLSearchParams();
    Object.entries(values).forEach(([k, v]) => { if (v.trim()) next.set(k, v.trim()); });
    const size = sp.get('size'); if (size) next.set('size', size);   // ไม่ใส่ page = กลับหน้า 1
    router.push(`${pathname}?${next}`);
  }

  return (
    <form className="fbar" onSubmit={e => { e.preventDefault(); push(draft); }}>
      {/* ช่องกรอกเหมือน react.md ข้อ 4.2 */}
      <div className="fx">
        <button className="btn" type="submit"><Search size={16} />{t('searchBtn')}</button>
        <button className="btn ghost" type="button" onClick={() => { setDraft({}); push({}); }}>
          <BrushCleaning size={16} />{t('clearBtn')}
        </button>
      </div>
    </form>
  );
}
```

- `Pager` ทำแบบเดียวกัน: เป็น `<Link href={?page=n}>` ก็พอ ไม่ต้องเป็น client component

### 4.3 Middleware กัน route ตาม role

```ts
// middleware.ts
import { NextResponse, type NextRequest } from 'next/server';

const ACCESS: [RegExp, string[]][] = [
  [/^\/my-capital/, ['owner']],
  [/^\/workspace/, ['maker', 'checker', 'approver']],
  [/^\/overview/, ['admin']],
  [/^\/enterprises/, ['maker', 'checker', 'approver', 'admin', 'owner']],
  [/^\/tasks/, ['checker', 'approver']],
  [/^\/valuation-queue/, ['checker', 'approver']],
  [/^\/valuation-jobs/, ['valfirm']],
  [/^\/finance/, ['checker', 'approver']],
  [/^\/reports/, ['checker', 'approver', 'admin']],
  [/^\/settings/, ['admin']],
  [/^\/portal/, ['external']],
];

export function middleware(req: NextRequest) {
  const session = readSession(req);                 // ถอด cookie (ขึ้นกับระบบ auth ที่เลือก)
  const { pathname } = req.nextUrl;
  if (!session) return NextResponse.redirect(new URL(`/login?next=${pathname}`, req.url));
  if (session.mustChangePassword && pathname !== '/login') return NextResponse.redirect(new URL('/login?step=new', req.url));
  const rule = ACCESS.find(([re]) => re.test(pathname));
  if (rule && !rule[1].includes(session.role)) return NextResponse.redirect(new URL('/', req.url));
  return NextResponse.next();
}
export const config = { matcher: ['/((?!login|create-account|api|_next|favicon.ico).*)'] };
```

- middleware เป็นแค่ด่านแรก backend ต้องตรวจสิทธิ์ซ้ำทุก request
- owner เข้า `/enterprises/:id` ได้เฉพาะวิสาหกิจของตัวเอง (backend ต้องเช็ก)

### 4.4 Server Action สำหรับ Declare payment

```ts
// app/(app)/enterprises/[id]/declare/actions.ts
'use server';
import { declareSchema } from '@/lib/rules';

export async function declarePayment(entId: string, formData: FormData) {
  const ent = await getEnterprise(entId);
  const parsed = declareSchema(ent).safeParse(Object.fromEntries(formData));
  if (!parsed.success) return { error: parsed.error.flatten() };
  await api.post(`/enterprises/${entId}/declarations`, formData);
  revalidatePath(`/enterprises/${entId}`);
  redirect(`/enterprises/${entId}`);
}
```

```ts
// lib/rules.ts: schema เดียวใช้ทั้ง client และ server
export const declareSchema = (ent: { cap: number; paid: number; insts: unknown[] }) =>
  z.object({
    amt: z.coerce.number().positive('errAmt'),
    payMode: z.enum(['full', 'partial']),
    files: z.array(z.any()).min(1, 'errFile'),
  }).refine(v => v.amt <= ent.cap - ent.paid, { message: 'errOver', path: ['amt'] })
    .refine(v => v.payMode === 'full' || ent.paid === 0, { message: 'errPartial', path: ['payMode'] })
    .refine(v => v.payMode === 'full' || v.amt >= Math.round(ent.cap * 0.3), { message: 'errMin30', path: ['amt'] });
```

- message เป็น key ของ i18n: `errAmt`, `errOver`, `errFile` มีอยู่แล้วใน `D` ของ demo
- `errPartial`, `errMin30` ยังไม่มี (demo ไม่ได้ตรวจกฎ 30% ตอนส่ง) ต้องเพิ่มข้อความทั้ง 2 ภาษา
- เพิ่มกฎชำระเป็นทรัพย์สินและกฎคำร้องค้างตาม README ข้อ 5

### 4.5 i18n (`next-intl`)

มีสองแบบ เลือกแบบเดียว
- **เก็บภาษาใน cookie** (แนะนำสำหรับระบบภายใน)
  - URL ไม่เปลี่ยน ใช้ `getRequestConfig` อ่าน cookie `lang`
  - ค่าเริ่มต้น `lo`
- **เก็บภาษาใน path** (`/lo/...`, `/en/...`)
  - เหมาะถ้ามีหน้า public ที่ต้องติด search engine

```tsx
// app/layout.tsx
import { Noto_Sans_Lao, IBM_Plex_Sans, IBM_Plex_Mono } from 'next/font/google';
const lao = Noto_Sans_Lao({ subsets: ['lao'], weight: ['400','500','600','700'], variable: '--f-lao' });
const plex = IBM_Plex_Sans({ subsets: ['latin'], weight: ['400','500','600','700'], variable: '--f-plex' });
const mono = IBM_Plex_Mono({ subsets: ['latin'], weight: ['400','500','600'], variable: '--f-mono' });

export default async function RootLayout({ children }: { children: React.ReactNode }) {
  const locale = await getLocale();
  return (
    <html lang={locale} className={`${lao.variable} ${plex.variable} ${mono.variable}`}>
      <body className={locale === 'lo' ? 'lo' : ''}>{/* NextIntlClientProvider */}{children}</body>
    </html>
  );
}
```

- แก้ `font-family` ใน CSS ให้ใช้ `var(--f-lao)`, `var(--f-plex)`, `var(--f-mono)` แทนชื่อ font ตรงๆ

### 4.6 แผนที่

```tsx
// components/MapView.tsx
'use client';
import dynamic from 'next/dynamic';
export const MapView = dynamic(() => import('./LeafletMap'), { ssr: false });
// LeafletMap.tsx ใช้ react-leaflet และตั้ง L.Icon.Default เหมือน react.md ข้อ 4.7
```

## 5. Login flow

```
/login (credentials) -> OTP -> session cookie (httpOnly)
  -> ถ้า mustChangePassword: modal ตั้งรหัสใหม่ (ปิดไม่ได้) -> redirect ตาม roleHome()
/create-account
  1 Applicant details -> 2 Verify email (OTP) -> 3 Documents (ตาม role)
  -> หน้า "Application submitted" (รอตรวจเอกสาร ผ่านแล้วส่งรหัสผ่านชั่วคราวทางอีเมล) -> /login?user=<email>
```

- ทำ step ของ login (`cred` / `otp` / `new`) เป็น state ใน client component เดียว เหมือน `vLogin()`
- Create Account ก็เช่นกัน: 3 ขั้นอยู่ใน client component เดียว อัปโหลดไฟล์ผ่าน route handler หรือ Server Action
  (ตรวจชนิด PDF/JPG/PNG และขนาด ≤10 MB ซ้ำฝั่ง server)
- หรือแยกเป็น `?step=`

## 6. Deploy

- **มี Node server** (Vercel, Docker `next start`, หรือ `output: 'standalone'`)
  - ใช้ได้ทุกฟีเจอร์: middleware, Server Actions, server rendering
- **ต้องเป็น static** (`output: 'export'`)
  - middleware, Server Actions, route handler แบบ dynamic และ `searchParams` ฝั่ง server จะใช้ไม่ได้
  - ต้องย้ายการกรองไปฝั่ง client ทั้งหมด
  - ถ้าต้อง static จริง **ใช้ React SPA ([`react.md`](./react.md)) ง่ายกว่า**
- ตั้งค่า env ของ backend (`API_BASE_URL`) แบบไม่มี prefix `NEXT_PUBLIC_` เพื่อไม่ให้หลุดไปที่ browser

## 7. Checklist ส่งงาน

- [ ] ทุก route ในตาราง README ข้อ 4 มี `page.tsx` และ middleware กันตาม role ถูก
- [ ] หน้ารายการอ่านตัวกรอง / หน้า จาก URL, refresh แล้วได้ผลเดิม, เปลี่ยนตัวกรองแล้วกลับหน้า 1
- [ ] FilterBar กรองเมื่อกด Search หรือ Enter เท่านั้น, Clear ล้างทั้งหมด
- [ ] Enterprise / Registration no. แยกคอลัมน์และแยกช่องค้นหา
- [ ] Business rule ใช้ zod schema เดียวกันทั้ง client และ Server Action
- [ ] ไม่มี hydration error (ระวังวันที่ / ตัวเลขที่ format ต่างกันระหว่าง server กับ client)
- [ ] ภาษาลาว / อังกฤษครบ, font ผ่าน `next/font`
- [ ] Responsive ตาม breakpoint ใน README ข้อ 7
- [ ] เทียบหน้าจอกับ demo ทีละหน้า
