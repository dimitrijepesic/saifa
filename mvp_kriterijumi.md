# SAIFA — MVP kriterijumi i raspored faza

Ovaj dokument definiše pet dimenzija po kojima se ocenjuje da li use case ulazi u MVP, pravilo dodeljivanja faze, i primenu po grupama scenarija iz `SAIFA_use_cases_final.md`. Otvorene odluke koje blokiraju scenario referencirane su ID-em iz `otvorene_odluke.md` (OD-1 … OD-12).

## 1. Pet dimenzija evaluacije (K1–K5)

### K1 — Korisnička blokada
Bez ovog scenarija nijedan korisnik ne može da obavi nijednu korisnu radnju na platformi. To je osnovna ulazna funkcija bez koje portal nema vrednost ni za jednu personu iz `funkcionalnosti.md`. Odnosi se na radnje koje su preduslov za sve ostalo (prijava, pristup katalogu, osnovni inference/posao).
- **Test pitanje:** Ako ovog scenarija nema, postoji li ijedna persona koja i dalje može da obavi neku korisnu radnju?

### K2 — Tehnički rizik
Scenario dokazuje da deo arhitekture sa visokim rizikom zaista radi u ovom konkretnom okruženju (HPC bridge, GPU serving, spawn sveski, experiment tracking preko delimično izolovane veze). To su mesta gde `funkcionalnosti.md`/`popis.md` označavaju rizik kao srednji/visok ili navode eliminacioni kriterijum.
- **Test pitanje:** Da li ovaj scenario prvi put u stvarnom okruženju potvrđuje komponentu za koju ne znamo pouzdano da radi?

### K3 — Zavisnost odozdo
Drugi MVP use case-ovi ne mogu biti implementirani bez ovog — on je substrat na koji se drugi scenariji oslanjaju (identitet, registar resursa, kvotni model, katalog). Uklanjanje ovog scenarija ruši lanac drugih MVP scenarija.
- **Test pitanje:** Koliko drugih MVP scenarija prestaje da bude izvodljivo ako ovaj izostane?

### K4 — EuroHPC verifikabilnost
Scenario mora biti demonstrabilan u prvom javnom milestone-u prema EuroHPC-u (prijava EuroHPC identitetom, posao na PARADOX/ITE, izveštaj o GPU satima i energiji, federisani katalog). Vezuje se za cilj „EuroHPC integration" iz `funkcionalnosti.md`.
- **Test pitanje:** Da li bismo ovo pokazali na prvoj javnoj demonstraciji/izveštaju ka EuroHPC-u?

### K5 — Compliance/bezbednost minimuma
Bez ovog scenarija platforma ne sme da servisira nijedan realan zahtev: izolacija tenanata, audit, kontrola pristupa (RBAC/ABAC), GDPR minimum, operativna bezbednost klastera. `popis.md` sek. 11 i `funkcionalnosti.md` 11/12 tretiraju ovo kao tvrde zahteve, ne opcije.
- **Test pitanje:** Da li bismo, bez ovog scenarija, prekršili bezbednosni ili compliance zahtev čim primimo prvog realnog korisnika?

## 2. Pravilo dodeljivanja faze

| Uslov | Faza |
| --- | --- |
| ≥2 kriterijuma (K1–K5), nije blokiran otvorenom odlukom | MVP |
| ≥2 kriterijuma, blokiran otvorenom odlukom | Pilot (dok se odluka ne zatvori, pa prelazi u MVP) |
| 1 kriterijum | Pilot |
| 0 kriterijuma, MoSCoW Should | Production |
| 0 kriterijuma, MoSCoW Could / Won't now | Later |

MoSCoW prioritet dolazi iz `upitnik-zahtevi.md` (Deo C) i popunjava se po klijentu; dok upitnik nije popunjen, za scenarije sa 0 kriterijuma koristi se podrazumevana procena iz `funkcionalnosti.md` tabele prioriteta.

## 3. Primena po grupama use case-ova

Faza i nosilac (K kriterijum) navedeni su po scenariju; gde je scenario blokiran otvorenom odlukom, naveden je ID iz `otvorene_odluke.md`.

### Autentikacija (use case sek. 1, deo prijave)
- **Registracija lokalnim nalogom** — MVP (K1 + K3 + K5). Bez naloga nema nijedne radnje.
- **Prijava institucionalnim SSO-om (eduGAIN/AMRES)** — MVP (K1 + K4 + K5). Akademska federacija demonstrabilna; zavisnost od AMRES pristupa je organizaciona, ne menja fazu.
- **Prijava EuroHPC identitetom (MyAccessID)** — MVP (K1 + K4). Direktno nosi EuroHPC verifikabilnost (Keycloak identity brokering).
- **Kreiranje i upravljanje API ključem** — MVP (K1 + K5). Programski pristup za inference/poslove; opoziv mora delovati odmah (`funkcionalnosti.md` 11 DoD).

### Katalog (use case sek. 2 + sek. 6 Datasetovi)
- **Pretraga i filtriranje javnog kataloga (anon)** — MVP (K1 + K3). Ulazna tačka „one-stop shop", full-text (`funkcionalnosti.md` 7 MVP).
- **Pretraga i filtriranje kataloga resursa (prijavljen)** — MVP (K1 + K3 + K5) za RBAC/public vidljivost; ABAC-striktna vidljivost blokirana **OD-8** → taj aspekt ostaje Pilot dok se OD-8 ne zatvori.
- **Pregled opisa i uzorka javnog dataseta** — MVP (K1 + K3). Javni pregled bez naloga.
- **Upload dataseta** — MVP (K3 + K1). Punjenje kataloga resursima (`funkcionalnosti.md` 4 MVP delimično).
- **Preuzimanje javnog dataseta** — MVP (K1 + K3).
- **Uređivanje metapodataka ili vidljivosti objavljenog dataseta** — Pilot (K5, 1 kriterijum; promena na osetljivo zahteva review).
- **Objava resursa u katalog uz slanje na pregled** — Pilot (review/approval, `funkcionalnosti.md` 13 Pilot); povezano sa **OD-1**.
- **Pregled i odluka o objavi dataseta** — Pilot (K5 delimično; review/approval Pilot); refinement blokiran **OD-1**.
- **Odobravanje objave osetljivog dataseta** — Pilot (K5). GDPR/compliance gate; zavisi od onboardinga osetljivih podataka.
- **Skidanje javnog resursa sa kataloga (moderacija)** — Pilot (K5, 1 kriterijum); masovna varijanta je **OD-2**.

### Inference (use case sek. 3)
- **Pozivanje modela (REST API ili Jupyter sveska)** — MVP po `funkcionalnosti.md` 2 (K1 + K2 + K4), ali blokiran **OD-3** (serving model), **OD-5** (kvotni model) i **OD-8** (ABAC vidljivost) → ostaje **Pilot** dok se te odluke ne zatvore.
- **Pokretanje batch inference posla nad datasetom** — Pilot (K2; blokiran **OD-4** rutiranje, **OD-3**, **OD-5**).
- **Podešavanje rate limitinga (kvote)** — Pilot (K5; blokiran **OD-5**).
- **Deploy sopstvenog modela kao inference endpoint** — Later (blokiran **OD-6**; napomena eksplicitno: ne za MVP).
- **Zamena verzije modela na endpointu bez prekida** — Later (blokiran **OD-7**; zavisi od OD-3).

### HPC / AI factory (use case sek. 4)
- **Pokretanje računski zahtevnog posla** — MVP po `funkcionalnosti.md` 5 (K1 + K2 + K4), ali blokiran **OD-4** (rutiranje) → **Pilot** dok se OD-4 ne zatvori.
- **Praćenje statusa i logova posla** — MVP (K1 + K2). Status iz SLURM/scheduler-a, live log (SSH+sbatch putanja, ne zavisi od operatera).
- **Preuzimanje rezultata posla** — MVP (K1 + K3). Rezultati iz MinIO, resumable.
- **Prijem notifikacije o ishodu posla** — MVP (K1, in-app deo isporuke rezultata); email kanal Pilot.
- **Prisilno zaustavljanje posla koji ugrožava stabilnost klastera** — MVP (K5 + K2). Operativna bezbednost klastera.
- **Zahtev za povećanje kvote** — Pilot (K5, 1 kriterijum); kvotni model povezan sa **OD-5**.
- **Odluka o zahtevu člana za povećanje kvote (predstavnik)** — Pilot (K5, 1 kriterijum); povezano sa **OD-5**.
- **Pokretanje posla iz zaključanog šablona (student)** — Pilot (K2 reuse, 1 kriterijum); edukacioni kontekst, zavisi od šablona.

### Model lifecycle (use case sek. 5)
- **Registracija istreniranog artefakta kao modela (verzija 1)** — MVP (K3 + K1). Registar je substrat za katalog/objavu (`funkcionalnosti.md` 1 MVP).
- **Dodavanje nove verzije postojećeg modela** — MVP (K3) za normalni tok; grana nepotpunog lineage-a blokirana **OD-10**.
- **Evaluacija verzije modela na benchmark setu** — MVP (K2; `funkcionalnosti.md` 10 basic harness MVP). Reproduktivnost evaluacije.
- **Fine-tuning modela nad sopstvenim labeled datasetom** — Pilot (K2; `funkcionalnosti.md` 3 basic LoRA), blokiran **OD-4** (rutiranje).
- **Treniranje modela nad datasetom iz kataloga** — Pilot (K2), blokiran **OD-4**.
- **Fine-tuning nad osetljivim podacima (compute-to-data)** — Pilot (K5 + K2), blokiran **OD-8** (ABAC vidljivost) i **OD-4**; zahteva ABAC/DPIA, kandidat za kasniju fazu.
- **Pregled i poređenje eksperimenata (experiment tracking)** — Pilot (K2, eliminacioni kriterijum); blokiran **OD-9**.
- **Promocija `run`-a u verziju registrovanog modela** — Pilot (K3); zavisi od experiment trackinga (**OD-9**).
- **Vraćanje servisnog aliasa na prethodnu verziju** — Pilot (K3) za alias u registru; živa instanca blokirana **OD-11**.
- **Podnošenje verzije modela za recenziju i objavu** — Pilot (review/approval, `funkcionalnosti.md` 13 Pilot).
- **Pregled evaluacije i odobravanje / odbijanje / vraćanje verzije (model reviewer)** — Pilot (review/approval); povezano sa **OD-1**.
- **Objava odobrene verzije lokalno i (opciono) u Pharos** — Pilot (K4); lokalna objava review-gated, Pharos izvoz zavisi od partnera (`funkcionalnosti.md` 9 MVP samo mock).
- **Objava leaderboard-a sa verzionisanjem benchmarka** — Pilot (`funkcionalnosti.md` 10 leaderboard Pilot); blokiran **OD-12**.

### Kolaboracija (use case sek. 7)
- **Kreiranje projekta** — MVP (K5 + K3; team vidljivost `funkcionalnosti.md` 13 MVP). Substrat za timsko deljenje.
- **Pozivanje člana u projekat** — MVP (K5 + K3; team vidljivost).
- **Uklanjanje člana iz projekta** — MVP (K5; oduzimanje pristupa resursima projekta).
- Napomena: review/approval kolaboracioni tokovi (odobravanje objave, validation reports) su Pilot i navedeni su u grupama Katalog i Model lifecycle.

### Edukacija (use case sek. 8)
- **Otvaranje interaktivne Jupyter lab sveske** — MVP (K2; JupyterHub + KubeSpawner `funkcionalnosti.md` 6 MVP) za samu infrastrukturu sveske; kurs-vezana lab vežba Pilot.
- **Pregled kurseva i learning pathova** — Pilot (LMS Pilot; `funkcionalnosti.md` 8 MVP samo biblioteka materijala).
- **Upis na kurs** — Pilot (LMS Pilot).
- **Praćenje napretka na kursu** — Pilot (LMS Pilot).
- **Pokretanje unapred konfiguriranog AI factory posla u lab vežbi** — Pilot (K2 reuse; zaključani SLURM šablon).
- **Preuzimanje sertifikata o završetku kursa** — Pilot/Later (sertifikati su deo LMS pilota; 0–1 kriterijum).
- **Kreiranje kursa i unos materijala (mentor)** — Pilot (LMS Pilot).
- **Praćenje napretka studenata (mentor)** — Pilot (LMS Pilot).
- **Konfiguracija resource limita za studente (predstavnik)** — Pilot (K5, 1 kriterijum); povezano sa **OD-5** (kvotni/limit model).

### Federacija (use case sek. 10)
Cela grupa je Pilot: `funkcionalnosti.md` 9 stavlja u MVP samo mock/read-only, a zrelost Pharos/IT4LIA API-ja je „najveća nepoznanica projekta" (visok rizik zavisnosti od partnera).
- **Pristup federisanim resursima sa partnerske platforme** — Pilot (K4; mock/read-only u MVP, realna veza Pilot).
- **Izvoz resursa na partnersku platformu** — Pilot (K4; zavisi od partnera).
- **Konfiguracija automatskog sync-a kataloga** — Pilot (zavisi od partnera; validacija šeme).
- **Ručni uvoz resursa sa partnerske platforme** — Pilot (mock u MVP).

### Administracija (use case sek. 1 — nalozi/organizacije/sesije, i sek. 9)
- **Registracija organizacije i zahtev za odobrenje** — MVP (K3 + K5; multi-tenant substrat).
- **Odobravanje ili odbijanje registracije organizacije (admin)** — MVP (K3 + K5).
- **Dodavanje / Uklanjanje člana u organizaciju** — MVP (K3 + K5; tenant izolacija i kvote).
- **Dodeljivanje uloge predstavnika organizacije** — MVP (K3; IAM uloge).
- **Kreiranje / Uređivanje / Deaktivacija naloga (admin)** — MVP (K3 + K5; IAM `funkcionalnosti.md` 11 MVP).
- **Dodeljivanje uloga i ABAC atributa korisniku** — MVP (K5 + K3); semantika ABAC vidljivosti povezana sa **OD-8**.
- **Prisilno opozivanje sesije korisnika pri bezbednosnom incidentu** — MVP (K5; bezbednosni minimum).
- **Dodeljivanje ili izmena kvote organizaciji (admin)** — MVP (K3 + K5); korisnički podlimiti povezani sa **OD-5**.
- **Raspodela kvote članu organizacije (predstavnik)** — Pilot (K3/K5); blokiran **OD-5**.
- **Praćenje potrošnje resursa organizacije** — MVP (K4; GPU sati/energija po projektu — EuroHPC izveštavanje).
- **Nadzor metrika platforme u realnom vremenu** — MVP (K4 + K2; `funkcionalnosti.md` 12 MVP).
- **Pregled audit loga** — MVP (K5 + K4; append-only, retencija, compliance izveštaji).
- **Konfiguracija politike gašenja neaktivnih GPU sesija** — MVP (K2; idle culling deo DoD `funkcionalnosti.md` 6).
- **Pregled lične istorije aktivnosti** — Pilot (K5, 1 kriterijum; read nad auditom).
- **Praćenje statusa i logova inference endpointa** — Pilot/Later (zavisi od deploja endpointa — **OD-6**).
- **Postavljanje alarma na pragove iskorišćenosti** — Pilot (1 kriterijum; MoSCoW Should).
- **Zahtev za potvrdu usklađenosti resursa sa sektorskim propisima** — Pilot (K5, 1 kriterijum; sektorska compliance).
- **Pokretanje DPIA procedure za dataset sa ličnim podacima** — Pilot (K5; GDPR DPIA, aktivira se sa onboardingom osetljivih podataka).

## 4. Ograničenja kriterijuma

Ovi kriterijumi eksplicitno ne rešavaju:

- **Redosled implementacije unutar MVP-a** — kriterijumi kažu šta jeste MVP, ne kojim redom se MVP scenariji grade.
- **Kapacitet tima** — ne uzimaju u obzir koliko ljudi i koliko vremena tim ima; mali tim može morati da seče i unutar MVP-a (`popis.md` vodeći princip: reuse before build).
- **Zavisnost od operatera klastera** — slurmrestd, S3/MinIO na HPC strani, kvote i pristup nisu zagarantovani (`popis.md` sek. 5, `funkcionalnosti.md` 5: visok rizik); kriterijumi ne mogu da preklasifikuju scenario koji zavisi od spoljne spremnosti.
- **Zavisnost od partnera (EuroHPC federacija)** — zrelost Pharos/IT4LIA API-ja i metapodataka je van kontrole tima i ne menja se ocenjivanjem (`funkcionalnosti.md` 9).
- **Zatvaranje otvorenih odluka** — kriterijumi identifikuju da je scenario blokiran (OD-1 … OD-12), ali ne donose samu arhitektonsku odluku; ona se razrešava kroz PoC/upitnik (`otvorene_odluke.md`).
- **MoSCoW i obim po klijentu** — konačan prioritet zavisi od popunjenog `upitnik-zahtevi.md` (učestalost/obim, zrelost klijenta), što kriterijumi ne mogu da pretpostave.
