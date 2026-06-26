# SAIFA AI Gateway — Popis tehnoloških mogućnosti po slojevima

Ovaj dokument je tehnički popis svih razmatranih opcija za svaki sloj infrastrukture platforme. Cilj je da se sve opcije testiraju, uporede sa potrebama klijenata, i da se na osnovu toga odaberu konačne tehnologije. Dokument ne donosi konačne odluke — služi kao osnova za evaluaciju. Sažeta odluka „gotovo ili custom" po sloju (skraćeni pregled) je u `SAIFA_gotove_komponente_po_slojevima.md`.

D
## 1. Frontend i UI sloj

- Framework: Next.js (React, SSR/SSG) · React SPA (Vite) · Vue 3 / Nuxt 3 · statički HTML/CSS/JS + Vite
- Biblioteke komponenti: Tailwind CSS + shadcn/ui · Material UI · Ant Design · PrimeVue
- Upravljanje stanjem: TanStack Query + Zustand · Redux Toolkit · Pinia
- Internacionalizacija (srpski ćir./lat. + engleski): next-intl · i18next · vue-i18n
- Pristup za različite korisnike: odvojeni pogledi po ulozi (ekspert / ne-tehnički / admin / student) iz jednog codebase-a; sektorski dashboardi (zdravstvo, energetika, ekologija, jezik/kultura) vođeni konfiguracijom, ne kodom

## 2. Interaktivni kod-interfejs (za ekspertske korisnike)

- JupyterHub (self-hosted): višekorisničko notebook okruženje, izolacija po korisniku, kvote
- Spawner za JupyterHub: KubeSpawner (pod po korisniku na Kubernetes-u) · SlurmSpawner (notebook kao SLURM job direktno na klasteru)
- JupyterLite: notebook u browseru (Wasm), bez servera — za obuke i demo
- code-server (VS Code u browseru): pun IDE za napredne korisnike
- Google Colab: eksterna opcija "otvori kopiju" samo za javne edukativne notebook-ove
- Marimo / Observable: lagane reaktivne alternative za edukativni sadržaj
- Politike: limiti CPU/RAM/GPU po korisniku, gašenje neaktivnih sesija (idle culling), GPU pristup po nivou uloge

## 3. API Gateway i BFF sloj

- Gateway: Kong Gateway (OSS) · Traefik · NGINX (+OpenResty) · custom FastAPI gateway
- BFF pristup: jedan gateway za sve persone vs. odvojeni BFF po tipu korisnika
- Rate limiting: po nivou korisnika, po tipu resursa (inference, HPC submit, federacija)
- Verzionisanje API-ja: URL putanja (/v1/, /v2/) vs. header verzionisanje; politika deprecation-a (Sunset/Deprecation headeri)
- Autentikacija: validacija JWT tokena (Keycloak) na gateway-u; kratkotrajni interni tokeni za komunikaciju servis-servis

## 4. Backend servisi

- Arhitektura: modularni monolit sa jasnim granicama domena vs. mikroservisi od starta; kriterijumi za izdvajanje servisa
- Jezik/framework: Python (FastAPI) · Java (Spring Boot) · Go (Gin) za high-throughput proxy
- Servisi (domeni): identity · model-registry · dataset-management · fine-tuning-orchestrator · job-scheduler · deployment-manager · hpc-bridge · inference-proxy · catalogue-aggregator · federation-sync · training-platform · notification · audit · **support** (ticketing: kreiranje issue-a, prosleđivanje timu, kontekst iz audit loga)
- Razdvajanje jobs vs deployments: **jobs** su kratkotrajni batch poslovi (SLURM/K8s, imaju terminalno stanje — završeno/neuspeh/zaustavljeno) · **deployments** su dugotrajni servisi (inference endpoint, web aplikacija — nemaju "završeno", imaju destroy). Ne smeju da dele isti sistem akcija/stanja jer to vodi UX konfuziji i orphan resursima (lekcija iz Themelio ADR-0012). Zaseban API, zasebne tabele, soft-delete sa istorijom.
- Per-resource secret za pisanje logova/statusa: svaki job/deployment dobija jedinstven token (hash u bazi); samo update log/status endpoint traži taj token, a list/get/destroy proveravaju pravo pristupa korisnika. Sprečava da bilo ko sa job ID-em čita/piše tuđe logove.

## 5. HPC integracija (PARADOX, ITE)

- Protokol za SLURM: SSH + sbatch/squeue/sacct · SLURM REST API (slurmrestd) · Open OnDemand (komplementaran portal)
- Prenos fajlova: rsync preko SSH (veliki fajlovi) · scp/SFTP (mali fajlovi) · S3-kompatibilan API na HPC strani (MinIO)
- Kontejneri na HPC: Singularity/Apptainer, build na platformi → push u registry → pull na klasteru
- Bezbednost: SSH sertifikati (ne statički ključevi) preko Vault SSH engine-a · namenski servisni nalog po klasteru · mrežna izolacija bridge servisa
- Kvote: CPU/GPU sati po korisniku i projektu; provera pre slanja, rezervacija, obračun po stvarnoj potrošnji
- Rutiranje između klastera: po tipu resursa, lokaciji podataka, afilijaciji korisnika, dubini reda čekanja, ceni

## 6. ML runtime i inference

- Serving apstrakcija (Kubernetes sloj iznad runtime-a): KServe (`InferenceService` CRD, standardizovan deploy/rollback/skaliranje, serving profili po modelu) · Ray Serve · goli Helm/K8s manifest po modelu. Runtime (ispod) se bira zasebno.
- Serving runtime: vLLM · Triton Inference Server · HuggingFace TGI · TorchServe · ONNX Runtime · Ollama
- Serving profili kao metapodatak modela (ne biraju ih korisnici po zahtevu): single-GPU · single-node multi-GPU (vLLM tensor parallelism) · multi-node multi-GPU (vLLM pipeline parallelism, KServe `workerSpec`) — topologija je deo kataloga modela
- Fine-tuning stack: HuggingFace Transformers + PEFT/LoRA · Axolotl · Unsloth · LLaMA-Factory
- Strategija treninga: puni fine-tuning vs. parameter-efficient (LoRA/QLoRA) — odluka po veličini dataseta i budžetu
- Praćenje eksperimenata: MLflow (self-hosted, primarni izbor) · Weights & Biases (samo kao rezerva ako MLflow podbaci) · ClearML
- Evaluacija i benchmarking: lm-evaluation-harness · sektorski benchmark datasetovi · leaderboard sa procesom odobravanja
- Inference API: streaming tokena (SSE) · batching · upravljanje kontekstom chat sesija

## 7. Sloj podataka

- Relaciona baza: PostgreSQL (+ PgBouncer pooling); multi-tenancy: row-level security · schema-per-tenant · database-per-tenant
- Vektorska baza: pgvector · Qdrant · Weaviate · Milvus
- Objektno skladište: MinIO (S3-kompatibilno) — modeli, datasetovi, artefakti, staging za HPC
- Interfejs skladišta iznad MinIO-a: sopstveni registar API (presigned multipart upload) vs. Git LFS-native (svaki resurs = Git repo, LFS pointer u repou, MinIO kao S3 backend ispod) vs. Xet (content-addressed, chunk-level deduplikacija — isti protokol koji HuggingFace Hub sada koristi po defaultu). Git LFS/Xet razmatrani jer ih Pharos koristi (vidi sekciju 16 za odluku).
- Message queue: RabbitMQ · Apache Kafka · NATS JetStream · Redis/Valkey Streams
- Keš: Valkey/Redis (API keš, sesije, rate-limit brojači, stanje kvota); Sentinel vs. Cluster mod — napomena: Redis je promenio licencu (2024), Valkey je open-source fork pod BSD, prirodniji za suvereno okruženje (vidi sekciju 16)
- Verzionisanje datasetova: DVC · LakeFS
- Katalog datasetova: sopstveni katalog · CKAN · Dataverse · OpenMetadata; DCAT izvoz za EU portale

## 8. Identitet i kontrola pristupa

- Identity provider: Keycloak (OIDC, OAuth2, SAML2) · alternativе (razmotrene, odbačene): Authentik, Zitadel
- Akademska federacija: eduGAIN preko AMRES-a (SAML brokering)
- EuroHPC identitet: Keycloak identity brokering ka EuroHPC AAI / MyAccessID
- RBAC uloge: platform-admin · data-admin · model-reviewer · ai-expert · advanced-user · basic-user · student · viewer · federation-partner
- ABAC atributi: sektorske sertifikacije (npr. zdravstvo) · nivo afilijacije (kvote) · GDPR saglasnosti · tenant
- API ključevi: generisanje, scope-ovanje, rotacija, opoziv
- Servis-servis autentikacija: kratkotrajni JWT (client credentials, token exchange) — bez statičkih tajni

## 8a. Organizacije, timovi i višenivovska kvota

- Entitet organizacija: životni ciklus `Na čekanju → Aktivna / Odbijeno`; tip (akademska / istraživačka / kompanija-SME), sektor, predstavnik kao upravljačka uloga
- Interne strukture (timovi/odeljenja): organizacija može sadržati timove kao pod-entitete (npr. ETF → Katedra za računarsku tehniku, ETF → LERMA lab). Tim ima svog lidera (delegiran od predstavnika), zasebnu pool kvotu iz ukupne kvote organizacije, i sopstvenu `team` vidljivost resursa. MVP: ravna organizacija bez timova; Pilot: timovi sa delegiranom kvotom.
- Tronivovska kvota (Pilot): pool organizacije (dodeljuje platform-admin) → pool tima (raspodeljuje predstavnik) → lična kvota člana (raspodeljuje lider tima ili predstavnik); MVP zadržava dvonivovski model (organizacija → član)
- Resursi kvote: CPU sati · GPU sati · storage; provera i rezervacija pre svakog posla, obračun po stvarnoj potrošnji
- Tokovi: zahtev člana za povećanje kvote → odluka lidera tima ili predstavnika; zahtev organizacije → odluka platform-admina
- Dinamičke promene kvote: svaka promena kvote ima trenutni efekat (Redis/Valkey rate-limit brojači ažurirani odmah); smanjenje ispod trenutne potrošnje ne prekida aktivne poslove, ali blokira nove
- Project/grant accounting: sopstveni model kvota (gore opisan) vs. Waldur (open-source platforma za upravljanje grantovima, projektima i alokacijama resursa). Waldur razmotriti ozbiljno jer ga Pharos koristi — accounting model bi se poklopio sa roditeljskom AI fabrikom (Themelio sinhronizuje korisnike AAI → Waldur, kvote su project-scoped preko Waldur-a). Otvorena odluka, traži zaseban ADR.

## 8b. Edukaciona platforma / LMS

Cela sekcija UC-ova (kursevi, upis, napredak, lab vežbe, sertifikati, kreiranje kursa, praćenje studenata) oslanja se na LMS sloj koji je dosad bio pomenut samo u decision tree-u. Ovde dobija mesto kao komponenta.

- LMS: Moodle (lakši hosting) · Open edX (bogatiji sadržaj, learning paths) — iza SAIFA API-ja, korisnik ne izlazi iz portala
- Lab vežbe: kurirane Jupyter sveske + pravi podaci iz MinIO (read-only montaža) + zaključani SLURM/Kubernetes šabloni za studente
- Sertifikati: generisanje PDF-a sa potpisom platforme po završetku kursa
- Resource limiti za studente: predstavnik akademske institucije postavlja CPU/RAM/GPU i trajanje notebook sesije na nivou cele institucije (ne po kursu) u okviru pool-a organizacije
- Jezici: srpski (ćir./lat.) i engleski

## 8c. Projekti i kolaboracija

- Entitet projekat: vlasnik + članovi; uloge `member` / `co-owner`; vidljivost privatno/tim/javno
- Članstvo: pozivnica sa stanjem (na čekanju / prihvaćeno / odbijeno / isteklo); uklanjanje člana oduzima pristup resursima projekta
- Deljenje artefakata: modeli, datasetovi i rezultati vidljivi unutar tima preko `team` vidljivosti, bez slanja fajlova "sa strane"

## 9. Infrastruktura i DevOps

- Kontejnerizacija: Docker · Podman (rootless)
- Orkestracija: k3s (pilot) · puni Kubernetes (produkcija) · Docker Compose (samo lokalni razvoj)
- Service mesh (opciono, kasnije): Istio · Linkerd
- CI/CD: GitLab CI · GitHub Actions; deploy: ArgoCD (GitOps)
- IaC: Terraform · Helm · Ansible
- Container registry: Harbor (self-hosted, skeniranje, potpisivanje) · DockerHub
- Observability: Prometheus + Grafana (metrike, alarmi) · Loki (logovi) · Tempo/Jaeger + OpenTelemetry (tracing) · Sentry (greške)
- Tajne: HashiCorp Vault — dinamički DB kredencijali, SSH sertifikati za HPC, Kubernetes auth, rotacija

## 10. Federacija sa EuroHPC (Pharos, IT4LIA)

- Sinhronizacija kataloga: zakazana (cron) i event-driven; izvoz SAIFA modela/datasetova/benchmarka, uvoz eksternih
- Apstrakcija izvora: CatalogueSource interfejs po partneru (Pharos API, IT4LIA API, CKAN, Dataverse)
- Bezbednost: mTLS + kredencijali po partneru, validacija šeme pre svake razmene, rate limiting
- Plugin interfejsi za proširenja: ModelProvider (HuggingFace, OpenAI, Anthropic, Mistral, Groq, Replicate...) · ComputeBackend (PARADOX, ITE, Kubernetes, cloud) · CatalogueSource · sektorski pogledi kao konfiguracija
- Feature flags: Unleash (self-hosted) · env varijable; ključni flagovi: ENABLE_HPC_SUBMISSION, ENABLE_FINE_TUNING, ENABLE_FEDERATION_SYNC, ENABLE_JUPYTER_HUB, ENABLE_TRAINING_PLATFORM
- Eksterni ModelProvider-i (OpenAI, Anthropic, Mistral, Groq, Replicate) su plative eksterne usluge — koriste se kroz proxy uz token budžet, nisu zamena za lokalne modele

## 11. Bezbednost i usklađenost (presek kroz sve slojeve)

- Zero-trust model: svaki zahtev autentifikovan na svakom sloju, bez poverenja u internu mrežu
- Multi-tenant izolacija: PostgreSQL RLS politike + filtriranje u aplikaciji + ACL u pretrazi
- GDPR: klasifikacija podataka (4 nivoa) · evidencija saglasnosti · automatizovan DSAR izvoz · pravo na brisanje · DPIA procedura za osetljive datasetove
- Integritet artefakata: SHA-256 checksum · opciono GPG/cosign potpisivanje · provera pre serviranja
- Audit log: append-only, hash-chain, pokriva sve CRUD operacije, HPC poslove, inference pozive, federaciju; retencija min. 2 godine

## 11a. Lifecycle i governance resursa

- Jedinstvena mašina stanja resursa: `Draft → Na recenziji → Odobren → Objavljen lokalno → Objavljen (Pharos) → Deprecated → Arhiviran`; važi za modele, datasetove i (gde ima smisla) workflow-e i kurseve
- Servisni aliasi (npr. `production`) odvojeni su od lifecycle stanja — rollback aliasa ne menja stanje verzije
- Review uloge i red za pregled: Data administrator (datasetovi) · Model reviewer (modeli); odluke `Odobri` / `Odbij` / `Vrati na doradu`
- Obavezno pre objave: popunjena kartica resursa (model/dataset card), ručna provera kompatibilnosti licence (dataset → izvedeni model), rezultati evaluacije za modele
- Moderacija: platform-admin može skinuti javni resurs (vidljivost `public → private`) uz obavezan razlog u audit logu — moderacija je post-objava, ne ulazna kapija
- Lineage — dva nivoa poverenja: automatski uhvaćen kroz platformski posao (pouzdan) vs. ručno unet za spolja uvezen artefakt (`lineage: nepotvrđen`)
- Verzionisanje: i modeli i datasetovi imaju verzije (v1 → v2 → vraćanje na raniju); ranije verzije ostaju nepromenljive i dostupne

## 12. Metapodaci i governance (centralni deo sistema, ne sporedni)

- Standardi i šeme: DCAT (izvoz ka EU portalima otvorenih podataka) · schema.org (struktuirani podaci na vebu) · DataCite (metapodaci istraživačkih podataka, DOI)
- Kartice resursa: model card i dataset card (svrha, ograničenja, podaci za trening, poznati rizici) — obavezne pre objave; svaka verzija resursa nosi obavezan **changelog** koji opisuje šta se promenilo u odnosu na prethodnu verziju (format slobodan tekst u MVP, strukturisan u Pilot)
- Lineage i provenance: odakle resurs dolazi i kroz koje transformacije je prošao (bazni model → fine-tune → verzija; izvorni dataset → obrada → verzija)
- Obavezni metapodaci po resursu: owner · license · **visibility** (public/private/team/platform) · domain · language · provenance/izvor · version · **changelog** · quality status · validation status · GDPR klasa · access policy · publication status
- Vidljivost resursa — četiri nivoa: `private` (samo vlasnik/tim koji ga je kreirao) · `team` (projekat/tim) · `platform` (svi prijavljeni korisnici platforme, bez anonimnih) · `public` (i anonimni posetioci)
- Tipovi resursa u katalogu (svi indeksirani u jednom upitu): modeli · datasetovi · kontejneri (Docker/Apptainer image u Harbor-u) · Jupyter sveske/šabloni · workflow šabloni (pipeline definicije) · benchmark setovi · kursevi/learning pathovi · šabloni poslova (job šabloni) · alati/plugin-ovi (ModelProvider-i, ComputeBackend-i)
- Kompatibilnost licenci: provera pre objave (da li licenca dataseta dozvoljava objavu izvedenog modela)
- Quality score i PII klasifikacija: automatska oznaka kvaliteta i nivoa osetljivosti, vidljiva u katalogu
- Source of truth: lokalni SAIFA registar je merodavan za lokalne resurse; federisani resursi se ne mešaju u istu bazu bez oznake porekla (vidi blueprint ispod)

## 13. Arhitekturni blueprint (tekstualni — ko sa kim komunicira)

- Korisnik → Frontend → API Gateway (Keycloak JWT validacija ovde) → Backend servisi.
- Policy decisions (RBAC/ABAC) donose se u backend-u pri svakoj operaciji, oslonjene na Keycloak uloge/atribute; pretraga dodatno filtrira po ACL-u (nikad ne prikazuje neodobren resurs).
- Source of truth za metapodatke: PostgreSQL registar (modeli, datasetovi, katalog). Težine i artefakti: MinIO. Identitet: Keycloak. Tajne: Vault. Nijedan drugi servis ne drži paralelnu "istinu".
- HPC posao: backend (job-scheduler) → hpc-bridge → SSH/sbatch ka PARADOX/ITE; status nazad preko sacct; rezultati u MinIO. Jobs (batch, terminalno stanje) i deployments (dugotrajni servisi) vode se kroz zasebne entitete (vidi sekciju 4).
- Inference: serving endpoint (KServe/vLLM ili goli vLLM) je **interni** — korisnik ga nikad ne zove direktno. Sav javni inference saobraćaj ide kroz API Gateway, gde se primenjuju token kvote, autorizacija i audit. Time se sprečava zaobilaženje kvota direktnim pozivom runtime-a, a serving topologija (single/multi-GPU) može da se menja bez promene na klijentu.
- Audit: svaki servis piše u append-only audit log (hash-chain); audit je samo-za-upis, nema brisanja.
- Federacija: federation-sync čita/piše preko CatalogueSource adaptera po partneru; uvezeni resursi ulaze u katalog isključivo sa oznakom porekla i nikad ne prepisuju lokalni source of truth.

## 14. Default PoC stack (odakle počinjemo)

Popis je namerno širok, ali tim ne sme da izgubi vreme testirajući sve. Ovo je podrazumevani izbor sa kog krećemo; menja se samo ako test ili upitnik daju razlog.

| Sloj | Preporučeni PoC izbor |
| --- | --- |
| Frontend | Next.js ili React SPA |
| IAM | Keycloak |
| Backend | FastAPI modularni monolit |
| DB | PostgreSQL |
| Objektno skladište | MinIO |
| Pretraga | OpenSearch + kasnije pgvector |
| Notebook | JupyterHub + KubeSpawner |
| HPC bridge | SSH/sbatch prvo, slurmrestd ako je dostupan |
| Inference | vLLM kao prvi kandidat |
| Experiment tracking | MLflow (self-hosted) |
| Monitoring | Prometheus / Grafana / Loki |
| Tajne | Vault |
| Deployment | k3s za pilot |

## 15. Decision tree (kada šta)

- Posao je interaktivan i kratak (chat, inference, mala obrada) → Kubernetes GPU pool. Posao je veliki/batch (trening, evaluacija) → SLURM (PARADOX/ITE).
- Sveska za obuku/demo bez teškog računanja → JupyterLite (bez servera). Sveska za pravi rad sa GPU/podacima → JupyterHub (KubeSpawner; SlurmSpawner kad treba direktno na klaster). Napredni korisnik traži pun IDE → code-server.
- Klijent najavi strukturirane kurseve sa sertifikatima → LMS (Moodle za lakši hosting, Open edX za bogatiji sadržaj). Klijent traži samo materijale → biblioteka tutorijala/svezaka, bez LMS-a.
- Pipeline je jednostavan linearni lanac → sopstveni job-scheduler. Pipeline traži grananje/retry/paralelizam → Argo Workflows (PoC), pa tek onda poređenje sa Airflow/Kubeflow.
- Semantička pretraga staje u pgvector → pgvector. Preraste obim/performanse → zaseban vektorski servis 
(Qdrant/Weaviate).

## 16. Licence i nabavka (presek)

- Feature flags: LaunchDarkly (SaaS, plaća se) ≠ Unleash (self-hosted, open-source)
- Experiment tracking: W&B cloud (SaaS) ≠ MLflow (self-hosted) 
- Keš/queue: Redis je promenio licencu 2024 (RSALv2/SSPL, više nije OSI-odobren) ≠ Valkey (BSD, open-source fork pod Linux Foundation) — za suvereno okruženje biramo Valkey.
- Interfejs skladišta: Git LFS-native / Xet (pristup koji Pharos/Themelio koristi) **odbijen kao primarni interfejs** — zahteva od korisnika `git` i razumevanje LFS pointer-a, što je suprotno zahtevu da se infrastruktura sakrije od ne-tehničkih korisnika (lekar, neko iz poljoprivrede). MinIO ostaje S3 sloj i source of truth za artefakte, kompatibilan sa Pharos LFS/Xet backend-om preko federacijskog adaptera (sekcija 10). Napomena: za Xet ne postoji open-source CAS server (protokol je javan, server bi se gradio od nule) — dodatni razlog da se ne usvaja kao primarni put. Federacija ka njima je granica koja se rešava adapterom, ne usvajanjem njihovog modela skladišta.
- Accounting/grant management: Waldur (open-source, self-hosted, koristi ga Pharos) vs. sopstveni dvonivovski model kvota (§8a) — otvorena ADR; usvajanje Waldur-a bi poravnalo accounting sa roditeljskom fabrikom, ali nosi sopstveni operativni teret.
- API gateway: AWS API Gateway (managed cloud) vezuje za cloud i nosi lock-in — koristimo Kong/Traefik/NGINX.
- Eksterni ModelProvider-i (OpenAI, Anthropic, Mistral, Groq, Replicate) su plative eksterne usluge — koriste se kroz proxy uz token budžet, nisu zamena za lokalne modele.

---

Sledeći korak: za svaki sloj testirati navedene opcije (počev od default PoC stacka), mapirati zahteve klijenata na mogućnosti, i dokumentovati odluku sa obrazloženjem (reuse-vs-build, trošak, operativna složenost) i razlogom zašto ostale opcije nisu izabrane.
