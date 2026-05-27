# NeuroReplay Keygen — Supabase + GitHub Pages тохиргооны заавар

---

## 1. Supabase Project үүсгэх

1. https://supabase.com → **Start your project**
2. GitHub-аар нэвтэр
3. **New project** → нэр өг (жишээ: `neuroreplay-keygen`)
4. Database Password хадгалаад **Create new project** дар

---

## 2. Database Table үүсгэх

**SQL Editor** хэсэгт доорх SQL-г ажиллуул:

```sql
create table keygen_history (
  id          bigserial primary key,
  type        text not null,           -- 'lic' эсвэл 'key'
  event_name  text,                    -- .lic tab-ийн тэмцээний нэр
  hwid        text,                    -- key tab-ийн Machine ID
  start_date  date not null,
  end_date    date not null,
  created_by  text,                    -- хэрэглэгчийн email
  created_at  timestamptz default now()
);

-- Зөвхөн нэвтэрсэн хэрэглэгч уншиж, бичих боломжтой
alter table keygen_history enable row level security;

create policy "auth users can read"
  on keygen_history for select
  using (auth.role() = 'authenticated');

create policy "auth users can insert"
  on keygen_history for insert
  with check (auth.role() = 'authenticated');

create policy "auth users can delete"
  on keygen_history for delete
  using (auth.role() = 'authenticated');
```

---

## 3. Хэрэглэгч нэмэх

**Authentication → Users → Add user** → и-мэйл + нууц үг оруулна.

> Ideree-ийн ажилтан бүрт нэг нэг хэрэглэгч үүсгэж өг.

**Чухал:** Supabase dashboard-аас "Confirm email" checkbox-ийг арилгах:
**Authentication → Settings → Email Auth → Confirm email** → OFF болгох
(тэгэхгүй бол хэрэглэгч email баталгаажуулах шаардлагатай болно)

---

## 4. API Keys авах

**Settings → API** хэсэгт:
- `Project URL` → copy
- `anon public` key → copy

---

## 5. index.html тохиргоо

`index.html` файлын эхний хэсэгт:

```javascript
const SUPABASE_URL  = 'https://xxxxxxxxxxxxxx.supabase.co';  // чинийх
const SUPABASE_ANON = 'eyJhbGciOiJIUzI1NiIsInR...';          // anon key
```

Эдгээр 2 утгыг Supabase-аас авсан зүйлсээр солино.

---

## 6. GitHub Pages-д байрлуулах

### 6a. Repository үүсгэх
1. https://github.com → **New repository**
2. Нэр: `neuroreplay-keygen` (эсвэл хүссэн нэр)
3. **Private** болгох (**чухал** — public бол код харагдана!)
4. Create repository

### 6b. Файл upload хийх
1. Repository-д орж **Add file → Upload files**
2. `index.html` нэг файлыг л upload хийнэ
3. Commit

### 6c. GitHub Pages асаах
1. Repository → **Settings → Pages**
2. Source: **Deploy from a branch**
3. Branch: `main`, folder: `/ (root)`
4. **Save**

Хэдэн минутын дараа:
`https://your-username.github.io/neuroreplay-keygen/`

---

## 7. Supabase CORS тохиргоо

**Supabase → Settings → API → CORS allowed origins** хэсэгт:
```
https://your-username.github.io
```
нэм.

---

## Дуусгавар

| Зүйл | Байдал |
|------|--------|
| Login | Supabase Auth (email + нууц үг) |
| .lic файл | Browser-д үүсгэж download |
| License Key | Browser-д үүсгэж clipboard |
| History | Supabase DB, бүх хэрэглэгч харна |
| Хамгаалалт | RLS — зөвхөн нэвтэрсэн хэрэглэгч |

---

## Асуудал гарвал

**"Invalid API key"** → SUPABASE_URL / SUPABASE_ANON буруу байна, дахин шалга  
**Login ажиллахгүй** → Email confirm OFF болсон эсэхийг шалга  
**History харагдахгүй** → SQL-д RLS policy зөв үүссэн эсэхийг шалга  
