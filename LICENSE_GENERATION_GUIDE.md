# NeuroReplay — License үүсгэх гарын авлага

Энэ гарын авлага нь NeuroReplay-н лиценз (.lic файл болон license key)
үүсгэх бүх зүйлийг хамарна.

> **Аюулгүй байдлын зарчим:** Лицензийг гарын үсэг зурдаг **хувийн түлхүүр
> (`nr-private.pem`)** нь зөвхөн (1) таны компьютер дээр болон (2) Supabase
> серверийн нууц (secret) дотор л байна. Энэ нь хэрэглэгчийн апп руу болон
> browser руу **хэзээ ч хүрэхгүй**. Тиймээс хэн ч installer задалж лиценз
> хуурамчаар үүсгэх боломжгүй.

---

## 0. Системийн бүтэц (ойлгох)

```
   [ Чи / ажилтан ]                         [ Үйлчлүүлэгч ]
         │                                         │
  Web keygen (browser) ── token ──► .lic/key ──►  NeuroReplay апп
         │                                         (зөвхөн public key-ээр
  Authorization (Supabase login)                    БАТАЛГААЖУУЛНА)
         │
         ▼
  sign-license Edge Function  ◄── NR_PRIVATE_KEY_B64 (Supabase secret)
  (хувийн түлхүүрээр гарын үсэг зурна — сервер талд)
```

Хоёр төрлийн лиценз:
- **.lic файл** — тэмцээнд зориулсан, ямар ч компьютер дээр ажиллана (огноогоор хязгаарлагдана).
- **License key (token)** — нэг тодорхой компьютерт (Machine ID) холбогдсон.

---

## 1. НЭГ УДААГИЙН ТОХИРГОО (эхлээд нэг л удаа хийнэ)

> Эдгээрийг аль хэдийн хийсэн бол **2-р хэсэг рүү** шууд орж болно.

### 1.1. Түлхүүрийн хос үүсгэх

`neuroreplay` (desktop апп) repo дотор:

```bash
node genkeys.js
```

Үүснэ:
- `nr-private.pem` → **НУУЦ.** Хэзээ ч git-д оруулж, апп-д хийж болохгүй (gitignore-д орсон).
- `nr-public.js` → апп-д явдаг (commit хийнэ).

⚠️ **`nr-private.pem`-ийг найдвартай нөөцөл** (жишээ нь нууцалсан USB, password manager).
Алдвал шинэ лиценз гаргаж чадахгүй. **Дахин `node genkeys.js` ажиллуулбал
өмнө зарсан БҮХ лиценз хүчингүй болно** — тиймээс дахин бүү ажиллуул.

### 1.2. Supabase-д хувийн түлхүүрийг secret болгож хийх

Эхлээд `nr-private.pem`-ээс Supabase-д тохирох base64 утгыг гарга
(`neuroreplay` repo дотор):

```bash
node -e "const fs=require('fs');const der=Buffer.from(fs.readFileSync('nr-private.pem','utf8').replace(/-----[^-]+-----/g,'').replace(/\s+/g,''),'base64');console.log(der.toString('base64'))"
```

Энэ нэг мөр base64 хэвлэнэ. Дараа нь (`neuroreplay_keygen_web_app` repo дотор):

```bash
supabase login
supabase link --project-ref emopxwgbxwibchyiklpw
supabase secrets set NR_PRIVATE_KEY_B64="<дээрх base64 утга>"
```

### 1.3. Edge Function-г deploy хийх

```bash
supabase functions deploy sign-license
```

Шалгах (логин хийсэн хэрэглэгчийн token-оор л ажиллана; secret зөв тавьсан эсэх):
Supabase Dashboard → Edge Functions → Logs.

### 1.4. Web keygen-г GitHub Pages руу шинэчлэх

`index.html`-г шинэ хувилбараар нь push хийнэ (нууц байхгүй болсон).

### 1.5. Desktop апп-г шинэ public key-тэйгээр build хийх

`neuroreplay` repo дотор:

```bash
npm run build
```

`nr-public.js` build-д орсон тул шинэ installer Ed25519 лицензийг таних болно.

> **Чухал:** Edge Function (1.2) болон desktop апп (`nr-public.js`) **нэг ижил
> түлхүүрийн хосоос** гарсан байх ёстой. Хэрэв `genkeys.js`-г дахин ажиллуулбал
> 1.2, 1.5-ыг **хоёуланг нь** дахин хийнэ.

---

## 2. ӨДӨР ТУТМЫН АШИГЛАЛТ — Web keygen-ээр лиценз үүсгэх

1. Web keygen хаягаа нээ: `https://<your-username>.github.io/<repo>/`
2. Имэйл + нууц үгээрээ **нэвтэр**.
3. Таб сонго:

### A) `.lic Файл` (тэмцээнд)
   - **Тэмцээний нэр**, **Эхлэх/Дуусах огноо** оруул.
   - (Төлбөртэй бол) Байгууллага, Утас, Имэйл бөглө.
   - **🎫 .lic Файл Үүсгэх** → файл download хий.
   - Имэйлийн текст автоматаар clipboard-д хуулагдана.
   - `.lic` файлыг үйлчлүүлэгчид илгээ.

### B) `License Key` (нэг компьютерт)
   - Үйлчлүүлэгчээс **Machine ID** (16 тэмдэгт) авна — апп нээхэд лицензийн
     цонхон дээр харагдана.
   - Machine ID, Эхлэх/Дуусах огноо оруул → **🔑 License Key Үүсгэх**.
   - Key автоматаар clipboard-д хуулагдана → үйлчлүүлэгчид илгээ.

> Бүх үүсгэлт Supabase **History**-д хадгалагдана.

---

## 3. CLI fallback — интернетгүй эсвэл шууд гаргах

`neuroreplay` repo дотор (`nr-private.pem` байх ёстой):

```bash
# .lic файл (интерактив)
node license.js generate

# Hardware-bound key token
node license.js genkey <MACHINE_ID> <YYYY-MM-DD> <YYYY-MM-DD>

# Шалгах
node license.js verify <файл.lic>
```

---

## 4. Үйлчлүүлэгч хэрхэн идэвхжүүлэх вэ

NeuroReplay нээгээд лицензийн цонхон дээр:
- **📁 .lic файл** таб → файлаа сонгож install, эсвэл
- **🔑 License Key** таб → key-ээ буулгаад (paste) Activate.

Дэлгэрэнгүйг `NEUROREPLAY_USER_MANUAL.md`-аас үзнэ үү.

---

## 5. Түгээмэл асуудал

| Шинж тэмдэг | Шалтгаан / Шийдэл |
|---|---|
| Web keygen "Signing failed (401)" | Нэвтрээгүй эсвэл session дууссан — дахин нэвтэр. |
| "NR_PRIVATE_KEY_B64 secret is not set" | 1.2 алхмыг хийгээгүй — secret тавь. |
| Үүсгэсэн лиценз апп дээр ажиллахгүй | Апп-ын `nr-public.js` болон Supabase secret өөр түлхүүрийн хосоос. 1.2 + 1.5-ыг ижил хосоор дахин хий. |
| Хуучин (HMAC) үед үүсгэсэн .lic ажиллахгүй | Шинэ систем тэдгээрийг таниӨхгүй — дахин шинээр үүсгэ. |
| "License not yet active" | Эхлэх огноо ирээдүйд байна. |
| "License expired" | Дуусах огноо өнгөрсөн. |

> **Тэмдэглэл:** Хуучин `HMAC_SECRET` git history-д үлдсэн ч одоо ашиггүй —
> апп түүгээр баталгаажуулахаа больсон.
