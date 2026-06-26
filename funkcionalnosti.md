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

Kako testiramo: prođemo ceo put registracije za mali model (~1 GB), veliki (7B, ~15 GB) i jedan eksterni API model, uz namerni prekid prenosa i proveru da checksum zaustavlja oštećen fajl. Kriterijum uspeha: ekspert registruje model bez čitanja dokumentacije, a prekinut prenos se sam oporavlja.

Definition of done: korisnik registruje model (upload / HF / eksterni API), dobije verziju i obavezne metapodatke (uklj. licencu), model je vidljiv u katalogu po zadatoj vidljivosti, a prekinut prenos se sam oporavlja.

Rizici: tehnički (veliki fajlovi, integritet) — nizak · compliance (provera licence modela pre objave) — srednji · zavisnost od partnera — niska.

## 2. Korišćenje modela (inference / chat)

Opcije:
- Interfejsi: chat UI (sektorski prilagođen) · REST API (OpenAI-kompatibilan) · poziv iz sveske
- Serving: vLLM · Triton Inference Server · HuggingFace TGI · ONNX Runtime (laki CPU modeli) · eksterni provajderi kroz proxy
- Prateće: SSE streaming · rate limiting i token budžeti · keširanje (Redis) · merenje potrošnje

Kako testiramo: isti 7B model podignemo na vLLM i TGI na istom GPU-u i pustimo load test (1/10/50 korisnika), mereći vreme do prvog tokena, p95 latenciju, propusnost i ponašanje pod preopterećenjem; chat UI testiramo sa ne-tehničkim korisnicima. Kriterijum: stabilan streaming pod opterećenjem i chat koji ne-tehnički korisnik koristi bez pomoći.

Definition of done: model se servira preko OpenAI-kompatibilnog API-ja i chat UI-ja, sa streamingom, rate limitom, token budžetom i merenjem potrošnje; ne-tehnički korisnik dobije odgovor bez pomoći.

Rizici: tehnički (GPU kapacitet, latencija pod opterećenjem) — srednji · operativni (održavanje serving stacka) — srednji.

## 3. Fine-tuning modela

Opcije:
- Pokretanje: vizard sa hiperparametrima · YAML šabloni · custom skripta iz sveske · API
- Stack: HuggingFace Transformers + PEFT (LoRA/QLoRA) · Axolotl · Unsloth · LLaMA-Factory
- Izvršavanje: SLURM job na PARADOX/ITE · Kubernetes Job za manje poslove
- Praćenje: MLflow · Weights & Biases · ClearML

Kako testiramo: isti bazni model i mali srpski instruction set provučemo kroz Axolotl, Unsloth i LLaMA-Factory, mereći trud konfiguracije, VRAM, trajanje i kvalitet pre/posle, i poredimo LoRA vs QLoRA. MLflow proveravamo preko veze ka klasteru (eliminacioni kriterijum); W&B samo ako MLflow podbaci. Kriterijum: ceo ciklus (config → trening → eval → registracija) prolazi bez ručnih intervencija.

Definition of done: korisnik pokrene fine-tuning iz šablona, posao ode na HPC/Kubernetes, prati napredak uživo, a rezultujući model se automatski registruje sa lineage-om ka baznom modelu i datasetu.

Rizici: tehnički (GPU sati, stabilnost treninga) — srednji · organizacioni (ko odobrava trošak GPU sati) — srednji.

## 4. Upravljanje datasetovima

Opcije:
- Upload: portal (presigned multipart u MinIO) · SDK/CLI · povezivanje izvora (Nacionalni data centar Kragujevac)
- Verzionisanje: DVC · LakeFS
- Validacija: provera formata, enkodiranja, heuristički PII sken
- GDPR: klasifikacija (4 nivoa) · saglasnosti · anonimizacija · lineage

Kako testiramo: za DVC i LakeFS odigramo upload → nova verzija → vraćanje na staru (i paralelnu anotaciju gde se razlikuju), uz uslov da korisnik ne vidi nijednu komandu alata. PII sken merimo na probnom datasetu sa namerno ubačenim ličnim podacima. Kriterijum: verzionisanje koje korisnik razume kao "verzija 1, verzija 2" bez ikakvog znanja o alatu ispod.

Definition of done: korisnik upload-uje dataset, dobije verziju, vrati se na staru, a sistem klasifikuje podatke i prijavi mogući PII pre nego što dataset može da se objavi ili podeli.

Rizici: compliance (PII, GDPR, osetljivi/zdravstveni podaci) — visok · tehnički (veliki datasetovi, anotacije) — srednji · organizacioni (ko je vlasnik dataseta) — srednji.

## 5. Pokretanje poslova na HPC (PARADOX, ITE)

Opcije:
- Interfejsi: forma u portalu · sveska sa SDK · raw API · zaključani šabloni za studente
- Konekcija: SSH + sbatch/squeue/sacct · slurmrestd REST API · Open OnDemand (komplementarno)
- Prenos: rsync (veliki fajlovi) · SFTP (skripte) · S3/MinIO na HPC strani
- Bezbednost: Vault SSH sertifikati · izolovan bridge servis · kvote sa rezervacijom i obračunom

Kako testiramo: razvijamo prvo protiv lokalnog mock SLURM-a, pa na pravom klasteru, sa fokusom na otpornost: prekid SSH veze između sbatch-a i čitanja job ID-a (dupli posao?), restart bridge servisa dok poslovi traju (ponovno usvajanje preko sacct?) i prekid rsync prenosa (nastavak?). slurmrestd testiramo samo ako ga operateri omoguće — SSH putanja je primarna. Kriterijum: nijedan prekid ne sme da proizvede dupli posao ili izgubljen rezultat.

Definition of done: korisnik pokrene job iz portala, vidi status i logove uživo, preuzme rezultat, a sistem sprečava dupli submit posle prekida veze i oporavlja stanje poslova posle restarta bridge servisa.

Rizici: zavisnost od operatera klastera (slurmrestd, S3/MinIO, kvote nisu zagarantovani) — visoka · tehnički (otpornost na prekide veze) — srednji · bezbednosni (pristup klasteru) — srednji.

## 6. Interaktivni rad u sveskama (eksperti i studenti)

Opcije:
- JupyterHub (SSO preko Keycloak-a): KubeSpawner · SlurmSpawner
- Alternative: JupyterLite (browser, bez servera) · code-server (pun IDE) · Marimo (edukativne sveske) · Google Colab (samo javni edukativni sadržaj)
- Politike: limiti po korisniku, gašenje neaktivnih sesija, GPU po ulozi

Kako testiramo: merimo vreme od „Otvori svesku" do prve ćelije za KubeSpawner (sekunde) i SlurmSpawner (minute) — to bira podrazumevani, i simuliramo pun klaster radi poruke korisniku. Istu lab vežbu damo u JupyterLite i JupyterHub-u, code-server probnim naprednim korisnicima, i testiramo gašenje neaktivnih GPU sesija (rad se ne sme izgubiti). Kriterijum: sveska se otvara dovoljno brzo da ne prekida tok rada, a neaktivne GPU sesije se oslobađaju bez gubitka rada.

Definition of done: sveska se otvara dovoljno brzo da ne prekida tok rada, izolovana je po korisniku sa kvotama, a neaktivne GPU sesije se oslobađaju bez gubitka rada.

Rizici: operativni (GPU kvote, idle culling) — srednji · tehnički (spawn latencija na SLURM-u) — srednji.

## 7. Pretraga i katalog resursa

Opcije:
- Pretraga: OpenSearch (full-text, faceti) · pgvector (semantička) · Qdrant/Weaviate (ako semantika preraste pgvector)
- Sektorski pogledi: konfiguracioni filteri nad istim katalogom
- Provenance: oznaka izvora (lokalno / Pharos / IT4LIA / CKAN)

Kako testiramo: napunimo probni katalog sa 100+ stavki i ~30 realnih upita, obavezno na srpskom i ćirilicom i latinicom („модел за здравство" = „model za zdravstvo"), pa poredimo čist full-text (OpenSearch) sa hibridom + semantika (pgvector) — ocena: da li je pravi rezultat u prvih 5. Kriterijum: pretraga radi podjednako na oba pisma i jezika, a sektorski filter nikad ne propusti neodobren resurs.

Definition of done: pretraga vraća tačan rezultat u prvih 5 na oba pisma i oba jezika, svaki resurs ima jasnu oznaku porekla, a sektorski filter nikad ne propusti neodobren resurs.

Rizici: tehnički (kvalitet semantike na srpskom) — srednji · bezbednosni (filter pristupa u pretrazi) — srednji.

## 8. Obuke i edukacija (LMS)

Opcije:
- LMS: Open edX · Moodle — iza SAIFA API-ja
- Lab vežbe: kurirane sveske + pravi podaci iz MinIO + zaključani SLURM šabloni na ITE
- Jezici: srpski (ćir./lat.) i engleski

Kako testiramo: isti probni kurs (lekcije + jedna hands-on lab vežba) postavimo na Open edX i Moodle i merimo trošak integracije (upis, napredak, sertifikat kroz API, bez izlaska iz portala) i ocenu probnih studenata; lab vežbu testiramo ceo lanac upis → sveska → dataset → posao na klasteru → provera → napredak. Kriterijum: student završi vežbu bez pomoći, a mi procenimo koji LMS traži manje održavanja (Moodle lakši za hosting, Open edX bogatiji).

Definition of done: student završi lab vežbu kroz portal bez izlaska iz SAIFA okruženja, sa upisanim napretkom i (opciono) sertifikatom.

Rizici: organizacioni (obim kurseva nepoznat dok klijenti ne kažu) — srednji · operativni (održavanje LMS-a) — srednji.

## 9. Federacija sa EuroHPC (Pharos, IT4LIA)

Opcije:
- Sync kataloga: zakazan + event-driven · izvoz modela/datasetova/benchmarka · uvoz sa oznakom porekla
- Apstrakcija: adapter po partneru (Pharos API, IT4LIA API, CKAN, Dataverse)
- Identitet: Keycloak brokering ka EuroHPC AAI · eduGAIN preko AMRES-a

Kako testiramo: u dva koraka — prvo mock Pharos po objavljenoj šemi (uključujući namerno pokvarene odgovore: sync ne sme upisati ništa polovično), pa test protiv pravog Pharos sandbox-a čim ga dobijemo (spremnost partnera je najveća nepoznanica). Kriterijum: model označen za objavu pojavi se u (mock) partnerskom katalogu bez ručne intervencije, a nijedan kvar partnera ne ošteti naš katalog.

Definition of done: model označen za objavu pojavi se u (mock) partnerskom katalogu bez ručne intervencije, uvezeni resursi nose oznaku porekla, a nijedan kvar partnera ne ošteti naš katalog.

Rizici: zavisnost od partnera (zrelost Pharos/IT4LIA API-ja i metapodataka) — visoka · tehnički (validacija šeme, otpornost na kvarove) — srednji.

## 10. Benchmarking i evaluacija

Opcije:
- Harness: lm-evaluation-harness + sektorski benchmark datasetovi (srpski jezik)
- Izvršavanje: platformski job na HPC ili GPU pool-u
- Leaderboard: objava po verziji benchmarka, uz odobrenje

Kako testiramo: lm-evaluation-harness na dva poznata modela, proveravajući reproduktivnost (dva pokretanja → praktično identično) i cenu (GPU sati po evaluaciji), i merimo koliko je posla uvesti custom srpski task. Kriterijum: evaluacija je ponovljiva, automatizovana kao običan platformski posao i košta predvidivo.

Definition of done: evaluacija je ponovljiva, pokreće se kao običan platformski posao, košta predvidivo, a rezultat ide na leaderboard tek uz odobrenje (vidi 13).

Rizici: tehnički (reproduktivnost) — srednji · trošak (GPU sati po evaluaciji) — srednji.

## 11. Upravljanje korisnicima i pristupom

Opcije:
- Identity provider: Keycloak (OIDC/OAuth2/SAML2) · prijava lokalna, eduGAIN/AMRES, EuroHPC identitet
- Uloge (RBAC): platform-admin, data-admin, model-reviewer, ai-expert, advanced-user, basic-user, student, mentor/instruktor, viewer, federation-partner
- Atributi (ABAC): sektorske sertifikacije, afilijacija → kvote, GDPR saglasnosti
- API ključevi: generisanje, scope-ovanje, rotacija, opoziv

Kako testiramo: Keycloak realm sa svim ulogama + automatizovani testovi izolacije koji za svaku ulogu pokušavaju svaku operaciju nad tuđim resursima (sve mora „zabranjeno") i ostaju trajno u CI-ju; eduGAIN sa AMRES test identitetom, i ciklus „napravi → koristi → opozovi API ključ" (opoziv deluje odmah). Kriterijum: nijedan test izolacije ne prolazi pogrešno, ni jednom.

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

Kako testiramo: monitoring stack dižemo već u dev okruženju i proveravamo da li iz njega odgovaramo na EuroHPC pitanja (GPU sati po projektu, energija, preuzimanja dataseta) — ako ne možemo, fali metrika; mrežni nadzor proveravamo malim NetFlow ogledom (softflowd → nfdump → Grafana), a audit pokušajem izmene/brisanja zapisa (ne sme uspeti ni kao admin baze). Kriterijum: svaki compliance izveštaj dobija se iz sistema bez ručnog kopanja po logovima.

Definition of done: svaki izveštaj koji compliance/EuroHPC traži dobija se iz sistema bez ručnog kopanja po logovima, a audit zapis ne može da se izmeni ni obriše čak ni kao admin baze.

Rizici: compliance (retencija, integritet audita) — srednji · operativni (održavanje monitoring stacka) — nizak.

## 13. Kolaboracija i timski rad

Opcije:
- Projekti i timski workspace-ovi: deljenje modela, datasetova i rezultata unutar tima
- Review/approval: odobravanje modela i datasetova, komentari, validation reports
- Kod-kolaboracija: interni GitLab/Gitea (merge request, review) — već potreban za DVC

Kako testiramo: prototip team vidljivosti (model vidljiv timu, nevidljiv ostalima) damo istraživačkoj grupi (3–5 ljudi) da prođe ciklus jedan trenira / drugi pregleda / treći odobrava, krećući od minimuma i dodajući samo što grupa zatraži. Kriterijum: review ciklus od treninga do odobrene objave prođe kroz platformu bez „sa strane" komunikacije.

Definition of done: review ciklus od treninga do odobrene objave prođe kroz platformu, sa jasnim ko-šta-odobrava (vidi RACI ispod), bez komunikacije "sa strane".

Rizici: organizacioni (definisanje ownership i approval procesa) — srednji · tehnički (team vidljivost) — nizak.

## 14. Workflow orchestration (ML pipeline-ovi)

Opcije:
- Kubeflow Pipelines · Apache Airflow · Argo Workflows · sopstveni job-scheduler (dovoljan za jednostavne lance, već u arhitekturi)

Kako testiramo: referentni pipeline od četiri koraka (preprocesiranje → trening → evaluacija → registracija) prvo izvedemo sopstvenim job-scheduler-om sa ulančavanjem preko događaja; ako je čitljivo i pouzdano, generički engine ne uvodimo. Tek ako klijenti traže složenije lance (grananje, retry, paralelizam) radimo PoC sa Argo Workflows. Kriterijum: odluku vodi složenost koju klijenti stvarno traže u upitniku, ne zamišljena.

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
