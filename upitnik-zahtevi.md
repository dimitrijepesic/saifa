# SAIFA AI Gateway — Upitnik za prikupljanje zahteva (Faza 0)

Upitnik prati listu otvorenih pitanja iz projektnog zadatka, ali je podeljen u tri dela tako da prvo otkriva **problem**, pa tek onda mapira na **mogućnosti** platforme (detalji u `funkcionalnosti.md`). Razlog: korisnici često ne znaju da li im treba OpenSearch, pgvector ili LMS — ali znaju koji dataset, model, tim, rok i ograničenje imaju.

- **Deo A — Use-case discovery:** šta želite da postignete?
- **Deo B — Operational reality:** šta već imate i šta vas ograničava?
- **Deo C — Gateway capabilities:** koje funkcije su potrebne i kada?

Cilj: popunjen upitnik → rang-lista za MVP scope. Svaka oblast u Delu C ima MoSCoW prioritet (Must / Should / Could / Won't now) i procenu učestalosti/obima, što daje merljiv ulaz za prioritizaciju umesto "puno teksta".

Postupak: upitnik popuniti u razgovoru sa svakim klijentom/sektorom posebno (zdravstvo, energetika, ekologija, jezik/kultura), pa odgovore ukrstiti sa opcijama iz `funkcionalnosti.md` i doneti odluke o tehnologijama za MVP.

---

# Deo A — Use-case discovery (šta želite da postignete)

## A1. Najvažniji use-case-ovi

- Pitanje: Koja su vaša 3 najvažnija AI use-case-a za narednih 12 meseci?
- Odgovor klijenta: ___

## A2. Očekivani rezultat

- Pitanje: Koji rezultat očekujete od Gateway-a: model, inference API, izveštaj, dashboard, dataset, publikacija, ili gotov servis?
- Odgovor klijenta: ___

## A3. Učestalost korišćenja

- Pitanje: Koliko često bi se Gateway koristio: dnevno, nedeljno, povremeno? Koliko korisnika i timova?
- Odgovor klijenta: ___

## A4. Eksperimentisanje vs. produkcija

- Pitanje: Da li vam je potreban production deployment (stalno dostupan servis) ili samo experimentation (istraživanje, prototip)?
- Odgovor klijenta: ___

## A5. Rok za rezultat

- Pitanje: Koji je prihvatljiv rok za dobijanje rezultata HPC posla (sati, dan, nedelja)?
- Odgovor klijenta: ___

---

# Deo B — Operational reality (šta imate i šta vas ograničava)

## B1. Podaci koje već imate

- Pitanje: Koji datasetovi već postoje, u kom formatu, ko je vlasnik, koja licenca važi? Da li imate anotacije, benchmark ili test setove?
- Odgovor klijenta: ___

## B2. Kretanje podataka

- Pitanje: Da li podaci smeju da napuste instituciju? Ima li podataka sa posebnim režimom pristupa (lični, zdravstveni, poverljivi)?
- Odgovor klijenta: ___

## B3. Postojeći sistemi i kapacitet

- Pitanje: Da li imate sopstveni IAM/SSO? Interni DevOps kapacitet? Postojeće sisteme sa kojima se Gateway mora integrisati?
- Odgovor klijenta: ___

## B4. Pravni i compliance kontakt

- Pitanje: Da li imate DPO / legal kontakt? Ko odobrava objavu modela/dataseta sa vaše strane?
- Odgovor klijenta: ___

## B5. Ograničenja

- Pitanje: Koja su vaša ograničenja: rokovi, budžet, osetljivost podataka, regulativa? Ko je product owner sa strane sektora?
- Odgovor klijenta: ___

## B6. Nivo tehničkog znanja

- Pitanje: Koji je nivo tehničkog znanja vaših korisnika (eksperti, srednji, početnici/studenti)? Koje pismo i jezik koriste?
- Odgovor klijenta: ___

---

# Deo C — Gateway capabilities (koje funkcije i kada)

Za svaku oblast: pitanje, opcije koje smo identifikovali, prostor za odgovor, MoSCoW prioritet i procena učestalosti/obima.

## C1. Korisnici i uloge

- Pitanje: Ko su vaši korisnici i koliko ih očekujete? (istraživači, SME/startup, javni sektor, studenti, mentori, administratori, EuroHPC partneri)
- Naše opcije: 10 definisanih uloga (platform-admin → viewer), različit UI po ulozi
- Odgovor klijenta: ___
- Prioritet (MoSCoW): ___ · Učestalost / obim: ___

## C2. Autentikacija

- Pitanje: Kako se korisnici prijavljuju? Da li vaša institucija ima sopstveni identity sistem?
- Naše opcije: lokalni nalog · institucionalni SSO preko eduGAIN/AMRES · EuroHPC identitet (MyAccessID) · sve preko Keycloak-a (OIDC/OAuth2/SAML2)
- Odgovor klijenta: ___
- Prioritet (MoSCoW): ___ · Učestalost / obim: ___

## C3. Autorizacija

- Pitanje: Ko sme da vidi, preuzme, pokrene, objavi ili deli dataset/model/workflow? Postoje li podaci sa posebnim režimom pristupa?
- Naše opcije: RBAC uloge + ABAC atributi (sektorske sertifikacije, afilijacija, GDPR saglasnost) + vidljivost public/private/team
- Odgovor klijenta: ___
- Prioritet (MoSCoW): ___ · Učestalost / obim: ___

## C4. Katalog resursa

- Pitanje: Koje tipove resursa treba katalogizovati i koje metapodatke morate imati? (licenca, jezik, domen, poreklo...)
- Naše opcije: modeli, datasetovi, workflow-i, sveske, kontejneri, kursevi; DCAT-usklađena šema; pretraga OpenSearch + semantička (pgvector)
- Odgovor klijenta: ___
- Prioritet (MoSCoW): ___ · Učestalost / obim: ___

## C5. Pristup podacima (data access)

- Pitanje: Da li korisnici preuzimaju podatke, pristupaju im kroz API, ili podaci ne smeju da napuste kontrolisano okruženje?
- Naše opcije: download (presigned URL) · API pristup · compute-to-data (obrada samo na nacionalnim klasterima, podaci se ne sele) · sandbox sa read-only pristupom
- Odgovor klijenta: ___
- Prioritet (MoSCoW): ___ · Učestalost / obim: ___

## C6. Model registry

- Pitanje: Da li vam treba samo pregled modela ili pun lifecycle (verzionisanje, evaluacija, deployment, rollback)?
- Naše opcije: registracija (HF import / upload / eksterni API) · verzionisanje sa lineage-om · evaluacioni status · deployment status · rollback
- Odgovor klijenta: ___
- Prioritet (MoSCoW): ___ · Učestalost / obim: ___

## C7. HPC pristup

- Pitanje: Kako vaši korisnici žele da pokreću poslove: forma u portalu, Jupyter sveska, API, ili gotovi šabloni? Koji nivo tehničkog znanja imaju?
- Naše opcije: submit forma (ekspert) · SDK iz sveske · raw API · zaključani šabloni (studenti/početnici); SSH+sbatch ili slurmrestd ka PARADOX/ITE
- Odgovor klijenta: ___
- Prioritet (MoSCoW): ___ · Učestalost / obim: ___

## C8. Deployment

- Pitanje: Šta treba da se deploy-uje i kako se koristi: stalno dostupan inference API, chat, batch obrada, kontejneri?
- Naše opcije: inference API (OpenAI-kompatibilan) · chat UI · batch poslovi na HPC · kontejnerizovani poslovi (Apptainer) · ML pipeline-ovi
- Odgovor klijenta: ___
- Prioritet (MoSCoW): ___ · Učestalost / obim: ___

## C9. Benchmarking

- Pitanje: Koje metrike i evaluacije su vam bitne? Da li vam treba javni leaderboard? Imate li sopstvene test setove?
- Naše opcije: lm-evaluation-harness · sektorski benchmark datasetovi (uklj. srpski jezik) · leaderboard sa procesom odobravanja · validation reports
- Odgovor klijenta: ___
- Prioritet (MoSCoW): ___ · Učestalost / obim: ___

## C10. Obuke (training platforma)

- Pitanje: Da li vam treba integrisan LMS sa kursevima i sertifikatima, ili samo biblioteka tutorijala/svezaka? Na kom jeziku?
- Naše opcije: Open edX ili Moodle iza SAIFA API-ja · hands-on labovi (sveska + pravi podaci + HPC) · learning paths · sertifikati · srpski (ćir./lat.) i engleski
- Odgovor klijenta: ___
- Prioritet (MoSCoW): ___ · Učestalost / obim: ___

## C11. Kolaboracija

- Pitanje: Da li korisnici rade u timovima/projektima? Treba li deljenje artefakata, komentari, formalni review/approval proces? Ko odobrava objavu?
- Naše opcije: timski workspace-ovi · team-scoped vidljivost · review/approval tokovi · komentari i validation reports · GitLab za kod
- Odgovor klijenta: ___
- Prioritet (MoSCoW): ___ · Učestalost / obim: ___

## C12. EuroHPC federacija

- Pitanje: Koje resurse treba deliti sa Pharos/IT4LIA (izvoz) i koje eksterne resurse vaši korisnici trebaju (uvoz)? Postoje li ograničenja deljenja?
- Naše opcije: katalog sync (zakazan + event-driven) · izvoz modela/datasetova/benchmarka · uvoz sa oznakom porekla · adapter po partneru
- Odgovor klijenta: ___
- Prioritet (MoSCoW): ___ · Učestalost / obim: ___

## C13. Monitoring

- Pitanje: Šta morate da pratite i izveštavate: potrošnju GPU sati, energiju, troškove, status poslova, preuzimanja, korišćenje modela?
- Naše opcije: Prometheus/Grafana dashboardi (GPU sati, energija, kvote, latencija) · usage izveštaji po korisniku/projektu · mrežni nadzor linkova (NetFlow/IPFIX/sFlow)
- Odgovor klijenta: ___
- Prioritet (MoSCoW): ___ · Učestalost / obim: ___

## C14. Compliance (GDPR i licence)

- Pitanje: Da li radite sa ličnim/osetljivim podacima (posebno zdravstvenim)? Ko je vaš DPO? Koje licence važe za vaše podatke i modele?
- Naše opcije: 4 nivoa klasifikacije · evidencija saglasnosti · DSAR izvoz · pravo na brisanje · anonimizacija · DPIA procedura · audit log (2+ godine) · provera licenci pre objave
- Odgovor klijenta: ___
- Prioritet (MoSCoW): ___ · Učestalost / obim: ___

---

## Kako se odgovori prevode u MVP scope (scoring)

Posle popunjavanja, svaka oblast iz Dela C dobija orijentacionu ocenu na osnovu tri ulaza, da bi se dobila rang-lista umesto utiska:

- MoSCoW prioritet: Must = visok, Should = srednji, Could = nizak, Won't now = van scope-a.
- Učestalost / obim: dnevno/mnogo timova diže prioritet; povremeno/jedan tim ga spušta.
- Zrelost (Deo B): ako klijent već ima dataset, anotacije, DPO i jasan cilj, funkcionalnost je spremna za rani MVP; ako ne, ide u kasniju fazu uz pripremu.

Oblasti koje su Must + česte + spremne ulaze u MVP; ostale se raspoređuju po fazama (Pilot / Production / Later) prema tabeli prioriteta u `funkcionalnosti.md`.
