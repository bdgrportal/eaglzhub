# 🦅 EAGLZ Hub · MVP (tesztfázis)

Zárt ügyfélportál: meghívókódos belépés, pénzügyi térkép, dokumentumtár, segítségközpont,
konzultáció-igény, kalkulátorok, tudástár, tanácsadói pult. Egyetlen `index.html`, backend: Supabase.

## Üzembe helyezés (kb. 20 perc)

1. **Supabase projekt** (ÚJ, külön a Nesttől!): supabase.com → New project → régió: EU (Frankfurt).
2. **SQL**: SQL Editor → `hub_setup.sql` tartalmát beilleszteni → Run. A végén írd át a tanácsadók
   e-mailjeit és kódjait a valósakra (8. blokk), vagy utólag a Table Editorban (`hub_advisors`).
3. **Auth beállítás**: Authentication → Providers → Email: ON.
   Tesztfázisban: Authentication → Settings → "Confirm email" **KI** (így a kollégák azonnal be tudnak lépni).
4. **URL-ek**: Authentication → URL Configuration → Site URL és Redirect URLs = a GitHub Pages címed
   (pl. `https://<felhasznalo>.github.io/eaglz-hub/`).
5. **Kulcsok**: Project Settings → API → Project URL + `anon public` kulcs → `index.html` tetején a `CFG` blokkba.
6. **GitHub Pages**: repo → Settings → Pages → Deploy from branch → main / root. Az `index.html` gyökérben legyen.
7. Nyisd meg az oldalt, regisztrálj, add meg a kódot, fogadd el a tesztfázis-tájékoztatót. Kész.

Tanácsadói belépés: a tanácsadó **ugyanazzal az e-maillel** regisztrál, ami a `hub_advisors` táblában van.
Kód nélkül, egyből a Tanácsadói pultra kerül. (Ügyfél-nézetet tesztelni a saját kódjával lehet.)

## Szerepek

| Szerep | Ki | Mit lát / tehet |
|---|---|---|
| **Kis admin** (`advisor`) | tanácsadó kollégák | a saját meghívókódjával érkezett ügyfelek: térkép, ügyek, jelzések, kerék, szint, megosztott dokumentumok |
| **Nagy admin** (`admin`) | Richárd + asszisztensek | minden ügyfél és ügy; ügyfél átrendelése másik tanácsadóhoz; 👑 Tanácsadók és kódok oldal: új kolléga felvétele, kód, szerep, aktiválás/kikapcsolás |

A szerep a `hub_advisors.role` mezőben van, adatbázis-szinten érvényesül (RLS), nem csak a felületen.
Nagy admin felvétele: a 👑 oldalon, vagy SQL: `update hub_advisors set role='admin' where email='...';`
A dokumentumokba a nagy admin is csak az ügyfél megosztási kapcsolójával lát bele (ezt ígéri a tájékoztató).

## Top 5 biztonsági beállítás (kötelező a teszt indítása előtt)

1. **Row Level Security mindenhol** (a SQL bekapcsolja): ügyfél csak a sajátját, tanácsadó csak a hozzá
   rendelt ügyfeleket látja. A Supabase Dashboardban ellenőrizd: minden `hub_*` táblán RLS = enabled.
2. **A kódlista nem lekérdezhető**: a kód beváltása szerveroldali függvény (`hub_redeem_code`), a
   `hub_advisors` táblát csak a saját tanácsadó sorára engedi a policy. Soha ne tedd publikussá.
3. **Privát dokumentumtár**: a `hub-docs` bucket `public = false`, aláírt (10 perces) linkekkel nyílik,
   a tanácsadó csak akkor lát bele, ha az ügyfél bekapcsolta a megosztást.
4. **Auth szigorítás**: Authentication → Settings: jelszó min. 8 karakter (az app is kényszeríti),
   "Enable email confirmations" a tesztfázis UTÁN vissza BE, rate limit alapértelmezett.
   Csak az `anon` kulcs kerül a frontendbe, a `service_role` kulcsot SOHA.
5. **Adatkezelés**: EU régió, HTTPS (GitHub Pages ad), `noindex` (a fájlban van), a tesztfázis-tájékoztató
   kötelező elfogadása belépéskor (`terms_accepted_at`), törlési kérelem esetén: Authentication → Users →
   Delete user (a kapcsolódó sorok automatikusan törlődnek).

## Fájlok
- `index.html` · a teljes alkalmazás (frontend)
- `hub_setup.sql` · táblák, jogosultságok, függvények, storage
