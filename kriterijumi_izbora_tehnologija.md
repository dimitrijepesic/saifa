# SAIFA — Kriterijumi za izbor tehnologija

Ovaj dokument definiše evaluacione kriterijume po tehnološkom domenu. Kriterijumi važe **pre** izbora konkretne tehnologije: kada tim izađe pred listu kandidata (`popis_final.md`), prema ovome se meri da li kandidat zadovoljava ono što use case-ovi traže.

Kriterijumi se izvode iz dva izvora:

- **Funkcionalni zahtevi** — šta sistem mora da radi, iz scenarija u `SAIFA_use_cases_final.md` (referenca: broj scenarija i sekcija).
- **MVP scope** — šta mora da radi odmah, iz dimenzija K1–K5 u `mvp_kriterijumi.md` i eliminacionih kriterijuma iz `funkcionalnosti.md`.

Svaki kriterijum ima **test-pitanje** (po uzoru na K1–K5): pitanje na koje kandidat odgovara „da" ili „ne", čime kriterijum ostaje upotrebljiv kao rubrika, a ne truizam. Kriterijumi ne donose arhitektonske odluke (to radi `otvorene_odluke.md`) i ne biraju tehnologiju — samo definišu prema čemu se bira.

Oznaka **[eliminacioni]** stoji uz kriterijum koji `funkcionalnosti.md` tretira kao tvrd uslov: kandidat koji ne prolazi ispada iz izbora bez obzira na ostalo.

---

## 1. Autentikacija i kontrola pristupa

Domen pokriva: prijavu (lokalni / institucionalni / EuroHPC nalog), API ključeve, RBAC uloge, ABAC atribute, opoziv sesija, servis-servis autentikaciju.

### Funkcionalni zahtevi

- **Tri nezavisna puta prijave** iz jedne tačke: lokalni nalog (sc. „Registracija lokalnim nalogom"), institucionalni SSO preko eduGAIN/AMRES (sc. „Prijava institucionalnim SSO-om"), EuroHPC identitet preko MyAccessID (sc. „Prijava EuroHPC identitetom"). Sva tri vode istog korisnika u isti pogled po ulozi.
- **Brokering eksternih identiteta**: SAML2 ka AMRES-u, OIDC ka EuroHPC AAI; mapiranje eksternih atributa u interne uloge/atribute pri prvoj prijavi.
- **API ključevi** sa scope-om, rotacijom i opozivom koji deluje **odmah** (sc. „Kreiranje i upravljanje API ključem"; DoD `funkcionalnosti.md` 11).
- **RBAC + ABAC u jednoj odluci**: pri svakoj operaciji odluku donosi backend na osnovu uloge (platform-admin, data-admin, model-reviewer, …) i atributa (sektorska sertifikacija, nivo afilijacije, GDPR saglasnost, tenant) — sc. „Dodeljivanje uloga i ABAC atributa korisniku".
- **Prisilni opoziv sesije** pri bezbednosnom incidentu, sa trenutnim dejstvom (sc. „Prisilno opozivanje sesije korisnika").
- **Servis-servis autentikacija** bez statičkih tajni: kratkotrajni tokeni (client credentials / token exchange) — `popis_final.md` §8.

### Kriterijumi

**A1 — Federacija identiteta preko oba protokola** [eliminacioni]
Provider mora istovremeno da broker-uje SAML2 (akademska federacija) i OIDC (EuroHPC AAI) i da ih svede na jedan interni identitet.
- *Test:* Može li jedan korisnik da se prijavi i institucionalnim SSO-om i EuroHPC nalogom i da bude isti subjekt sa istim ulogama, bez dva odvojena naloga?

**A2 — Atributi iz eksternog IdP-a stižu do odluke o pristupu**
Mapiranje eksternih claim-ova (afilijacija, sektor) u interne ABAC atribute mora biti deklarativno, bez izmene koda po instituciji.
- *Test:* Kad nova institucija uđe u federaciju, da li se njeni atributi mapiraju konfiguracijom, ili zahteva intervenciju u kodu?

**A3 — Opoziv sa trenutnim dejstvom** [eliminacioni]
Opoziv API ključa i opoziv sesije moraju prekinuti pristup u realnom vremenu, ne tek po isteku tokena.
- *Test:* Posle opoziva, da li sledeći zahtev sa istim ključem/sesijom pada — ili token i dalje radi do isteka?

**A4 — Razdvajanje RBAC i ABAC bez dupliranja logike**
Uloge i atributi moraju se evaluirati u istoj tački odluke, da se izbegne ABAC u jednom servisu i RBAC u drugom (lekcija o kanonskoj logici, `popis_final.md` §13).
- *Test:* Postoji li jedna tačka u kojoj se i uloga i atribut proveravaju, ili je politika rasuta po servisima?

**A5 — Servis-servis bez statičkih tajni**
Interna komunikacija mora koristiti kratkotrajne tokene; statički deljeni secret nije prihvatljiv (zero-trust, `popis_final.md` §11).
- *Test:* Može li servis da dokaže identitet drugom servisu bez ijedne tajne upisane u konfiguraciju?

**A6 — Suvereno hostovanje**
Provider mora raditi self-hosted u domaćem okruženju; SaaS-only rešenje ne prolazi (suverenost podataka, EuroHPC kontekst).
- *Test:* Radi li potpuno unutar naše infrastrukture, bez obaveznog poziva ka eksternom servisu?

---

## 2. Katalog resursa

Domen pokriva: pretragu i filtriranje (anonimni i prijavljeni), prikaz opisa/uzorka, full-text, vidljivost po ACL-u, federisane resurse u istom prikazu.

### Funkcionalni zahtevi

- **Javna pretraga bez naloga** (sc. „Pretraga i filtriranje javnog kataloga") — one-stop ulazna tačka, full-text (`funkcionalnosti.md` 7 MVP).
- **Pretraga koja nikad ne propušta neodobren resurs**: rezultati filtrirani po ACL-u i sektorskom atributu (sc. „Pretraga i filtriranje kataloga resursa"; DoD `funkcionalnosti.md` 7: „sektorski filter nikad ne propusti neodobren resurs").
- **Heterogeni tipovi u jednom indeksu**: model, dataset, kurs, sveska, workflow — lokalni i federisani, sa oznakom porekla (sc. „Pristup federisanim resursima"; `popis_final.md` §12 source of truth).
- **Bogati metapodaci kao filteri**: domain, language, license, visibility, GDPR klasa, quality status (`popis_final.md` §12 obavezni metapodaci).
- **Semantička pretraga** kao izgledna nadogradnja nad full-text (`popis_final.md` §15 decision tree: pgvector → zaseban servis).

### Kriterijumi

**K1 — Filter po pravu pristupa na nivou indeksa** [eliminacioni]
Pretraga mora da primeni ACL/ABAC filter tako da neodobren resurs ne uđe u rezultate — ne da ga vrati pa sakrije u UI-ju.
- *Test:* Da li motor pretrage može da filtrira po atributu pristupa pre vraćanja rezultata, ili se filtriranje radi tek na klijentu (rizik curenja postojanja resursa)?

**K2 — Full-text nad srpskim i engleskim**
Indeks mora podržati analizu teksta na srpskom (latinica/ćirilica) i engleskom u istom upitu.
- *Test:* Vraća li pretraga na srpskom relevantne rezultate sa ispravnim tretmanom oblika reči, ili tretira tekst kao goli string?

**K3 — Jedan upit nad mešanim tipovima resursa**
Korisnik mora dobiti modele, datasetove, kurseve i federisane resurse iz jedne pretrage, sa oznakom tipa i porekla.
- *Test:* Može li se u jednom rezultatu naći lokalni model i federisani dataset, jasno označeni, bez dva odvojena pretraživača?

**K4 — Filteri vezani za governance metapodatke**
Polja koja governance zahteva (GDPR klasa, license, quality status, domain) moraju biti indeksirana kao filteri, ne samo prikazni tekst.
- *Test:* Mogu li se rezultati suziti po GDPR klasi i licenci, ili su ti podaci samo u opisu?

**K5 — Putanja ka semantičkoj pretrazi bez zamene motora**
Rešenje treba da omogući dodavanje vektorske pretrage kasnije bez bacanja postojećeg indeksa.
- *Test:* Može li se semantička pretraga dodati uz postojeći full-text, ili zahteva migraciju celog kataloga na drugi sistem?

---

## 3. Inference serving

Domen pokriva: poziv modela (REST/sveska), batch inference, rate limiting, streaming, kvotnu proveru. **Domen je suštinski otvoren** po pitanju serving modela (deljeni pool vs. self-service) i ne pretpostavlja ishod te odluke — kriterijumi mere obe putanje.

### Funkcionalni zahtevi

- **Poziv modela kroz gateway, nikad direktno runtime** (sc. „Pozivanje modela"; `popis_final.md` §13: serving endpoint je interan, sav saobraćaj kroz gateway radi kvota/audita).
- **Provera kvote pre izvršenja** sa atribucijom potrošnje po korisniku (sc. „Pozivanje modela" korak 4; sc. „Pokretanje batch inference posla").
- **Rate limiting po korisniku i tipu resursa** (sc. „Podešavanje rate limitinga"; `popis_final.md` §3).
- **Streaming tokena (SSE)** i upravljanje kontekstom chat sesije (`popis_final.md` §6).
- **Serving topologija kao metapodatak modela** (single-GPU / multi-GPU), promenljiva bez izmene klijenta (`popis_final.md` §6, §13).
- **Batch inference nad datasetom** sa rezultatom u skladište (sc. „Pokretanje batch inference posla nad datasetom").

### Kriterijumi

**I1 — Runtime iza gateway-a, ne izložen korisniku** [eliminacioni]
Serving runtime mora moći da radi kao interni servis čiji se saobraćaj prisilno provlači kroz gateway (kvota, audit, autorizacija).
- *Test:* Može li se runtime postaviti tako da korisnik nikako ne može da ga pozove zaobilazeći gateway?

**I2 — Promena serving topologije bez promene klijentskog ugovora**
Prelazak modela sa single-GPU na multi-GPU (ili obrnuto) ne sme da menja API koji klijent vidi.
- *Test:* Ako se model prebaci na multi-node serving, da li klijentski poziv ostaje identičan?

**I3 — Streaming tokena**
Runtime/serving sloj mora podržati inkrementalno vraćanje tokena (SSE) za interaktivni inference.
- *Test:* Vraća li odgovor token-po-token, ili tek ceo odgovor na kraju?

**I4 — Tačka za naplatu kvote i rate limit pre izvršenja**
Mora postojati mesto gde se kvota proverava i troši pre nego što runtime krene da računa.
- *Test:* Da li se zahtev preko kvote odbija pre trošenja GPU vremena, ili tek pošto se posao odradi?

**I5 — Podrška za obe serving putanje** (otvorenost kao dimenzija)
Sloj apstrakcije iznad runtime-a treba da podrži i deljeni upravljani pool i (kasnije) self-service deploj, da izbor serving modela ne zaključa stack.
- *Test:* Ako odluka padne na deljeni pool sada i self-service kasnije, da li ista apstrakcija pokriva oba, ili svaki ishod traži drugu tehnologiju?

**I6 — Standardizovan deploy/rollback/skaliranje**
Serving sloj treba da nudi deklarativan deploy, rollback verzije i skaliranje kao prvoklasne operacije (`popis_final.md` §6: KServe `InferenceService` CRD kao kandidat).
- *Test:* Jesu li rollback i skaliranje deklarativne operacije nad serving objektom, ili ručni zahvati po modelu?

---

## 4. HPC orkestracija

Domen pokriva: slanje računski zahtevnih poslova na PARADOX/ITE, praćenje statusa/logova, preuzimanje rezultata, prisilno zaustavljanje, prenos fajlova, kvote, kontejnere na klasteru. **Rutiranje Kubernetes ↔ SLURM je suštinski otvoreno** — kriterijumi mere sposobnost rutiranja, ne prag.

### Funkcionalni zahtevi

- **Slanje posla na klaster bez da korisnik zna SLURM** (sc. „Pokretanje računski zahtevnog posla") — apstrakcija HPC terminologije.
- **Status i live log nezavisno od operatera klastera** (sc. „Praćenje statusa i logova posla"; MVP put: SSH+sbatch ne zavisi od slurmrestd-a — `funkcionalnosti.md` 5).
- **Rezultati iz skladišta, resumable preuzimanje** (sc. „Preuzimanje rezultata posla").
- **Prisilno zaustavljanje posla koji ugrožava klaster** (sc. „Prisilno zaustavljanje posla") — operativna bezbednost.
- **Provera i rezervacija kvote pre slanja, obračun po stvarnoj potrošnji** (`popis_final.md` §5, §8a).
- **Prenos velikih fajlova** ka/od klastera (`popis_final.md` §5: rsync/SSH, S3 na HPC strani).
- **Kontejneri na klasteru** (Singularity/Apptainer; build na platformi → push → pull, `popis_final.md` §5).
- **Per-resource secret** za pisanje logova/statusa posla (`popis_final.md` §4).

### Kriterijumi

**H1 — Rad bez slurmrestd-a** [eliminacioni]
HPC bridge mora funkcionisati preko SSH+sbatch/squeue/sacct, jer dostupnost SLURM REST API-ja na PARADOX/ITE nije zagarantovana (visok rizik zavisnosti od operatera).
- *Test:* Radi li slanje, status i log ako klaster ne izloži slurmrestd — ili rešenje pada bez REST API-ja?

**H2 — Apstrakcija SLURM-a od korisnika** [eliminacioni]
Korisnik ne sme morati da piše sbatch skripte ili poznaje particije/QOS; to mapira platforma.
- *Test:* Može li ne-tehnički korisnik da pokrene posao bez ijedne linije SLURM sintakse?

**H3 — Sposobnost rutiranja po više osa** (otvorenost kao dimenzija)
Bridge mora moći da rutira posao po tipu/veličini, lokaciji podataka, afilijaciji i dubini reda — bez obzira na to kako će prag biti definisan.
- *Test:* Ako odluka o rutiranju padne na „tip posla" ili na „GPU sati", da li isti bridge podržava oba, ili svako pravilo traži drugu integraciju?

**H4 — Provera kvote pre rezervacije resursa** [eliminacioni]
Kvota se mora proveriti i rezervisati pre slanja, sa obračunom po stvarnoj potrošnji.
- *Test:* Da li se posao preko kvote odbija pre nego što zauzme čvor, ili tek pošto se izvrši?

**H5 — Prenos velikih artefakata bez ograničenja veličine**
Mehanizam prenosa mora podneti velike modele/datasetove ka klasteru i resumable preuzimanje rezultata nazad.
- *Test:* Prenosi li višegigabajtni artefakt pouzdano, sa nastavkom posle prekida — ili puca na velikim fajlovima?

**H6 — Bezbedan pristup klasteru bez statičkih ključeva**
Pristup mora ići preko kratkotrajnih SSH sertifikata (Vault SSH engine), namenski servisni nalog po klasteru (`popis_final.md` §5).
- *Test:* Autentifikuje li se bridge ka klasteru bez statičkog SSH ključa upisanog u konfiguraciju?

**H7 — Izolacija prava na log/status po poslu**
Pisanje loga/statusa mora tražiti per-resource token, dok list/get/destroy proveravaju pravo korisnika (`popis_final.md` §4) — da niko sa job ID-em ne čita tuđe logove.
- *Test:* Može li neko sa tuđim job ID-em da pročita ili upiše log tog posla?

---

## 5. Storage

Domen pokriva: skladištenje modela/datasetova/artefakata, upload/download, staging za HPC, verzionisanje, interfejs nad object store-om, relacionu bazu sa multi-tenancy.

### Funkcionalni zahtevi

- **Object store kao source of truth za artefakte** (`popis_final.md` §7, §13: težine i artefakti u MinIO, metapodaci u PostgreSQL).
- **Upload velikih fajlova** (presigned multipart) i resumable download (sc. „Upload dataseta", „Preuzimanje javnog dataseta").
- **Interfejs nad object store-om koji ne traži `git` od korisnika** — sakriti LFS/pointer mehaniku od ne-tehničkih korisnika (`popis_final.md` §7, §16: Git LFS/Xet odbijeni kao primarni interfejs).
- **Verzionisanje datasetova i modela**: ranije verzije nepromenljive i dostupne (`popis_final.md` §11a; DVC/LakeFS kandidati §7).
- **Multi-tenant izolacija u relacionoj bazi**: row-level security / schema-per-tenant / database-per-tenant (`popis_final.md` §7, §11).
- **Staging za HPC**: isti artefakt dostupan i platformi i klasteru (S3-kompatibilan API na HPC strani).
- **Integritet artefakta**: SHA-256 checksum, opciono potpisivanje, provera pre serviranja (`popis_final.md` §11).
- **Kompatibilnost sa Pharos backend-om preko adaptera**, bez usvajanja njihovog modela skladišta (`popis_final.md` §16).

### Kriterijumi

**S1 — Object store sakriven iza platformskog API-ja** [eliminacioni]
Korisnik upload-uje i preuzima kroz platformski interfejs (presigned URL), bez `git`, LFS pointera ili poznavanja bucket strukture.
- *Test:* Može li lekar ili agronom da postavi dataset bez ijednog `git`/CLI koraka?

**S2 — Multipart upload i resumable download** [eliminacioni]
Skladište mora podržati upload velikih fajlova u delovima i nastavak preuzimanja posle prekida.
- *Test:* Da li upload od više GB uspeva u delovima i da li se prekinut download nastavlja, ili kreće iznova?

**S3 — Jedinstven source of truth, bez paralelne istine**
Object store drži artefakte, relaciona baza metapodatke; nijedan drugi servis ne sme držati svoju kopiju istine (`popis_final.md` §13).
- *Test:* Postoji li tačno jedno merodavno mesto za artefakt i jedno za metapodatak, ili se stanje duplira?

**S4 — Multi-tenant izolacija na nivou baze**
Baza mora podržati izolaciju tenanata (RLS ili schema/db-per-tenant) tako da upit jednog tenanta ne vidi tuđe redove.
- *Test:* Da li tenant može tehnički da dohvati red drugog tenanta ako aplikativni filter zakaže — ili ga baza zaustavlja?

**S5 — Nepromenljive verzije artefakata**
Verzionisanje mora čuvati ranije verzije nepromenljivo i dostupno (rollback na raniju verziju, `popis_final.md` §11a).
- *Test:* Ostaje li v1 bit-identična i dohvatljiva pošto nastane v2?

**S6 — Isti artefakt dostupan klasteru (HPC staging)**
Skladište mora biti dostupno i sa PARADOX/ITE strane (S3-kompatibilno), da se isti artefakt ne kopira ručno na klaster.
- *Test:* Može li posao na klasteru da pročita artefakt direktno iz skladišta, bez zasebne ručne kopije?

**S7 — Integritet pre serviranja**
Checksum (SHA-256) i opciono potpis moraju se proveriti pre nego što se artefakt servira ili pusti u posao.
- *Test:* Da li se oštećen ili izmenjen artefakt odbija pre upotrebe?

**S8 — Federacijski adapter bez usvajanja tuđeg modela skladišta**
Skladište mora biti premostivo ka Pharos LFS/Xet backend-u kroz adapter, a da SAIFA ne pređe na git-native model.
- *Test:* Može li se federacija sa Pharos-om rešiti adapterom nad postojećim object store-om, bez migracije na njihov interfejs?

---

## 6. Experiment tracking

Domen pokriva: beleženje `run`-ova (parametri, metrike), poređenje eksperimenata, promociju `run`-a u verziju modela. Rad preko delimično izolovane veze ka klasteru je **eliminacioni kriterijum** iz `funkcionalnosti.md`.

### Funkcionalni zahtevi

- **Beleženje `run`-ova uživo i konačno** (sc. „Pregled i poređenje eksperimenata") — uživo za poslove `U toku`, konačno za `Završen`.
- **Poređenje više `run`-ova** sa parametrima i metrikama (tabela i grafici), sa tretmanom `run`-ova bez zajedničkih metrika.
- **Beleženje preko veze ka klasteru koja je delimično izolovana** (sc. korak A3) — alat mora buffer-ovati za vreme nedostupnosti i sinhronizovati po povratku veze.
- **Promocija `run`-a u registrovanu verziju modela** (sc. „Promocija `run`-a u verziju registrovanog modela") — veza ka registru.
- **Reproduktivnost evaluacije** (sc. „Evaluacija verzije modela na benchmark setu"; `funkcionalnosti.md` 10).

### Kriterijumi

**E1 — Beleženje preko delimično izolovane veze ka klasteru** [eliminacioni]
Alat mora pisati metrike iz posla koji se izvršava na klasteru kroz mrežno ograničenu vezu; ako alat zahteva stalnu direktnu konekciju, ispada.
- *Test:* Beleži li alat metrike iz posla na PARADOX/ITE i kad veza nije stalno otvorena — ili gubi beleženje čim konekcija padne?

**E2 — Buffer i sinhronizacija po povratku veze**
Za vreme nedostupnosti alat treba da buffer-uje, a po povratku da dopuni `run` bez gubitka prethodno zabeleženog.
- *Test:* Posle prekida veze tokom posla, da li se zabeleženi međurezultati naknadno sinhronizuju, ili `run` trajno ostaje krnj?

**E3 — Poređenje `run`-ova sa nepotpunim metrikama**
Poređenje mora raditi i kad `run`-ovi nemaju identičan skup metrika (prikaz praznina, ne pad).
- *Test:* Prikazuje li poređenje dva `run`-a sa različitim metrikama bez greške?

**E4 — Veza ka registru modela**
`run` mora moći da se promoviše u verziju registrovanog modela, sa očuvanim lineage-om ka eksperimentu.
- *Test:* Može li se iz `run`-a napraviti verzija modela tako da ostane zapis koji `run` ju je proizveo?

**E5 — Self-hosted, suvereno**
Alat mora raditi self-hosted; cloud-only (npr. W&B cloud) je samo rezerva, ne primarni izbor (`popis_final.md` §16).
- *Test:* Radi li potpuno u domaćoj infrastrukturi bez obaveznog cloud servisa?

**E6 — Reproduktivnost zabeleženog**
Zabeleženi parametri i okruženje moraju biti dovoljni da se evaluacija ponovi.
- *Test:* Da li `run` sadrži dovoljno da se rezultat reprodukuje, ili nedostaju ulazi/okruženje?

---

## 7. Edukacija

Domen pokriva: interaktivne sveske, lab vežbe sa pravim podacima, zaključane šablone za studente, kurseve/learning pathove, upis, napredak, sertifikate, resource limite za studente. Veći deo LMS funkcija je Pilot; infrastruktura sveske je MVP.

### Funkcionalni zahtevi

- **Interaktivna sveska po korisniku, sa izolacijom i kvotom** (sc. „Otvaranje interaktivne Jupyter lab sveske"; JupyterHub + KubeSpawner, `funkcionalnosti.md` 6 MVP).
- **Gašenje neaktivnih GPU sesija** (idle culling) — deo DoD-a (sc. „Konfiguracija politike gašenja neaktivnih GPU sesija").
- **Lab vežba sa read-only montažom pravih podataka iz skladišta** i zaključanim šablonom posla (sc. „Pokretanje unapred konfiguriranog AI factory posla u lab vežbi"; `popis_final.md` §8b).
- **Zaključani šablon posla za studenta** — student pokreće, ne menja parametre (sc. „Pokretanje posla iz zaključanog šablona").
- **Resource limiti za studente na nivou institucije** unutar pool-a organizacije (sc. „Konfiguracija resource limita za studente").
- **Kursevi, upis, napredak, sertifikati** iza platformskog API-ja (LMS sloj, Pilot; sc. sekcija 8).
- **Sadržaj na srpskom i engleskom**.

### Kriterijumi

**D1 — Sveska po korisniku sa izolacijom i limitima** [eliminacioni]
Notebook okruženje mora dati izolovanu sesiju po korisniku sa CPU/RAM/GPU limitima i trajanjem sesije.
- *Test:* Dobija li svaki student izolovanu svesku sa zasebnim limitima, ili dele isto okruženje?

**D2 — Idle culling GPU sesija** [eliminacioni]
Sistem mora automatski gasiti neaktivne GPU sveske (skupi resurs ne sme da visi).
- *Test:* Gasi li se neaktivna GPU sesija sama posle praga, ili drži GPU dok je korisnik ne zatvori ručno?

**D3 — Zaključan šablon posla za studenta**
Mora postojati način da student pokrene unapred konfigurisan posao bez menjanja parametara/resursa.
- *Test:* Može li student da pokrene vežbu a da ne može da promeni SLURM/resource parametre šablona?

**D4 — Read-only montaža pravih podataka u svesku**
Lab vežba mora moći da montira stvarni dataset iz skladišta read-only, bez kopije i bez prava izmene.
- *Test:* Vidi li sveska prave podatke za vežbu bez mogućnosti da ih izmeni ili iznese?

**D5 — Limiti studenata u okviru pool-a organizacije**
Resource limiti studenata moraju se postavljati na nivou institucije unutar kvote organizacije, ne otvoreno.
- *Test:* Da li zbir studentskih sesija ostaje unutar pool-a institucije, ili može da ga probije?

**D6 — LMS iza platformskog API-ja**
Ako se uvodi LMS, korisnik ne sme izlaziti iz portala; kursevi/upis/napredak idu kroz SAIFA API (`popis_final.md` §8b).
- *Test:* Ostaje li korisnik unutar portala kroz ceo kurs, ili ga LMS izbacuje na zaseban sistem?

**D7 — Dvojezični sadržaj (sr/en)**
Edukacioni sloj mora podržati sadržaj na srpskom i engleskom.
- *Test:* Može li se isti kurs ponuditi na srpskom i engleskom?

---

## 8. Federacija

Domen pokriva: pristup federisanim resursima, izvoz/uvoz ka partnerima (Pharos, IT4LIA), sync kataloga, oznaku porekla. Cela grupa je Pilot (`funkcionalnosti.md` 9: MVP samo mock/read-only); zrelost partnerskih API-ja je najveća nepoznanica projekta — kriterijumi to tretiraju kao primarni rizik, ne pretpostavljaju gotov API.

### Funkcionalni zahtevi

- **Apstrakcija izvora po partneru**: jedan interfejs (CatalogueSource) nad različitim backend-ima — Pharos API, IT4LIA API, CKAN, Dataverse (`popis_final.md` §10).
- **Uvezeni resurs ulazi sa oznakom porekla i nikad ne prepisuje lokalni source of truth** (sc. „Pristup federisanim resursima"; `popis_final.md` §13).
- **Izvoz lokalnog resursa na partnersku platformu** (sc. „Izvoz resursa na partnersku platformu").
- **Sync kataloga** zakazan i event-driven, sa validacijom šeme pre svake razmene (sc. „Konfiguracija automatskog sync-a"; `popis_final.md` §10).
- **Bezbedna razmena**: mTLS + kredencijali po partneru, rate limiting (`popis_final.md` §10).
- **Mock/read-only u MVP-u**, realna veza tek kad partnerski API sazri.

### Kriterijumi

**F1 — Adapter po partneru iza jednog interfejsa** [eliminacioni]
Federacija mora biti iza apstrakcije (CatalogueSource) tako da novi partner znači nov adapter, ne izmenu jezgra.
- *Test:* Kad se doda IT4LIA pored Pharos-a, da li je to nov adapter — ili dira jezgro kataloga?

**F2 — Oznaka porekla bez mešanja u lokalnu istinu** [eliminacioni]
Uvezeni resurs mora ući isključivo sa oznakom porekla i ne sme prepisati lokalni source of truth.
- *Test:* Može li federisani resurs da prepiše ili zameni lokalni zapis — ili uvek stoji odvojeno označen?

**F3 — Validacija šeme pre razmene**
Pre svakog uvoza/izvoza šema partnerskog resursa mora se validirati; nevalidan resurs se odbija.
- *Test:* Odbija li se resurs sa neusklađenom šemom pre nego što uđe u katalog?

**F4 — Rad sa mock-om dok partnerski API ne sazri**
Adapter mora raditi protiv mock/read-only izvora u MVP-u, da razvoj ne čeka spremnost partnera.
- *Test:* Može li se federacija razvijati i demonstrirati protiv mock-a, bez živog Pharos API-ja?

**F5 — Bezbedna međuplatformska veza**
Razmena mora ići preko mTLS sa kredencijalima po partneru i rate limitingom.
- *Test:* Je li veza ka partneru uzajamno autentifikovana i ograničena po stopi, ili otvorena?

**F6 — Tolerancija na promenu partnerske šeme**
Pošto je zrelost partnerskog API-ja nepoznanica, adapter treba da izoluje promene njihove šeme od ostatka sistema.
- *Test:* Ako Pharos promeni format, da li je udar ograničen na adapter, ili se prelije na katalog i registar?

---

## Napomena o granicama ovih kriterijuma

Kriterijumi definišu **prema čemu** se bira tehnologija po domenu. Oni eksplicitno ne:

- **donose arhitektonske odluke** — gde je domen otvoren (serving model, prag rutiranja), kriterijum meri sposobnost kandidata da podrži oba ishoda, ali sam izbor ostaje u `otvorene_odluke.md`;
- **rangiraju kandidate** — test-pitanje kaže prolazi/ne prolazi po kriterijumu; ukupna ocena kandidata je zaseban korak (PoC + upitnik klijenta);
- **uzimaju u obzir kapacitet tima ni redosled implementacije** — to pokriva `mvp_kriterijumi.md` (faze) i planiranje sprinta;
- **zamenjuju eliminacione kriterijume iz `funkcionalnosti.md`** — oni su ovde preuzeti i označeni **[eliminacioni]**, ali merodavan izvor ostaje `funkcionalnosti.md`.
