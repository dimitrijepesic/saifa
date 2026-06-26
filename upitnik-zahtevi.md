# SAIFA AI Gateway — Upitnik za prikupljanje zahteva


Struktura:
- **Deo A — Profil i zrelost klijenta** (segmentacija, postojeće stanje, prepreke). Određuje koliko od platforme klijent uopšte može da koristi i šta mu je realan ulaz.
- **Deo B — Funkcionalni zahtevi po modulu** (opcije iz `funkcionalnosti.md`/`popis_final.md`, vezane za use case-ove iz `SAIFA_use_cases_final.md`). Gde odgovor klijenta služi kao ulaz za arhitektonsku odluku, naveden je ID `OD-X`.
- **Deo C — Prioritizacija, obim i uspeh** (MoSCoW koji `mvp_kriterijumi.md` očekuje, procena obima, kriterijumi prihvatanja).


## Deo A — Profil i zrelost klijenta

Cilj Dela A: segmentirati klijenta i utvrditi realan ulaz pre nego što se priča o funkcijama. Bez ovoga se modul-pitanja popunjavaju u vakuumu.

### A1. Tip i sektor organizacije

- Pitanje: Kojem tipu pripadate i u kom sektoru radite?
- Naše opcije: akademija/istraživanje · javna ustanova · SME/startup · velika firma · NVO; sektor: jezik i kultura · zdravstvo · poljoprivreda · održivost · drugo
- Odgovor klijenta: ___

### A2. Korisnici i obim

- Pitanje: Koliko ljudi iz vaše organizacije bi koristilo platformu i u kojim ulogama? (istraživači, inženjeri, studenti, mentori, donosioci odluka)
- Naše opcije: pojedinac · mali tim (<10) · odeljenje (10–50) · cela organizacija (50+); mapira se na 10 uloga iz IAM modela
- Odgovor klijenta: ___

### A3. Samoprocena AI/HPC zrelosti

- Pitanje: Gde ste danas u radu sa AI i superračunarima?
- Naše opcije (lestvica): (0) ne radimo AI još · (1) koristimo gotove AI servise/API-je · (2) treniramo/fine-tune-ujemo na cloud GPU · (3) imamo sopstveni GPU/klaster · (4) iskusni HPC korisnici (SLURM, kontejneri)
- Zašto pitamo: nivo određuje podrazumevani UI (zaključani šabloni vs. raw API) i obim podrške.
- Odgovor klijenta: ___

### A4. Trenutna primarna AI aktivnost

- Pitanje: Šta sada radite (ili planirate prvo) sa AI?
- Naše opcije: trening modela od nule · fine-tuning postojećih · inference u produkciji · analiza/priprema podataka · evaluacija/benchmark · još uvek istražujemo
- Odgovor klijenta: ___

### A5. Postojeća infrastruktura i alati

- Pitanje: Šta trenutno koristite za AI radne tokove i odakle vam compute?
- Naše opcije: komercijalni API (OpenAI/Anthropic/…) · sopstveni GPU/server · nacionalni/akademski HPC · javni cloud (AWS/GCP/Azure) · ništa zasad; alati: Jupyter · MLflow/W&B · sopstveni skripti · ništa
- Zašto pitamo: reuse/migracija — šta već imamo da povežemo umesto da gradimo, i šta klijent mora da „napusti".
- Odgovor klijenta: ___

### A6. Veštine i nedostaci (skill gap)

- Pitanje: Koje od ovih veština imate interno, a gde vam treba podrška?
- Naše opcije (po stavci: imamo / delimično / nemamo): rad sa SLURM/HPC · ML inženjering (trening/fine-tuning) · MLOps/deployment · priprema i upravljanje podacima · GDPR/usklađenost
- Zašto pitamo: nedostatak HPC ekspertize je glavna prepreka usvajanju; određuje obim obuke i „managed" vs. „self-service" pristup.
- Odgovor klijenta: ___

### A7. Glavne prepreke za usvajanje

- Pitanje: Šta vas danas najviše sprečava da koristite AI/HPC? (rangirajte top 3)
- Naše opcije: trošak · tehnička složenost · nedostatak veština · pristup GPU resursima · bezbednost/poverljivost podataka · nemamo upotrebljive podatke
- Odgovor klijenta: ___

---

## Deo B — Funkcionalni zahtevi po modulu

Po modulu: pitanje za klijenta, opcije koje smo identifikovali, vezani use case, i (gde je primenljivo) otvorena odluka koju odgovor klijenta pomaže da se zatvori.

### B1. Korisnici i uloge

- Pitanje: Koje od 10 definisanih uloga vam stvarno trebaju i da li sve prijavljene osobe imaju iste mogućnosti ili se razlikuju po pravu pristupa?
- Naše opcije: 10 uloga (platform-admin → viewer); princip: iste mogućnosti osim kad se pravo pristupa stvarno razlikuje
- → use case: sek. 1 (Pristup platformi i nalozi)
- Odgovor klijenta: ___

### B2. Autentikacija

- Pitanje: Kako se korisnici prijavljuju? Imate li sopstveni identity sistem (eduGAIN/AMRES članstvo)?
- Naše opcije: lokalni nalog · institucionalni SSO (eduGAIN/AMRES) · EuroHPC identitet (MyAccessID) · sve preko Keycloak-a (OIDC/OAuth2/SAML2)
- → use case: sek. 1 (prijava)
- Odgovor klijenta: ___

### B3. Autorizacija i vidljivost

- Pitanje: Ko sme da vidi, preuzme, pokrene, objavi ili deli resurs? Postoje li podaci koje deo korisnika ne sme ni da *vidi* da postoje?
- Naše opcije: RBAC uloge + ABAC atributi (sektorske sertifikacije, afilijacija, GDPR saglasnost) + vidljivost public/private/team
- → use case: sek. 1, sek. 2; **→ ulaz za OD-8** (da li katalog uračunava ABAC u vidljivost, ili se ABAC proverava tek pri akciji)
- Odgovor klijenta: ___

### B4. Katalog resursa

- Pitanje: Koje tipove resursa katalogizujemo i koji metapodaci su vam obavezni (licenca, jezik, domen, poreklo)?
- Naše opcije: modeli, datasetovi, workflow-i, sveske, kontejneri, kursevi; DCAT-usklađena šema; pretraga OpenSearch + semantička (pgvector)
- → use case: sek. 2 (Katalog), sek. 6 (Datasetovi)
- Odgovor klijenta: ___

### B5. Pristup podacima (data access)

- Pitanje: Da li korisnici preuzimaju podatke, pristupaju im preko API-ja, ili podaci ne smeju da napuste kontrolisano okruženje?
- Naše opcije: download (presigned URL) · API pristup · compute-to-data (obrada samo na nacionalnim klasterima, podaci se ne sele) · sandbox sa read-only pristupom
- → use case: sek. 6; **→ ulaz za OD-8**
- Odgovor klijenta: ___

### B6. Model registry i lifecycle

- Pitanje: Treba li vam samo pregled modela ili pun lifecycle (verzionisanje, evaluacija, deployment, rollback)?
- Naše opcije: registracija (HF import / upload / eksterni API) · verzionisanje sa lineage-om · evaluacioni status · deployment status · rollback
- → use case: sek. 5 (Model lifecycle); interne odluke OD-9 (experiment tracking preko izolovane veze), OD-10 (lineage politika), OD-11 (rollback semantika) — rešavaju se PoC-om, ne ovde
- Odgovor klijenta: ___

### B7. HPC pristup i pokretanje poslova

- Pitanje: Kako korisnici žele da pokreću poslove (forma u portalu, Jupyter sveska, API, gotovi šabloni) i koliki im je tipičan posao?
- Naše opcije: submit forma (ekspert) · SDK iz sveske · raw API · zaključani šabloni (studenti/početnici); SSH+sbatch ili slurmrestd ka PARADOX/ITE
- → use case: sek. 4 (AI factory i pokretanje poslova); **→ ulaz za OD-4** (prag rutiranja Kubernetes ↔ SLURM — zavisi od tipičnih veličina posla iz Dela C2) i **→ ulaz za OD-5** (kvotni model)
- Odgovor klijenta: ___

### B8. Serviranje modela i pristupne površine

- Pitanje: Preko čega korisnici koriste modele? Treba li im hostovani chat (tip ChatGPT/Claude), i da li žele da povezuju **spoljne agentske/coding alate** (Claude Code, Cursor, Copilot, SDK) na platformske modele, ili im je dovoljna platformska površina?
- Naše opcije: hostovani chat UI · Jupyter sveska · OpenAI-kompatibilan API · Anthropic-kompatibilan API (Claude Code) · spoljni agentski/coding klijenti preko gateway-a · batch poslovi na HPC · kontejnerizovani poslovi (Apptainer)
- → use case: sek. 3 (Inference); **→ ulaz za OD-3** (deljeni upravljani pool vs. self-service deploj) i **→ ulaz za OD-6**
- Napomena (ref. Pharos/Themelio): svi klijenti idu kroz jedan upravljani gateway, bez direktnog pristupa serving endpointu; ako klijent traži spoljne agentske klijente, gateway mora da izloži provider-kompatibilne facade (`/v1/chat/completions`, `/v1/messages`) — proširuje API zahteve preko „OpenAI-kompatibilan".
- Odgovor klijenta: ___

### B9. Benchmarking i evaluacija

- Pitanje: Koje metrike su vam bitne? Treba li javni leaderboard? Imate li sopstvene test setove (uklj. srpski jezik)?
- Naše opcije: lm-evaluation-harness · sektorski benchmark datasetovi · leaderboard sa procesom odobravanja · validation reports
- → use case: sek. 5 (evaluacija); interna odluka OD-12 (mešanje verzija benchmarka) — ne pita se klijent
- Odgovor klijenta: ___

### B10. Obuke (training platforma)

- Pitanje: Treba li vam integrisan LMS sa kursevima i sertifikatima, ili biblioteka tutorijala/svezaka? (Format i jezik vidi C4.)
- Naše opcije: Open edX/Moodle iza SAIFA API-ja · hands-on labovi (sveska + pravi podaci + HPC) · learning paths · sertifikati
- → use case: sek. 8 (Edukacija i kursevi)
- Odgovor klijenta: ___

### B11. Kolaboracija i review

- Pitanje: Radite li u timovima/projektima? Treba li deljenje artefakata, komentari, formalni review/approval proces pre objave?
- Naše opcije: timski workspace-ovi · team-scoped vidljivost · review/approval tokovi · komentari i validation reports · GitLab za kod
- → use case: sek. 7 (Kolaboracija); interna odluka OD-1 (zasebno `rejected` stanje u review toku) — ne pita se klijent
- Odgovor klijenta: ___

### B12. EuroHPC federacija

- Pitanje: Koje resurse delite sa Pharos/IT4LIA (izvoz) i koji eksterni resursi vam trebaju (uvoz)? Postoje li ograničenja deljenja?
- Naše opcije: katalog sync (zakazan + event-driven) · izvoz modela/datasetova/benchmarka · uvoz sa oznakom porekla · adapter po partneru
- → use case: sek. 10 (Federacija)
- Odgovor klijenta: ___

### B13. Monitoring i kvote

- Pitanje: Šta morate da pratite i izveštavate? Na kom nivou se dodeljuje kvota — **organizacija / projekat / korisnik** — i da li je dovoljna atribucija potrošnje po korisniku u logovima, ili treba tvrd korisnički podlimit?
- Naše opcije: dashboardi (GPU sati, energija, kvote, latencija) · usage po korisniku/projektu · mrežni nadzor (NetFlow/IPFIX/sFlow). Dve vrste kvote: (1) **compute** — GPU/CPU sati za poslove i trening; (2) **inference** — token budžet sa vremenskim prozorima (npr. 5h / nedelja / mesec).
- → use case: sek. 9 (Administracija, monitoring); **→ ulaz za OD-5**
- Napomena (ref. Pharos/Themelio): token budžet je deljeni pool na nivou *projekta* (svi članovi troše iz istog), uz atribuciju po korisniku — direktan kandidat za rešenje OD-5; razdvajanje compute/inference kvote je njihova zasebna odluka, ne jedinstveni „GPU sat".
- Odgovor klijenta: ___

### B14. Compliance (GDPR i licence)

- Pitanje: Radite li sa ličnim/osetljivim podacima (posebno zdravstvenim)? Ko je vaš DPO (vidi C5)? Koje licence važe za vaše podatke i modele? Ako koristite hostovani chat — da li je potrebna istorija razgovora i upload fajlova, i kakva retencija/brisanje?
- Naše opcije: 4 nivoa klasifikacije · evidencija saglasnosti · DSAR izvoz · pravo na brisanje · anonimizacija · DPIA procedura · audit log (2+ godine) · provera licenci pre objave; retencija istorije razgovora i otpremljenih fajlova kao zasebna odluka
- → use case: sek. 9 (usklađenost)
- Odgovor klijenta: ___

---

## Deo C — Prioritizacija, obim i uspeh

### C1. MoSCoW prioriteti

`mvp_kriterijumi.md` koristi ovu kolonu za scenarije sa 0 tehničkih kriterijuma. Napomena klijentu: MoSCoW nije obećanje redosleda — tehnički MVP kriterijumi (K1–K5) mogu da pomere stavku u Pilot bez obzira na „Must". Klijent označava prioritet po sposobnosti: **M** (must), **S** (should), **C** (could), **W** (won't now).

| Sposobnost | M | S | C | W |
| --- | --- | --- | --- | --- |
| Prijava i pristup katalogu | | | | |
| Inference (poziv gotovog modela) | | | | |
| Hostovani chat / spoljni agentski klijenti | | | | |
| Pokretanje HPC posla | | | | |
| Fine-tuning sopstvenog modela | | | | |
| Upload i upravljanje datasetovima | | | | |
| Compute-to-data nad osetljivim podacima | | | | |
| Model registry i verzionisanje | | | | |
| Benchmarking / leaderboard | | | | |
| Self-service deploj endpointa | | | | |
| Timski rad i review proces | | | | |
| Kursevi i obuke (LMS) | | | | |
| Federacija sa Pharos/IT4LIA | | | | |
| Monitoring potrošnje i izveštaji | | | | |

### C2. Obim i učestalost

- Pitanje: Koliki su vam tipični i maksimalni poslovi?
- Naše opcije: veličina dataseta (<1 GB · 1–100 GB · 100 GB–1 TB · >1 TB) · veličina modela (<1B · 1–10B · 10–70B · >70B parametara) · procena GPU sati mesečno (compute) · procena potrošnje tokena mesečno (inference) · broj poslova nedeljno · interaktivno vs. batch
- Zašto pitamo: ujedno ulaz za **OD-4** (gde je realan prag K8s↔SLURM) i za dimenzionisanje obe vrste kvote (compute vs. token budžet).
- Odgovor klijenta: ___

### C3. Kriterijumi uspeha

- Pitanje: Šta platforma mora da uradi da biste je smatrali korisnom? Šta je pilot uspeh, a šta produkcioni?
- Naše opcije (primer): „možemo da pokrenemo X bez sopstvenog GPU" · „fine-tune na našim podacima za <Y dana" · „podaci nikad ne napuštaju klaster" · „studenti rade lab vežbe bez setup-a"
- Odgovor klijenta: ___

### C4. Jezik i format podrške/obuke

- Pitanje: Na kom jeziku i u kom formatu vam treba dokumentacija, podrška i obuka?
- Naše opcije: jezik (srpski latinica · srpski ćirilica · engleski); format (pisana dokumentacija · video · radionice uživo · hands-on labovi · konsultacije 1-na-1)
- Odgovor klijenta: ___

### C5. Kontakt i DPO

- Pitanje: Ko je tehnički kontakt za follow-up, a ko DPO za pitanja zaštite podataka?
- Naše opcije: tehnički kontakt (ime/email) · DPO (ime/email)
- Odgovor klijenta: ___

