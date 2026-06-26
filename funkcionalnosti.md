# SAIFA AI Gateway — Funkcionalnosti, opcije i plan testiranja

Ovaj dokument polazi od funkcionalnosti platforme (šta korisnik radi) i za svaku popisuje opcije realizacije, a zatim opisuje kako planiramo da testiramo koja opcija je najbolja za naš slučaj korišćenja. Zajedno sa `popis.md` (popis tehnologija po slojevima infrastrukture) čini osnovu za izbor tehnologija: `popis.md` odgovara na pitanje "šta sve postoji", a ovaj dokument na pitanja "čemu to služi kod nas" i "kako ćemo proveriti šta nam odgovara".

Da bi se izbegao utisak da sve treba raditi odmah: svaka funkcionalnost je svrstana u fazu isporuke (MVP / Pilot / Production / Later) u tabeli prioriteta ispod, ima jasan "definition of done" i popisane rizike. Konačni redosled i obim određuju odgovori klijenata iz `upitnik-zahtevi.md`.

## Mapiranje na ciljeve Gateway-a

Iz inicijalnog opisa, Gateway treba da ispuni šest ciljeva. Svaka funkcionalnost postoji da bi podržala bar jedan od njih — to je njena poslovna vrednost, ne sama tehnologija.

| Cilj Gateway-a | Funkcionalnosti koje ga nose |
| --- | --- |
| One-stop shop (jedna ulazna tačka) | 7 katalog/pretraga, 2 inference, 11 IAM, 1 model registry |
| Federated access (federisani pristup) | 9 federacija, 7 katalog (oznaka porekla), 11 IAM (EuroHPC identitet) |
| Learning platform | 8 LMS, 6 sveske, 10 benchmarking (edukativni) |
| Launchpad for innovation | 3 fine-tuning, 5 HPC submit, 14 workflow, 6 sveske |
| Secure collaboration | 13 kolaboracija, 11 IAM, 12 audit, 4 dataset (GDPR) |
| EuroHPC integration | 9 federacija, 5 HPC submit, 11 IAM (MyAccessID), 12 monitoring (izveštavanje) |

## Korisnički scenariji (persone)

Funkcionalnosti dobijaju smisao kroz konkretne korisnike. Svaki scenario je ujedno i test usvajanja — ako persona ne prođe svoj put bez pomoći, funkcionalnost nije gotova.

- **Istraživač** želi da fine-tune-uje model na svom datasetu: prijava → nađe bazni model i dataset → pokrene fine-tuning iz šablona na HPC → prati napredak → novi model se registruje. (funkc. 1, 3, 4, 5, 11)
- **Lekar / sektorski stručnjak** želi sandbox nad kontrolisanim zdravstvenim podacima: prijava → otvori sandbox sa read-only pristupom → radi obradu koja ne iznosi podatke van klastera (compute-to-data). (funkc. 4, 5, 6, 11, 12)
- **Student** radi lab vežbu: upis na kurs → otvori svesku → pokrene mali posao na klasteru kroz zaključan šablon → automatska provera → upisan napredak. (funkc. 6, 8, 5)
- **SME / startup** traži gotov inference API: prijava → nađe model u katalogu → dobije OpenAI-kompatibilan endpoint sa token budžetom. (funkc. 2, 7, 11)
- **Analitičar iz javnog sektora** traži dataset i izveštaj: pretraga kataloga → preuzimanje ili API pristup → potrošnja vidljiva u monitoringu. (funkc. 7, 4, 12)
- **EuroHPC partner** uvozi/izvozi resurse: federisani katalog sa oznakom porekla, sync uz odobravanje objave. (funkc. 9, 13)

## Tabela prioriteta (faze isporuke)

"basic" = osnovna verzija dovoljna da scenario prođe; prazno polje = nije u toj fazi.

| Funkcionalnost | MVP | Pilot | Production | Later |
| --- | --- | --- | --- | --- |
| 11 Login / uloge | da | da | da | |
| 7 Katalog i pretraga | full-text | + semantička | da | |
| 2 Inference / chat | basic (1 serving) | + sektorski chat | da | |
| 1 Registracija modela | upload + HF import | + eksterni API, lineage | da | |
| 5 HPC submit | basic (SSH+sbatch) | + šabloni, kvote | da | |
| 4 Dataset upload/verzionisanje | delimično | da | da | |
| 6 Sveske | JupyterHub (KubeSpawner) | + SlurmSpawner | da | |
| 3 Fine-tuning | basic LoRA šablon | da | da | |
| 12 Monitoring / audit | basic | da | da | |
| 13 Kolaboracija | team vidljivost | + review/approval | da | |
| 10 Benchmarking | basic harness | + leaderboard | da | |
| 8 LMS | biblioteka materijala | LMS pilot | da | |
| 9 Federacija | mock / read-only | pilot (sandbox) | da | |
| 14 Workflow orchestration | | pilot (own scheduler) | da | generički engine |

## Kako ceo pipeline izgleda (kontekst za sve funkcionalnosti)

Bez obzira na to koju funkcionalnost korisnik koristi, put kroz sistem je uvek isti i sve opcije ispod se uklapaju u njega:

1. Korisnik se prijavljuje (lokalni nalog, univerzitetski nalog ili EuroHPC identitet) i dobija pogled prilagođen svojoj ulozi.
2. U katalogu pronalazi resurs: model, dataset, kurs, svesku ili workflow — lokalni ili federisani (Pharos, IT4LIA).
3. Pokreće akciju: postavlja pitanje modelu, pokreće fine-tuning, šalje posao na superračunar, otvara svesku ili lab vežbu.
4. Platforma posao izvršava tamo gde mu je mesto: na Kubernetes GPU pool-u (brze, interaktivne stvari) ili na PARADOX/ITE klasteru (veliki poslovi), uz proveru kvota.
5. Rezultat se vraća korisniku: odgovor u chatu, novi model u registru, fajlovi u skladištu, položena lab vežba — uz notifikaciju.
6. Sve vreme ispod radi nadzor, merenje potrošnje i audit — svaki korak je zabeležen.

## Metodologija: kako ćemo birati između opcija

Plan nam je da ne biramo tehnologije na osnovu utisaka i popularnosti, nego da svaku ozbiljnu odluku donesemo posle malog, vremenski ograničenog testa (proof-of-concept od nekoliko dana, u dev okruženju — Docker Compose ili mali k3s klaster). Svaki test radimo ovako:

1. Pre testa zapišemo šta tačno merimo i šta je kriterijum uspeha (da posle ne pomeramo golove).
2. Testiramo opcije na istom zadatku, sa istim podacima/modelom, da poređenje bude pošteno.
3. Ocenjujemo svaku opciju (1–5) po šest kriterijuma:
   - iskustvo korisnika — posebno za ne-tehničke korisnike, jer su oni najveći rizik za usvajanje platforme;
   - trud integracije — koliko koda i konfiguracije treba da se uklopi u našu arhitekturu;
   - operativna složenost — da li to mali tim može da održava godinama;
   - performanse i trošak — latencija, propusnost, GPU sati, hardverski zahtevi;
   - zrelost i zajednica — dokumentacija, učestalost release-ova, koliko je lako naći pomoć;
   - usklađenost — GDPR, suverenitet podataka (self-hosted), licence.
4. Odluku zapišemo kao kratak zapisnik (decision record): šta smo testirali, rezultati, šta smo izabrali i zašto. Tako svaka odluka ostaje proverljiva i može da se preispita kad se okolnosti promene.
5. Gde je moguće, u test uključujemo prave korisnike: za korisničke interfejse barem 3–5 osoba iz ciljne grupe (npr. neko iz zdravstva za sektorski chat), jer naše mišljenje o "jednostavnosti" ne vredi mnogo.

Redosled testiranja pratiće prioritete iz upitnika za klijente (`upitnik-zahtevi.md`): prvo testiramo opcije za funkcionalnosti koje klijenti zaista traže, a egzotičnije opcije ostavljamo za kasnije ili preskačemo.

## 1. Registracija i dodavanje modela

Opcije:
- Načini dodavanja: import sa HuggingFace Hub-a (po ID-u) · upload sopstvenog checkpoint-a · registracija eksternog API modela (OpenAI, Anthropic, Mistral, Groq, Replicate) · programski kroz svesku/CLI (SDK / REST API)
- Skladištenje težina: MinIO (S3-kompatibilno), resumable multipart upload, SHA-256 provera integriteta
- Metapodaci: registar u PostgreSQL-u (licenca, jezik, tip zadatka, lineage), verzionisanje, vidljivost public/private/team

Kako testiramo: uzećemo tri reprezentativna slučaja — mali model (~1 GB), veliki model (7B, ~15 GB) i jedan eksterni API model — i proći ceo put registracije za svaki. Namerno ćemo prekidati download na pola da vidimo da li se nastavlja ili kreće ispočetka, i proveriti da li checksum provera zaista zaustavlja oštećen fajl. Merimo ukupno vreme, broj koraka za korisnika i šta se dešava kad nešto krene po zlu. Kriterijum uspeha: ekspert registruje model bez čitanja dokumentacije, a prekinut prenos se sam oporavlja.

Definition of done: korisnik registruje model (upload / HF / eksterni API), dobije verziju i obavezne metapodatke (uklj. licencu), model je vidljiv u katalogu po zadatoj vidljivosti, a prekinut prenos se sam oporavlja.

Rizici: tehnički (veliki fajlovi, integritet) — nizak · compliance (provera licence modela pre objave) — srednji · zavisnost od partnera — niska.

## 2. Korišćenje modela (inference / chat)

Opcije:
- Interfejsi: chat UI (sektorski prilagođen) · REST API (OpenAI-kompatibilan) · poziv iz sveske
- Serving: vLLM · Triton Inference Server · HuggingFace TGI · ONNX Runtime (laki CPU modeli) · eksterni provajderi kroz proxy
- Prateće: SSE streaming · rate limiting i token budžeti · keširanje (Redis) · merenje potrošnje

Kako testiramo: isti model (7B, po mogućstvu srpski fine-tune) podižemo na vLLM i TGI na istom GPU-u, pa puštamo load test (k6 ili locust): 1, 10, 50 paralelnih korisnika. Merimo vreme do prvog tokena, p95 latenciju, propusnost tokena u sekundi i zauzeće GPU memorije. Posebno gledamo ponašanje pod preopterećenjem — da li degradira elegantno ili puca. Za chat UI radimo test sa ne-tehničkim korisnicima: dajemo im zadatak ("pitajte model X i ocenite iskustvo") i gledamo gde zapinju. Kriterijum: stabilan streaming pod opterećenjem i chat koji ne-tehnički korisnik koristi bez pomoći.

Definition of done: model se servira preko OpenAI-kompatibilnog API-ja i chat UI-ja, sa streamingom, rate limitom, token budžetom i merenjem potrošnje; ne-tehnički korisnik dobije odgovor bez pomoći.

Rizici: tehnički (GPU kapacitet, latencija pod opterećenjem) — srednji · operativni (održavanje serving stacka) — srednji.

## 3. Fine-tuning modela

Opcije:
- Pokretanje: vizard sa hiperparametrima · YAML šabloni · custom skripta iz sveske · API
- Stack: HuggingFace Transformers + PEFT (LoRA/QLoRA) · Axolotl · Unsloth · LLaMA-Factory
- Izvršavanje: SLURM job na PARADOX/ITE · Kubernetes Job za manje poslove
- Praćenje: MLflow · Weights & Biases · ClearML

Kako testiramo: isti bazni model i isti dataset (mali srpski instruction set, npr. 5–10k primera) provlačimo kroz Axolotl, Unsloth i LLaMA-Factory. Merimo: koliko traje podešavanje konfiguracije (subjektivno, ali beležimo sate), VRAM potrošnju, trajanje treninga i — najbitnije — kvalitet rezultata na istom evaluacionom setu pre/posle. Paralelno poredimo LoRA i QLoRA na istom zadatku da utvrdimo koliko gubimo kvaliteta za koliko ušteđenih resursa. Za praćenje eksperimenata podižemo MLflow self-hosted i proveravamo da li radi preko veze ka klasteru (delimično izolovano okruženje) — to je za nas eliminacioni kriterijum, pa W&B testiramo samo ako MLflow podbaci. Kriterijum: ceo ciklus (config → trening → eval → registracija) prolazi bez ručnih intervencija.

Definition of done: korisnik pokrene fine-tuning iz šablona, posao ode na HPC/Kubernetes, prati napredak uživo, a rezultujući model se automatski registruje sa lineage-om ka baznom modelu i datasetu.

Rizici: tehnički (GPU sati, stabilnost treninga) — srednji · organizacioni (ko odobrava trošak GPU sati) — srednji.

## 4. Upravljanje datasetovima

Opcije:
- Upload: portal (presigned multipart u MinIO) · SDK/CLI · povezivanje izvora (Nacionalni data centar Kragujevac)
- Verzionisanje: DVC · LakeFS
- Validacija: provera formata, enkodiranja, heuristički PII sken
- GDPR: klasifikacija (4 nivoa) · saglasnosti · anonimizacija · lineage

Kako testiramo: za DVC i LakeFS igramo isti scenario iz stvarnog života: upload dataseta, dodavanje nove verzije, vraćanje na staru, i — gde se razlikuju — paralelna anotacija u dve grane sa spajanjem. Merimo koliko je to komplikovano za korisnika (koji ne sme da vidi nijednu DVC/LakeFS komandu — sve ide kroz naš API) i za nas koji to održavamo. Za PII sken pravimo probni dataset sa namerno ubačenim ličnim podacima (imena, JMBG-ovi, brojevi telefona) i merimo koliko ih sken hvata, jer od toga zavisi GDPR automatika. Kriterijum: verzionisanje koje korisnik razume kao "verzija 1, verzija 2" bez ikakvog znanja o alatu ispod.

Definition of done: korisnik upload-uje dataset, dobije verziju, vrati se na staru, a sistem klasifikuje podatke i prijavi mogući PII pre nego što dataset može da se objavi ili podeli.

Rizici: compliance (PII, GDPR, osetljivi/zdravstveni podaci) — visok · tehnički (veliki datasetovi, anotacije) — srednji · organizacioni (ko je vlasnik dataseta) — srednji.

## 5. Pokretanje poslova na HPC (PARADOX, ITE)

Opcije:
- Interfejsi: forma u portalu · sveska sa SDK · raw API · zaključani šabloni za studente
- Konekcija: SSH + sbatch/squeue/sacct · slurmrestd REST API · Open OnDemand (komplementarno)
- Prenos: rsync (veliki fajlovi) · SFTP (skripte) · S3/MinIO na HPC strani
- Bezbednost: Vault SSH sertifikati · izolovan bridge servis · kvote sa rezervacijom i obračunom

Kako testiramo: prvo sve razvijamo protiv lokalnog mock SLURM-a (kontejner sa pravim slurmctld/slurmd), pa tek onda na pravom klasteru. Ključni testovi otpornosti: prekid SSH veze tačno između sbatch komande i čitanja job ID-a (da li dupliramo posao?), restart bridge servisa dok poslovi traju (da li ih sistem ponovo "usvoji" preko sacct?), i prekid rsync prenosa velikog dataseta (da li nastavlja gde je stao?). slurmrestd testiramo samo ako ga operateri klastera omoguće — zato je SSH putanja primarna, jer ne zavisi ni od koga. Merimo i kašnjenje live log streama (od linije u SLURM logu do browsera). Kriterijum: nijedan scenario prekida ne sme da proizvede dupli posao ili izgubljen rezultat.

Definition of done: korisnik pokrene job iz portala, vidi status i logove uživo, preuzme rezultat, a sistem sprečava dupli submit posle prekida veze i oporavlja stanje poslova posle restarta bridge servisa.

Rizici: zavisnost od operatera klastera (slurmrestd, S3/MinIO, kvote nisu zagarantovani) — visoka · tehnički (otpornost na prekide veze) — srednji · bezbednosni (pristup klasteru) — srednji.

## 6. Interaktivni rad u sveskama (eksperti i studenti)

Opcije:
- JupyterHub (SSO preko Keycloak-a): KubeSpawner · SlurmSpawner
- Alternative: JupyterLite (browser, bez servera) · code-server (pun IDE) · Marimo (edukativne sveske) · Google Colab (samo javni edukativni sadržaj)
- Politike: limiti po korisniku, gašenje neaktivnih sesija, GPU po ulozi

Kako testiramo: merimo vreme od klika "Otvori svesku" do prve izvršene ćelije za KubeSpawner (očekujemo sekunde) i SlurmSpawner (očekujemo minute zbog reda čekanja) — to direktno odlučuje koji je podrazumevan. Simuliramo pun klaster da vidimo kakvu poruku korisnik dobija kad resursa nema. Za obuke pravimo jednu istu lab vežbu u JupyterLite i na JupyterHub-u i dajemo je grupi studenata — ako JupyterLite pokriva vežbu, štedimo serverske resurse. code-server dajemo dvojici-trojici naprednih korisnika na dve nedelje i slušamo šta kažu. Posebno testiramo gašenje neaktivnih GPU sesija: da li upozorenje stiže na vreme i da li se rad gubi (ne sme — radni direktorijum je na trajnom disku). Kriterijum: sveska se otvara dovoljno brzo da ne prekida tok rada, a neaktivne GPU sesije se oslobađaju bez gubitka rada.

Definition of done: sveska se otvara dovoljno brzo da ne prekida tok rada, izolovana je po korisniku sa kvotama, a neaktivne GPU sesije se oslobađaju bez gubitka rada.

Rizici: operativni (GPU kvote, idle culling) — srednji · tehnički (spawn latencija na SLURM-u) — srednji.

## 7. Pretraga i katalog resursa

Opcije:
- Pretraga: OpenSearch (full-text, faceti) · pgvector (semantička) · Qdrant/Weaviate (ako semantika preraste pgvector)
- Sektorski pogledi: konfiguracioni filteri nad istim katalogom
- Provenance: oznaka izvora (lokalno / Pharos / IT4LIA / CKAN)

Kako testiramo: punimo probni katalog sa 100+ stvarnih stavki (modeli, datasetovi, kursevi) i pravimo listu od 30-ak realnih upita — obavezno i na srpskom, i ćirilicom i latinicom, jer je to naš specifičan problem (korisnik kuca "модел за здравство" i "model za zdravstvo" i očekuje iste rezultate). Poredimo čist full-text (OpenSearch) sa hibridom full-text + semantička pretraga (pgvector): da li semantika opravdava dodatnu složenost? Ocenjuje se ručno: za svaki upit, da li je pravi rezultat u prvih 5. Kriterijum: pretraga radi podjednako dobro na oba pisma i oba jezika, a sektorski filter nikad ne propusti neodobren resurs.

Definition of done: pretraga vraća tačan rezultat u prvih 5 na oba pisma i oba jezika, svaki resurs ima jasnu oznaku porekla, a sektorski filter nikad ne propusti neodobren resurs.

Rizici: tehnički (kvalitet semantike na srpskom) — srednji · bezbednosni (filter pristupa u pretrazi) — srednji.

## 8. Obuke i edukacija (LMS)

Opcije:
- LMS: Open edX · Moodle — iza SAIFA API-ja
- Lab vežbe: kurirane sveske + pravi podaci iz MinIO + zaključani SLURM šabloni na ITE
- Jezici: srpski (ćir./lat.) i engleski

Kako testiramo: postavljamo isti probni kurs (par lekcija + jedna hands-on lab vežba) i na Open edX i na Moodle, pa merimo dve stvari: koliko nas košta integracija (upis, praćenje napretka, sertifikat — sve kroz API, korisnik ne sme da oseti da je "izašao" iz SAIFA portala) i kako ga ocenjuju probni studenti. Lab vežbu testiramo ceo lanac: upis → otvaranje sveske → čitanje dataseta → slanje malog posla na klaster → automatska provera rezultata → upisan napredak. Kriterijum: student završi vežbu bez ičije pomoći, a mi procenimo koji LMS traži manje održavanja (Moodle je lakši za hosting, Open edX bogatiji — odluka zavisi od obima kurseva koje klijenti najave).

Definition of done: student završi lab vežbu kroz portal bez izlaska iz SAIFA okruženja, sa upisanim napretkom i (opciono) sertifikatom.

Rizici: organizacioni (obim kurseva nepoznat dok klijenti ne kažu) — srednji · operativni (održavanje LMS-a) — srednji.

## 9. Federacija sa EuroHPC (Pharos, IT4LIA)

Opcije:
- Sync kataloga: zakazan + event-driven · izvoz modela/datasetova/benchmarka · uvoz sa oznakom porekla
- Apstrakcija: adapter po partneru (Pharos API, IT4LIA API, CKAN, Dataverse)
- Identitet: Keycloak brokering ka EuroHPC AAI · eduGAIN preko AMRES-a

Kako testiramo: ovde smo zavisni od partnera, pa testiramo u dva koraka. Prvo pravimo mock Pharos servis po njihovoj objavljenoj šemi metapodataka i na njemu razvijamo i testiramo ceo izvoz/uvoz, uključujući i namerno pokvarene odgovore (pogrešna šema, timeout, polovičan odgovor) — sync ne sme da upiše ništa polovično. Drugi korak je test protiv pravog Pharos sandbox okruženja, čim ga dobijemo kroz tehničke radionice — to tražimo od partnera odmah, jer je njihova spremnost najveća nepoznanica celog projekta. Kriterijum: model označen za objavu pojavi se u (mock) partnerskom katalogu bez ručne intervencije, a nijedan kvar partnera ne ošteti naš katalog.

Definition of done: model označen za objavu pojavi se u (mock) partnerskom katalogu bez ručne intervencije, uvezeni resursi nose oznaku porekla, a nijedan kvar partnera ne ošteti naš katalog.

Rizici: zavisnost od partnera (zrelost Pharos/IT4LIA API-ja i metapodataka) — visoka · tehnički (validacija šeme, otpornost na kvarove) — srednji.

## 10. Benchmarking i evaluacija

Opcije:
- Harness: lm-evaluation-harness + sektorski benchmark datasetovi (srpski jezik)
- Izvršavanje: platformski job na HPC ili GPU pool-u
- Leaderboard: objava po verziji benchmarka, uz odobrenje

Kako testiramo: puštamo lm-evaluation-harness na dva poznata modela i proveravamo dve stvari: reproduktivnost (isti model, dva pokretanja — rezultati moraju biti praktično identični, inače leaderboard nema smisla) i cenu (koliko GPU sati košta standardna evaluacija, da znamo šta možemo da priuštimo po modelu). Paralelno sastavljamo prvi mali srpski benchmark set i proveravamo koliko je posla uvesti custom task u harness. Kriterijum: evaluacija je ponovljiva, automatizovana kao običan platformski posao, i košta predvidivo.

Definition of done: evaluacija je ponovljiva, pokreće se kao običan platformski posao, košta predvidivo, a rezultat ide na leaderboard tek uz odobrenje (vidi 13).

Rizici: tehnički (reproduktivnost) — srednji · trošak (GPU sati po evaluaciji) — srednji.

## 11. Upravljanje korisnicima i pristupom

Opcije:
- Identity provider: Keycloak (OIDC/OAuth2/SAML2) · prijava lokalna, eduGAIN/AMRES, EuroHPC identitet
- Uloge (RBAC): platform-admin, data-admin, model-reviewer, ai-expert, advanced-user, basic-user, student, mentor/instruktor, viewer, federation-partner
- Atributi (ABAC): sektorske sertifikacije, afilijacija → kvote, GDPR saglasnosti
- API ključevi: generisanje, scope-ovanje, rotacija, opoziv

Kako testiramo: podižemo Keycloak realm sa svim ulogama i pišemo automatizovane testove izolacije koji za svaku ulogu pokušavaju svaku operaciju nad tuđim resursima — sve mora da vrati "zabranjeno". Ovi testovi ostaju trajno u CI-ju, jer je curenje podataka između tenanta najgora greška koju platforma može da napravi. eduGAIN prijavu testiramo sa AMRES test identitetom čim dobijemo pristup (organizaciono pitanje — pokrećemo odmah). Posebno testiramo scenario "ekspert pravi API ključ, koristi ga iz skripte, opoziva ga" — opoziv mora da deluje odmah. Kriterijum: nijedan test izolacije ne prolazi pogrešno, ni jednom.

Definition of done: svaka uloga vidi i radi samo ono što sme, nijedan test izolacije ne prolazi pogrešno, a opoziv API ključa deluje odmah.

Rizici: bezbednosni (curenje između tenanta) — visok · organizacioni (eduGAIN/AMRES i EuroHPC AAI pristup) — srednji.

## 12. Administracija, nadzor i usklađenost

Opcije:
- Monitoring: Prometheus + Grafana (HPC redovi, latencija, GPU iskorišćenost, kvote, GPU sati i energija) · Loki (logovi) · OpenTelemetry tracing · Sentry
- Mrežni nadzor pristupnih tačaka i linkova ka HPC: NetFlow · IPFIX · sFlow — lanac exporter (ruter/firewall) → collector (nfdump, GoFlow) → analyzer (ntopng, ElastiFlow + Grafana)
- Audit: append-only log svih operacija, retencija 2+ godine, compliance izveštaji
- GDPR alati: DSAR izvoz, pravo na brisanje, DPIA procedura
- Notifikacije: email + in-app
- Feature flags: Unleash · env varijable

Kako testiramo: monitoring stack podižemo već u dev okruženju (ne na kraju!) i proveravamo da li iz njega možemo da odgovorimo na pitanja koja će nam EuroHPC postavljati: koliko je GPU sati potrošeno po projektu, kolika je potrošnja energije, koliko je preuzimanja dataseta bilo. Ako na neko pitanje ne možemo da odgovorimo iz dashboarda, fali nam metrika. Za mrežni nadzor pravimo mali ogled: softverski exporter (softflowd) na dev mašini → nfdump kolektor → vizualizacija u Grafani, da ceo NetFlow lanac vidimo na delu pre razgovora sa mrežnim timom o pravim ruterima. Audit testiramo pokušajem prevare: da li možemo da izmenimo ili obrišemo postojeći zapis? Ne smemo moći, čak ni kao admin baze. Kriterijum: svaki izveštaj koji compliance traži dobija se iz sistema, bez ručnog kopanja po logovima.

Definition of done: svaki izveštaj koji compliance/EuroHPC traži dobija se iz sistema bez ručnog kopanja po logovima, a audit zapis ne može da se izmeni ni obriše čak ni kao admin baze.

Rizici: compliance (retencija, integritet audita) — srednji · operativni (održavanje monitoring stacka) — nizak.

## 13. Kolaboracija i timski rad

Opcije:
- Projekti i timski workspace-ovi: deljenje modela, datasetova i rezultata unutar tima
- Review/approval: odobravanje modela i datasetova, komentari, validation reports
- Kod-kolaboracija: interni GitLab/Gitea (merge request, review) — već potreban za DVC

Kako testiramo: pravimo prototip team vidljivosti (model vidljiv timu, nevidljiv ostalima) i dajemo ga jednoj istraživačkoj grupi (3–5 ljudi) da prođe stvaran ciklus: jedan trenira, drugi pregleda, treći odobrava za objavu. Slušamo gde im proces smeta — kolaboracione funkcionalnosti je lako prekomplikovati, pa namerno krećemo od minimuma i dodajemo samo ono što grupa zatraži. Kriterijum: review ciklus od treninga do odobrene objave prođe kroz platformu bez "sa strane" komunikacije (mejlova sa fajlovima u prilogu).

Definition of done: review ciklus od treninga do odobrene objave prođe kroz platformu, sa jasnim ko-šta-odobrava (vidi RACI ispod), bez komunikacije "sa strane".

Rizici: organizacioni (definisanje ownership i approval procesa) — srednji · tehnički (team vidljivost) — nizak.

## 14. Workflow orchestration (ML pipeline-ovi)

Opcije:
- Kubeflow Pipelines · Apache Airflow · Argo Workflows · sopstveni job-scheduler (dovoljan za jednostavne lance, već u arhitekturi)

Kako testiramo: definišemo referentni pipeline od četiri koraka (preprocesiranje → trening → evaluacija → registracija) i prvo ga izvedemo našim job-scheduler-om sa ulančavanjem preko događaja. Ako to bude čitljivo i pouzdano, generički engine nam ne treba — svaki od navedenih alata je ozbiljan sistem za održavanje i ne uvodimo ga "za svaki slučaj". Ako se pokaže da klijenti traže složenije lance (grananje, retry politike po koraku, paralelizam), radimo PoC sa Argo Workflows (najprirodniji na Kubernetes-u) i tek onda poredimo sa Airflow/Kubeflow. Kriterijum odluke: složenost pipeline-ova koju klijenti stvarno traže u upitniku, ne ona koju možemo da zamislimo.

Definition of done: referentni pipeline (preprocesiranje → trening → evaluacija → registracija) prođe kroz sopstveni job-scheduler bez ručnih intervencija; generički engine se uvodi samo ako ga klijentske potrebe opravdaju.

Rizici: tehnički (složenost koju klijenti traže nepoznata) — srednji · rizik prekomerne izgradnje (uvođenje generičkog engine-a bez potrebe) — srednji.

## Ownership i odobravanje (RACI)

Mentor je tačno primetio da nedostaje ko šta odobrava. Sledeća matrica definiše vlasništvo nad ključnim koracima životnog ciklusa resursa. R = radi, A = krajnje odgovoran (jedan po redu), C = konsultuje se, I = obaveštava se.

| Akcija | Platform tim | Data owner | Model owner | Sektorski ekspert | Legal / DPO | HPC operater | Federation partner |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Odobrenje dataseta za internu upotrebu | C | A/R | I | C | C | | |
| Legal/compliance (GDPR) review dataseta | I | C | | C | A/R | | |
| Tehnička validacija modela | R | | A | C | | | |
| Odobrenje modela za internu objavu | C | | A/R | C | C | | |
| Objava modela/dataseta u Pharos/EuroHPC | R | C | C | I | C | | A |
| Dodela i obračun HPC kvota | R | | | | | A/C | |
| Sinhronizacija federacije | A/R | | | | C | | C |

Napomena: "model owner" i "data owner" su uloge na strani sektora/tima, ne platformskog tima — platforma obezbeđuje proces, sektor donosi odluku o sadržaju.

---

Sledeći korak: prioritete testiranja određujemo prema odgovorima klijenata iz `upitnik-zahtevi.md`; svaki završen test dobija kratak zapisnik odluke (šta smo testirali, rezultati po kriterijumima, izbor i obrazloženje), koji čuvamo uz ova dva dokumenta.
