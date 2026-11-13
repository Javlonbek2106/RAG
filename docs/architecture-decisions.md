# Arxitektura qarorlari

Manba: [architecture-audit.md](architecture-audit.md). Rejalashtirish: [architecture-roadmap.md](architecture-roadmap.md).

Format: har qaror uchun **Holat**, **Kontekst/dalil**, **Qaror**, **Oqibat**. Dalil belgilari auditdagidek: [T] tasdiqlangan, [O] oldingi o'lchov, [X] taxmin.

## 1. Mavjud qarorlar (qabul qilingan, saqlanadi)

| ID | Qaror | Dalil | Oqibat |
|---|---|---|---|
| ADR-001 | **Modulli monolit** + alohida, nazorat ostidagi worker jarayoni; mikroservis yo'q | Tsiklsiz qatlamlar [T]; to'siqlar tashqi xizmatlarda (Ollama, Qdrant) [T] | Oddiy joylashtirish. Qayta ko'rish sharti: o'lchangan va monolit ichida hal bo'lmaydigan to'siq |
| ADR-002 | **PostgreSQL manba, Qdrant hosila** | Faollashtirish bitta tranzaksiya; qidiruv faol id lar bilan filtrlanadi [T] | Qdrant qayta qurilishi kerak, lekin hozir buning vositasi yo'q (QDR-01/02) |
| ADR-003 | **Versiya qatori o'zi ish (job)** (alohida `jobs` jadvalisiz) | Navbat/heartbeat/lease/qayta urinish versiyada, 129 DB test [T] | Hujjat holati — hosila xulosa (`_sync_document`) |
| ADR-004 | **Faollashtirish avlodi** (`activation_generation`/`uploaded_generation`) + **qat'iy lease** | Tasodifiy interleaving, 30 raund ikki rollback, mutatsiya tekshiruvi [T] | Rollback doimiy qaror; eskirgan nomzod `SUPERSEDED` bo'lib saqlanadi (isrof: oxirigacha ishlanadi) |
| ADR-005 | **Deterministik nuqta id** `uuid5(versiya:indeks)` + faollashtirishdan oldin `verify` + DB da tekshirilgan son | Qayta urinish dublikat bermaydi [T] | Qisman bo'laklardan davom etish mumkin (T1.2) |
| ADR-006 | **Qidiruv faol versiya id lari bilan filtrlanadi, xavfsiz yopiladi** | `answer.py:49-54` [T]; 10 000 id da +8 ms [T] | Hajm hujjat soniga chiziqli (RET-02) |
| ADR-007 | **Faqat loopback, autentifikatsiyasiz** (vaqtinchalik, 3-bosqichgacha) | `HOST` qat'iy, Host/Origin tekshiruvi [T] | Faqat yakka foydalanuvchi; ko'p foydalanuvchida Critical (SEC-01/02) |
| ADR-008 | **Prompt asosidagi himoya + manba ko'rsatish**; modelda vositalar yo'q, UI faqat `textContent` | XSS yo'q [T]; injection 3/4 bajarilmadi [T] | Ta'sir javob mazmuni bilan cheklangan; to'liq himoya emas (LLM-04) |
| ADR-009 | **512 tokenli bo'laklar, "triplet" jadval, gibrid va reranker yo'q** | 256 token 74% < 80%; Markdown jadval, BM25: foyda yo'q; reranker: javob 80→78%, +10 s [O] | Dalilsiz o'zgartirilmaydi; generator o'zgarsa qayta baholash |
| ADR-010 | **Docling + Tesseract worker jarayonida**, Excel uchun o'z o'quvchi | Excel javobi 50% → 100% [O] | Og'ir kutubxonalar faqat worker/`ingest` da (lazy) |
| ADR-011 | **Faqat qo'shimcha SQL migratsiyalar** (advisory lock, bir marta) | Jonli bazada 3 migratsiya xavfsiz qo'llandi [T] | Halokatli o'zgarishlar alohida tasdiq bilan (DOC-07) |
| ADR-012 | **Konfiguratsiya `RAG_*` muhit o'zgaruvchilari, `config.NAME` orqali** | 70 qator, testlar `monkeypatch` qiladi [T] | Import paytida o'qiladi, tekshiruvsiz (ARC-05) |
| ADR-013 | **Vanilla JS UI**, qayta yuklanmaydigan polling, `textContent`, qat'iy CSP | e2e xavfsizlik testi [T] | A11y va yangilanish narxi qarzi (API-04/05, DB-01) |
| ADR-014 | **Baholash ishlab chiqarish kodini o'lchaydi**; ma'lumotlar git dan tashqarida | `eval` `answer/ingest/store/llm` ni ishlatadi [T] | Manifest yo'q (TST-02) |

## 2. Qayta ko'rib chiqilishi kerak bo'lgan qarorlar

| ID | Qaror | Muammo | Taklif |
|---|---|---|---|
| RC-1 | **Qat'iy 30 daqiqalik job timeout** (`config.py:23`) | Katta jadvalli hujjat indekslanmaydi (ING-01) | ADR-P1 |
| RC-2 | **LLM chaqiruvi sinxron, timeoutsiz, bekor qilinmaydi, Ollama ketma-ket** | Osilish, navbat, bekor qilinmaslik (LLM-01/02/03) | ADR-P2 |
| RC-3 | **Sahifalanmagan ro'yxat va polling** | 10 000 hujjatda 3,4 MiB (DB-01) | ADR-P5 |
| RC-4 | **Uch ombor, moslashtirishsiz** | Jim nomuvofiqlik (DOC-03) | ADR-P4 |
| RC-5 | **Global noyob hujjat nomi** | Ko'p foydalanuvchida to'qnashuv/sizish (DOC-06) | ADR-P8 |
| RC-6 | **Embedding identifikatsiyasi saqlanmaydi** | Jim vektor aralashishi (QDR-01) | ADR-P3 |
| RC-7 | **Yuklash Starlette orqali, limit tanadan keyin** | Disk to'ldirish (API-02) | ADR-P10 |
| RC-8 | **Versiyalar abadiy saqlanadi** | Chegarasiz o'sish (DOC-02) | ADR-P7 |
| RC-9 | **Himoya faqat prompt** | Injection (LLM-04) | ADR-P9 |
| RC-10 | **Baholash yaxlitligi va quvvati** | n=69 (±9,4 pp), manifest yo'q (TST-02) | ADR-P11 |

## 3. Taklif etilgan ADR lar (tasdiq kerak)

Hammasi **Taklif** holatida; hech biri amalga oshirilmagan.

### ADR-P1: Job timeout taraqqiyotga asoslanadi
- **Kontekst**: 381 betlik byulleten 49–60 daqiqa [O], chegara 30 daqiqa; hujjat hajmiga qaramay `FAILED` va worker qayta ishga tushadi (ING-01).
- **Qaror (taklif)**: mutlaq timeout o'rniga **taraqqiyot bo'lmagan** vaqt chegarasi (masalan, 10 daqiqa); worker bosqich/sahifa taraqqiyotini DB ga yozadi; yuqori mutlaq chegara (masalan, hajmga qarab) faqat himoya sifatida.
- **Alternativlar**: (a) mutlaq chegarani oshirish (qotgan parserni uzoq kutadi); (b) sahifa × byudjet (sahifa soni oldindan ma'lum bo'lmasa noaniq).
- **Oqibat**: taraqqiyot maydoni va UI ko'rsatkichi; qotgan parser baribir tezroq aniqlanadi. Qabul: T0.3.
- **Dalil kerak**: LT-2 (sahifa/s turli hujjatlar uchun).

### ADR-P2: LLM shlyuzi (navbat, timeout, bekor qilish, isitish)
- **Kontekst**: Ollama so'rovlarni ketma-ket bajaradi, uzilgan mijoz generatsiyani to'xtatmaydi, timeout yo'q, sovuq start 28 s [T, E3].
- **Qaror (taklif)**: `llm` modulida chegaralangan navbat (band bo'lsa aniq javob), so'rov timeouti, `num_predict`, `keep_alive`, mijoz uzilganda oqimni yopish; bir vaqtda 1 generatsiya (Ollama xususiyatiga mos).
- **Alternativlar**: parallel Ollama nusxalari (RAM/CPU ni bo'lishadi, tezlashuv o'lchanmagan [X]); alohida GPU xizmati (o'lchangan asos paydo bo'lganda).
- **Oqibat**: ko'p foydalanuvchida kutish aniq va cheklangan. Qabul: T0.4, T3.5 (LT-1).
- **Dalil kerak**: Ollama oqimi yopilganda generatsiya to'xtashini alohida tekshirish [X].

### ADR-P3: Indeks metama'lumoti va qayta indekslash
- **Kontekst**: embedding modeli o'zgarsa vektorlar jimgina aralashadi; qayta qurish buyrug'i yo'q (QDR-01/02).
- **Qaror (taklif)**: `index_meta` (model, o'lcham, tokenizer, bo'laklash parametrlari, kolleksiya nomi); versiyada `embedded_with`; ishga tushishda nomuvofiqlikni rad etish; `rag reindex` yangi kolleksiyani PG+fayllardan quradi, `verify` qiladi, so'ng atomik almashtiradi (`COLLECTION` o'zgarishi). Faollashtirish yo'li o'zgarmaydi.
- **Alternativlar**: kolleksiya nomiga model nomini kiritish (aralashishni oldini oladi, lekin qayta qurishni hal qilmaydi).
- **Oqibat**: modelni xavfsiz almashtirish mumkin. Qabul: T1.1.

### ADR-P4: Moslashtirish (reconcile) va sog'liq
- **Kontekst**: `READY` versiya vektori yo'qolsa aniqlanmaydi, yetim fayl/nuqta sanalmaydi (DOC-03), kuzatuvchanlik minimal (WRK-02).
- **Qaror (taklif)**: faqat o'qiydigan `reconcile` (versiya vektor soni, fayl mavjudligi/hashi, yetimlar); `/healthz`, `/readyz`, `/api/status`; tuzatish alohida buyruqlar bilan, tasdiq bilan.
- **Oqibat**: jim nomuvofiqlik ko'rinadi; zaxira/tiklash mashqining tekshiruv vositasi. Qabul: T0.2.

### ADR-P5: API v1, xato konverti, sahifalash
- **Kontekst**: versiyasiz yo'llar, uch xil xato shakli, sahifalanmagan ro'yxat (API-01, ARC-06, DB-01).
- **Qaror (taklif)**: `/api/v1/...`; xato `{error:{code,message}}`; ro'yxatda `limit/cursor`, `q`, `If-None-Match`; eski yo'llar o'tish davrida saqlanadi; OpenAPI hujjati ixtiyoriy yoqiladi (autentifikatsiya joriy bo'lguncha o'chiq qoladi).
- **Oqibat**: tashqi mijozlar (.NET) barqaror shartnomaga ega; UI yuki kamayadi. Qabul: T1.5.

### ADR-P6: Suhbat modeli
- **Kontekst**: funksiya mavjud emas (CNV-01); vazifa matnida bor deb ko'rsatilgan.
- **Qaror (taklif, funksiya kerak bo'lsagina)**: PostgreSQL da `conversations(id, owner_id nullable, created_at)` va `messages(id, conversation_id, seq, role, content, sources, created_at)`; `seq` suhbat qulfi ostida beriladi (tartib kafolati); retrieval hujjat holatiga tegmaydi (`active_version_ids` o'zgarishsiz); tarix oynasi token byudjeti bilan; ixtiyoriy savolni qayta yozish (qo'shimcha LLM chaqiruvi: avval foydasini baholash).
- **Alternativlar**: brauzerda saqlash (izolyatsiya va persistentlik yo'q).
- **Oqibat**: `owner_id` 3-bosqich bilan; LLM navbati (ADR-P2) hal bo'lishi kerak. Qabul: T2.5.
- **Dalil kerak**: baholash to'plamida davomli savollarda yaxshilanish.

### ADR-P7: Versiyalarni saqlash siyosati
- **Kontekst**: `SUPERSEDED`/`FAILED` versiyalar abadiy (DOC-02).
- **Qaror (taklif)**: `rag prune`: har hujjat uchun oxirgi N (standart 3) `SUPERSEDED` saqlanadi, qolganining fayli va vektorlari o'chiriladi (qator va tarix saqlanadi, holat `PURGED` emas, balki `chunk_count` va fayl yo'li bo'shatiladi: **ochiq savol**); avtomatik emas.
- **Oqibat**: rollback kafolati N ichida saqlanadi; N dan tashqariga qaytish rad etiladi (aniq xabar). Qabul: T1.6.
- **Ochiq savol**: tarixiy qatorni saqlash/saqlamaslik, N ni sozlash.

### ADR-P8: Ko'p foydalanuvchi ma'lumotlar modeli va autentifikatsiya
- **Kontekst**: egasi tushunchasi yo'q, global nom, autentifikatsiya yo'q (SEC-01/02, DOC-06).
- **Qaror (taklif)**: `documents.owner_id`; `(owner_id, name)` noyob; nom NFC ga normallashtiriladi; `active_version_ids(owner)` va Qdrant filtri egasi bo'yicha (fail-closed saqlanadi); ruxsat modeli (egasi, ko'rish) alohida tasdiq bilan; sessiya/token.
- **Alternativlar**: har foydalanuvchiga alohida kolleksiya (izolyatsiya kuchli, lekin ko'p kolleksiya va qayta indekslash narxi [X]).
- **Oqibat**: 0004+ migratsiya (faqat qo'shimcha; mavjud hujjatlar bitta "merosxo'r" egaga). Qabul: T3.1, T3.2.

### ADR-P9: Prompt-injection chuqurlashtirilgan himoya
- **Kontekst**: faqat prompt; 1/4 bajarildi [T]; fayl nomi ham promptda (LLM-04).
- **Qaror (taklif)**: hujjat bloklarini aniq ajratuvchilar bilan belgilash va "ma'lumot, ko'rsatma emas" qoidasi; fayl nomini tozalash/qisqartirish; chiqish tekshiruvi (mavjud manba raqamlari); kengaytirilgan injection to'plami; modelga vosita bermaslik (hozirgi holat saqlanadi).
- **Oqibat**: xavf kamayadi, yo'qolmaydi; hech qachon "yechildi" deb belgilanmaydi. Qabul: T1.9.

### ADR-P10: Yuklashni qayta ishlash
- **Kontekst**: limit tana to'liq spool qilingandan keyin tekshiriladi (API-02).
- **Qaror (taklif)**: 1-bosqich `Content-Length` bo'yicha oldindan rad etish; 2-bosqich: oqimli ko'p qismli tahlil (chegaraga yetganda uzish); sinxron qismni threadpool ga (API-03).
- **Oqibat**: disk to'ldirish yo'li yopiladi. Qabul: T0.5, T1.3.

### ADR-P11: Baholash boshqaruvi
- **Kontekst**: n=69 (±9,4 pp), n=9 rad etish, n=4 injection; manifest yo'q (TST-02).
- **Qaror (taklif)**: baholash ma'lumotlari manifesti (SHA-256) va `eval.run` boshida tekshiruv; kengaytirilgan to'plam (≥200) mavjudini o'zgartirmasdan; hisobotda 95% oraliq; qarorlar juft taqqoslash bilan; CI da kechasi/qo'lda.
- **Oqibat**: sifat qarorlari statistik himoyalangan. Qabul: T1.8.

## 4. Qarorlar jurnali

| Sana | Hodisa |
|---|---|
| 2026-11-13 | Audit o'tkazildi; barcha ADR-P1…P11 **Taklif** holatida, tasdiq kutilyapti |
