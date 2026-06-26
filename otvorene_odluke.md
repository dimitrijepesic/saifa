# SAIFA — Otvorene arhitektonske odluke

Ovo su pitanja koja još nisu zatvorena, a utiču na to koji scenariji ulaze u MVP. Svaka odluka navodi šta blokira (preko use case ID-eva iz `SAIFA_use_cases_final.md`), ponuđene opcije sa preporukom, i fazu. Faza se određuje po kriterijumima K1–K5 iz `mvp_kriterijumi.md`: odluka je **MVP** ako blokira bar jedan use case koji prolazi ≥2 kriterijuma, inače **Later**.

## Pregled

| ID | Pitanje | Faza |
| --- | --- | --- |
| OD-1 | Zaseban status `rejected` (≠ `draft`) za odbijene resurse? | Later |
| OD-2 | Masovno podešavanje vidljivosti kataloga kao zaseban use case? | Later |
| OD-3 | Serving: deljeni pool ili self-service deploy? | MVP |
| OD-4 | Prag rutiranja Kubernetes ↔ SLURM? | MVP |
| OD-5 | Korisnički podlimiti kvote ili samo budžet institucije? | MVP |
| OD-6 | Postoji li self-service deploj modela u MVP-u? | Later |
| OD-7 | Vlasništvo serving lifecycle-a i provera API-kompatibilnosti? | Later |
| OD-8 | Vidljivost kataloga uračunava ABAC ili tek pri akciji? | MVP |
| OD-9 | Experiment tracking preko delimično izolovane veze? | Later |
| OD-10 | Sme li se verzija modela sačuvati sa nepotpunim lineage-om? | Later |
| OD-11 | Rollback aliasa dira živu instancu ili samo pokazivač? | Later |
| OD-12 | Sme li leaderboard da meša verzije benchmarka? | Later |
| OD-13 | Naplata (billing) za komercijalne korisnike? | Later |
| OD-14 | Timovi/odeljenja unutar organizacije? | MVP → Pilot |

---

### OD-1 — Status „odbijen" za resurse

**Pitanje:** Da li uvesti zasebno terminalno stanje `rejected` (odbijen, bez ponovnog slanja), različito od `draft` (vraćen na doradu)?
**Blokira:** CAT-UC-004 (sada „Odbij" i „Vrati na doradu" oba vode u `draft`); nedosledno sa MOD-UC-011 koji već koristi stanje „Odbijen".
**Opcije:**
- (a) zadržati jedinstveno `draft` za oba ishoda (trenutno stanje);
- (b) **uvesti zaseban `rejected` ≠ `draft`** — *preporuka*: usklađuje dataset-review sa model-review tokom i ostavlja auditu trag o odbijanju.

**Faza:** Later (blokira samo review/approval tok, koji je ionako Pilot).

---

### OD-2 — Masovno podešavanje vidljivosti kataloga

**Pitanje:** Da li u MVP uvesti grupno skidanje/podešavanje vidljivosti (selekcija više resursa, grupna primena, izveštaj o preskočenima)?
**Blokira:** ne blokira postojeći tok — pojedinačno skidanje (CAT-UC-005) radi i bez ovoga; otvara potrebu za novim use case-om.
**Opcije:**
- (a) **ne uvoditi u MVP** — pojedinačno skidanje dovoljno za moderaciju (*preporuka*);
- (b) modelovati zaseban use case sa svojim ekranom i logikom.

**Faza:** Later.

---

### OD-3 — Serving model

**Pitanje:** Deljeni upravljani serving pool (korisnik samo poziva gateway) ili dedicated/self-service deploj (korisnik diže sopstveni endpoint)?
**Blokira:** INF-UC-001, INF-UC-002, INF-UC-004, INF-UC-005.
**Opcije:**
- (a) **deljeni upravljani pool** — *preporuka*: `funkcionalnosti.md` 2 predviđa MVP „1 serving", endpoint je interan i nedostupan korisniku;
- (b) self-service/dedicated deploj;
- (c) hibrid: deljeni pool u MVP, self-service kasnije iza feature flag-a.

**Faza:** MVP (nadređena odluka — utiče na OD-6, OD-7, OD-11).

---

### OD-4 — Prag rutiranja Kubernetes ↔ SLURM

**Pitanje:** Po čemu se meri „veličina posla" i gde je prag za izbor klastera?
**Blokira:** INF-UC-002, AIF-UC-001.
**Opcije:**
- (a) numerički prag (procenjeni GPU sati / veličina ulaza) — traži definiciju praga;
- (b) **rutiranje po tipu posla** — interaktivno/kratko → Kubernetes GPU pool, batch/veliko → SLURM (*preporuka*, `popis_final.md` sek. 15);
- (c) korisnik bira između dopuštenih klastera (uz compute-to-data ograničenje).

**Faza:** MVP.

---

### OD-5 — Nivoi kvote

**Pitanje:** Uvode li se korisnički podlimiti kvote ili samo budžet institucije uz atribuciju potrošnje po korisniku?
**Blokira:** INF-UC-003, INF-UC-001, INF-UC-002; povezano sa ORG-UC-010 i AIF-UC-005.
**Opcije:**
- (a) samo budžet institucije + atribucija po korisniku u logovima (jednostavnije);
- (b) **institucija + korisnički podlimiti**, gde svaki nivo i prozor moraju nezavisno da prođu — *preporuka*: drugi scenariji već modeluju ličnu kvotu člana.

**Faza:** MVP.

---

### OD-6 — Self-service deploj u MVP-u

**Pitanje:** Postoji li self-service deploj modela kao endpoint u MVP-u i pod kojim uslovima?
**Blokira:** INF-UC-004.
**Opcije:**
- (a) **ne u MVP** — deploj ostaje odobren platformski servis nad deljenim pool-om (*preporuka*);
- (b) self-service uz obavezno odobrenje administratora + pravilo da endpoint nije vidljiviji od modela;
- (c) self-service samo uz proveru kvote/kapaciteta.

**Faza:** Later (zavisi od OD-3).

---

### OD-7 — Vlasništvo serving lifecycle-a

**Pitanje:** Kome pripada serving lifecycle (deploj + zamena verzije) i ko proverava API-kompatibilnost nove verzije?
**Blokira:** INF-UC-005.
**Opcije:**
- ceo serving lifecycle treba da pripadne **istom akteru** (*preporuka*);
- za API-kompatibilnost: (b) **automatska provera** uz deklarisanu ulazno/izlaznu šemu po verziji (*preporuka*) ili (c) ručna provera.

**Faza:** Later (zavisi od OD-3).

---

### OD-8 — ABAC i vidljivost kataloga

**Pitanje:** Da li vidljivost resursa u katalogu već uračunava ABAC atribute (resurs koji korisnik ne sme da koristi se ne prikazuje) ili se ABAC proverava tek pri akciji?
**Blokira:** MOD-UC-003, CAT-UC-002, INF-UC-001.
**Opcije:**
- (a) **vidljivost uračunava ABAC + provera i pri akciji** (defense-in-depth) — *preporuka*: `funkcionalnosti.md` 7 traži da sektorski filter nikad ne propusti neodobren resurs;
- (b) katalog prikazuje resurs, ABAC tek pri akciji (rizik otkrivanja postojanja);
- (c) osetljivi skriveni, ostali vidljivi uz proveru pri akciji.

**Faza:** MVP.

---

### OD-9 — Experiment tracking preko izolovane veze

**Pitanje:** Šta radi experiment tracking kad je alat nedostupan preko delimično izolovane veze ka klasteru tokom posla?
**Blokira:** MOD-UC-004; posredno MOD-UC-007.
**Opcije:**
- (a) **alat koji buffer-uje i po povratku veze sinhronizuje** — *preporuka*: ovo je eliminacioni kriterijum za izbor MLflow-a (`funkcionalnosti.md` 3);
- (b) `run` ostaje delimičan uz oznaku, bez naknadne sinhronizacije.

**Faza:** Later.

---

### OD-10 — Lineage politika

**Pitanje:** Sme li se verzija modela sačuvati sa nepotpunim lineage-om?
**Blokira:** MOD-UC-006 (grana nepotpunog lineage-a); posredno MOD-UC-005.
**Opcije:**
- (a) **dozvoliti snimanje uz oznaku `lineage: nepotpun` u draftu, uz obaveznu dopunu pre objave** — *preporuka*: konzistentno sa MOD-UC-005 koji već koristi `lineage: nepotvrđen`;
- (b) blokirati snimanje dok se lineage ne dopuni.

**Faza:** Later.

---

### OD-11 — Šta dira rollback aliasa

**Pitanje:** Da li rollback servisnog aliasa dira živu serviranu instancu ili samo pokazivač u registru?
**Blokira:** MOD-UC-010.
**Opcije:**
- (a) **rollback menja samo alias/pokazivač u registru, ne dira živu instancu** — *preporuka* za MVP (self-service deployment ionako nije u MVP);
- (b) rollback prebacuje i živu serviranu instancu (zahteva da model deployment bude u MVP).

**Faza:** Later (zavisi od OD-3 i OD-6).

---

### OD-12 — Mešanje verzija benchmarka na leaderboard-u

**Pitanje:** Sme li leaderboard da meša rezultate sa različitih verzija benchmarka?
**Blokira:** MOD-UC-013.
**Opcije:**
- (a) **ne dozvoliti mešanje** — jedan leaderboard = jedna verzija benchmarka (*preporuka*: inače poređenje gubi smisao);
- (b) dozvoliti mešanje uz jasnu oznaku verzije po redu.

**Faza:** Later.

---

### OD-13 — Naplata (billing)

**Pitanje:** Da li platforma uvodi model naplate (billing) za komercijalne korisnike (SME/startup)?
**Blokira:** ne blokira nijedan MVP use case; tiče se svih koji troše kvotu (INF-UC-001, INF-UC-002, AIF-UC-001, INF-UC-003, USG-UC-001).
**Opcije:**
- (a) **nema naplate u MVP/Pilot** — kvote su administrativna kontrola, ne finansijska transakcija (*preporuka za sada*);
- (b) pay-per-use za SME/startup uz billing servis (faktura, payment gateway, kredit);
- (c) prepaid kredit — organizacija kupuje kredit i troši ga kroz kvotu.

Merenje već postoji (GPU sati i tokeni se beleže po korisniku/projektu), ali finansijska transakcija traži zaseban billing servis, payment provider i pravni okvir — ADR sa pravnim timom pre implementacije.
**Faza:** Later (značajna pravna i operativna složenost).

---

### OD-14 — Timovi/odeljenja unutar organizacije

**Pitanje:** Da li se uvode timovi/odeljenja kao pod-entiteti unutar organizacije, i kada?
**Blokira:** ORG-UC-011 (Kreiranje tima); posredno svi use case-ovi sa `team` vidljivošću i kvotom.
**Opcije:**
- (a) **MVP: ravna organizacija** (organizacija → član), bez timova — jednostavnije (*preporuka za MVP*, `popis_final.md` §8a);
- (b) Pilot: organizacija → tim (delegirana kvota, lider tima) → član — za institucije sa jasnom sub-strukturom (fakultet → katedra → istraživačka grupa);
- (c) odmah u MVP ako upitnik pokaže da rad bez timova ne funkcioniše.

Pilot varijanta traži tronivovsku kvotu i nove use case-ove za lidera tima.
**Faza:** MVP (a) → Pilot (b) ako upitnik potvrdi potrebu.

---

## Zavisnosti između odluka

- **OD-3 je nadređena.** Od njenog ishoda zavise OD-6 (ima li self-service deploja), OD-7 (vlasništvo serving lifecycle-a) i OD-11 (da li rollback dira živu instancu).
- **OD-6 → OD-11.** Pitanje da li rollback dira živu instancu postoji samo ako self-service deploj endpointa uopšte postoji.
