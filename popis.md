# SAIFA AI Gateway — Popis tehnoloških mogućnosti po slojevima

Ovaj dokument je tehnički popis svih razmatranih opcija za svaki sloj infrastrukture platforme. Cilj je da se sve opcije testiraju, uporede sa potrebama klijenata, i da se na osnovu toga odaberu konačne tehnologije. Dokument ne donosi konačne odluke — služi kao osnova za evaluaciju.

Da popis ne bi ostao samo lista bez smera, dodate su tri stvari koje vode izbor: vodeći princip (ispod), kriterijum eliminacije po sloju (šta diskvalifikuje opciju), i preporučeni "default PoC stack" — odakle počinjemo dok upitnik ne kaže drugačije.

**Vodeći princip:** reuse before build, integrate before replace, custom only where SAIFA-specific value exists. Gateway je integraciona platforma, a ne jedna monolitna aplikacija — gradimo sami samo onaj sloj koji nosi specifičnu SAIFA vrednost (orkestracija, sektorski pogledi, onboarding, EuroHPC federacija).

## 1. Frontend i UI sloj

- Framework: Next.js (React, SSR/SSG) · React SPA (Vite) · Vue 3 / Nuxt 3 · statički HTML/CSS/JS + Vite
- Biblioteke komponenti: Tailwind CSS + shadcn/ui · Material UI · Ant Design · PrimeVue
- Upravljanje stanjem: TanStack Query + Zustand · Redux Toolkit · Pinia
- Internacionalizacija (srpski ćir./lat. + engleski): next-intl · i18next · vue-i18n
- Pristup za različite korisnike: odvojeni pogledi po ulozi (ekspert / ne-tehnički / admin / student) iz jednog codebase-a; sektorski dashboardi (zdravstvo, energetika, ekologija, jezik/kultura) vođeni konfiguracijom, ne kodom
- Kriterijum eliminacije: otpada sve bez zrele i18n podrške za oba pisma i sve što otežava jedan codebase sa pogledima po ulozi; managed/cloud UI hosting otpada zbog zahteva za self-hosting.

## 2. Interaktivni kod-interfejs (za ekspertske korisnike)

- JupyterHub (self-hosted): višekorisničko notebook okruženje, izolacija po korisniku, kvote
- Spawner za JupyterHub: KubeSpawner (pod po korisniku na Kubernetes-u) · SlurmSpawner (notebook kao SLURM job direktno na klasteru)
- JupyterLite: notebook u browseru (Wasm), bez servera — za obuke i demo
- code-server (VS Code u browseru): pun IDE za napredne korisnike
- Google Colab: eksterna opcija "otvori kopiju" za javne edukativne notebook-ove
- Marimo / Observable: lagane reaktivne alternative za edukativni sadržaj
- Politike: limiti CPU/RAM/GPU po korisniku, gašenje neaktivnih sesija (idle culling), GPU pristup po nivou uloge
- Kriterijum eliminacije: Google Colab samo za javni edukativni sadržaj (kontrolisani podaci ne smeju van okruženja); otpada svaka opcija bez izolacije po korisniku i kvota.

## 3. API Gateway i BFF sloj

- Gateway: Kong Gateway (OSS) · Traefik · NGINX (+OpenResty) · custom FastAPI gateway · AWS API Gateway
- BFF pristup: jedan gateway za sve persone vs. odvojeni BFF po tipu korisnika
- Rate limiting: po nivou korisnika, po tipu resursa (inference, HPC submit, federacija)
- Verzionisanje API-ja: URL putanja (/v1/, /v2/) vs. header verzionisanje; politika deprecation-a (Sunset/Deprecation headeri)
- Autentikacija: validacija JWT tokena (Keycloak) na gateway-u; kratkotrajni interni tokeni za komunikaciju servis-servis
- Kriterijum eliminacije: AWS API Gateway otpada (nije self-hosted, vezuje za cloud); otpada sve što teško integriše Keycloak JWT validaciju, ima slabu observability ili nameće veliki operativni teret malom timu. Custom FastAPI gateway gradimo tek ako standardni gateway ne pokrije potrebe — ne gradimo bez potrebe.

## 4. Backend servisi

- Arhitektura: modularni monolit sa jasnim granicama domena vs. mikroservisi od starta; kriterijumi za izdvajanje servisa
- Jezik/framework: Python (FastAPI) · Java (Spring Boot) · Go (Gin) za high-throughput proxy
- Servisi (domeni): identity · model-registry · dataset-management · fine-tuning-orchestrator · job-scheduler · hpc-bridge · inference-proxy · catalogue-aggregator · federation-sync · training-platform · notification · audit
- Kriterijum eliminacije: mikroservisi od starta otpadaju zbog DevOps složenosti za mali tim — kreće se od modularnog monolita. Servis se izdvaja iz monolita tek kad ima jasan razlog: nezavisno skaliranje, drugi jezik (npr. Go za inference-proxy), ili izolacija kvara.

## 5. HPC integracija (PARADOX, ITE)

- Protokol za SLURM: SSH + sbatch/squeue/sacct · SLURM REST API (slurmrestd) · Open OnDemand (komplementaran portal)
- Prenos fajlova: rsync preko SSH (veliki fajlovi) · scp/SFTP (mali fajlovi) · S3-kompatibilan API na HPC strani (MinIO)
- Kontejneri na HPC: Singularity/Apptainer, build na platformi → push u registry → pull na klasteru
- Bezbednost: SSH sertifikati (ne statički ključevi) preko Vault SSH engine-a · namenski servisni nalog po klasteru · mrežna izolacija bridge servisa
- Kvote: CPU/GPU sati po korisniku i projektu; provera pre slanja, rezervacija, obračun po stvarnoj potrošnji
- Rutiranje između klastera: po tipu resursa, lokaciji podataka, afilijaciji korisnika, dubini reda čekanja, ceni
- Kriterijum eliminacije: slurmrestd i S3/MinIO na HPC strani zavise od operatera klastera — ne računamo na njih u MVP-u, primarna je SSH+sbatch putanja jer ne zavisi ni od koga. Statički SSH ključevi otpadaju (samo Vault sertifikati).

## 6. ML runtime i inference

- Serviranje modela: vLLM · Triton Inference Server · HuggingFace TGI · TorchServe · ONNX Runtime · Ollama
- Fine-tuning stack: HuggingFace Transformers + PEFT/LoRA · Axolotl · Unsloth · LLaMA-Factory
- Strategija treninga: puni fine-tuning vs. parameter-efficient (LoRA/QLoRA) — odluka po veličini dataseta i budžetu
- Praćenje eksperimenata: MLflow · Weights & Biases · ClearML
- Evaluacija i benchmarking: lm-evaluation-harness · sektorski benchmark datasetovi · leaderboard sa procesom odobravanja
- Inference API: streaming tokena (SSE) · batching · upravljanje kontekstom chat sesija
- Kriterijum eliminacije: cloud-only experiment tracking (W&B cloud) otpada zbog izolacije i suvereniteta — prvo self-hosted MLflow, W&B samo ako MLflow podbaci. Serving alat koji ne degradira elegantno pod opterećenjem otpada.

## 7. Sloj podataka

- Relaciona baza: PostgreSQL (+ PgBouncer pooling); multi-tenancy: row-level security · schema-per-tenant · database-per-tenant
- Vektorska baza: pgvector · Qdrant · Weaviate · Milvus
- Objektno skladište: MinIO (S3-kompatibilno) — modeli, datasetovi, artefakti, staging za HPC
- Message queue: RabbitMQ · Apache Kafka · NATS JetStream · Redis Streams
- Keš: Redis (API keš, sesije, rate-limit brojači, stanje kvota); Sentinel vs. Cluster mod
- Verzionisanje datasetova: DVC · LakeFS
- Katalog datasetova: sopstveni katalog · CKAN · Dataverse · OpenMetadata; DCAT izvoz za EU portale
- Kriterijum eliminacije: zaseban vektorski servis (Qdrant/Weaviate/Milvus) uvodimo tek kad pgvector preraste potrebe; database-per-tenant otpada za MVP zbog operativnog tereta (kreće se od RLS). Za message queue biramo jedan, ne više njih "za svaki slučaj".

## 8. Identitet i kontrola pristupa

- Identity provider: Keycloak (OIDC, OAuth2, SAML2) · alternativе: Authentik, Zitadel
- Akademska federacija: eduGAIN preko AMRES-a (SAML brokering)
- EuroHPC identitet: Keycloak identity brokering ka EuroHPC AAI / MyAccessID
- RBAC uloge: platform-admin · data-admin · model-reviewer · ai-expert · advanced-user · basic-user · student · viewer · federation-partner
- ABAC atributi: sektorske sertifikacije (npr. zdravstvo) · nivo afilijacije (kvote) · GDPR saglasnosti · tenant
- API ključevi: generisanje, scope-ovanje, rotacija, opoziv
- Servis-servis autentikacija: kratkotrajni JWT (client credentials, token exchange) — bez statičkih tajni
- Kriterijum eliminacije: alternative Keycloak-u (Authentik, Zitadel) otpadaju ako nemaju zrelo SAML brokering ka eduGAIN i EuroHPC AAI — to je tvrd zahtev, ne opcija.

## 9. Infrastruktura i DevOps

- Kontejnerizacija: Docker · Podman (rootless)
- Orkestracija: k3s (pilot) · puni Kubernetes (produkcija) · Docker Compose (samo lokalni razvoj)
- Service mesh (opciono, kasnije): Istio · Linkerd
- CI/CD: GitLab CI · GitHub Actions; deploy: ArgoCD (GitOps)
- IaC: Terraform · Helm · Ansible
- Container registry: Harbor (self-hosted, skeniranje, potpisivanje) · DockerHub
- Observability: Prometheus + Grafana (metrike, alarmi) · Loki (logovi) · Tempo/Jaeger + OpenTelemetry (tracing) · Sentry (greške)
- Tajne: HashiCorp Vault — dinamički DB kredencijali, SSH sertifikati za HPC, Kubernetes auth, rotacija
- Kriterijum eliminacije: service mesh se ne uvodi u MVP/pilot (složenost > korist na ovoj skali); DockerHub otpada za interne artefakte zbog skeniranja/potpisivanja (Harbor self-hosted).

## 10. Federacija sa EuroHPC (Pharos, IT4LIA)

- Sinhronizacija kataloga: zakazana (cron) i event-driven; izvoz SAIFA modela/datasetova/benchmarka, uvoz eksternih
- Apstrakcija izvora: CatalogueSource interfejs po partneru (Pharos API, IT4LIA API, CKAN, Dataverse)
- Bezbednost: mTLS + kredencijali po partneru, validacija šeme pre svake razmene, rate limiting
- Plugin interfejsi za proširenja: ModelProvider (HuggingFace, OpenAI, Anthropic, Mistral, Groq, Replicate...) · ComputeBackend (PARADOX, ITE, Kubernetes, cloud) · CatalogueSource · sektorski pogledi kao konfiguracija
- Feature flags: LaunchDarkly · Unleash (self-hosted) · env varijable; ključni flagovi: ENABLE_HPC_SUBMISSION, ENABLE_FINE_TUNING, ENABLE_FEDERATION_SYNC, ENABLE_JUPYTER_HUB, ENABLE_TRAINING_PLATFORM
- Kriterijum eliminacije: LaunchDarkly (SaaS) otpada zbog self-hosting/suvereniteta — koristimo Unleash ili env varijable.

## 11. Bezbednost i usklađenost (presek kroz sve slojeve)

- Zero-trust model: svaki zahtev autentifikovan na svakom sloju, bez poverenja u internu mrežu
- Multi-tenant izolacija: PostgreSQL RLS politike + filtriranje u aplikaciji + ACL u pretrazi
- GDPR: klasifikacija podataka (4 nivoa) · evidencija saglasnosti · automatizovan DSAR izvoz · pravo na brisanje · DPIA procedura za osetljive datasetove
- Integritet artefakata: SHA-256 checksum · opciono GPG/cosign potpisivanje · provera pre serviranja
- Audit log: append-only, hash-chain, pokriva sve CRUD operacije, HPC poslove, inference pozive, federaciju; retencija min. 2 godine

## 12. Metapodaci i governance (centralni deo sistema, ne sporedni)

Za Gateway koji deli datasetove, modele, workflow-e i benchmark rezultate ka Pharos/IT4LIA, metapodaci su jezgro sistema. Bez doslednog modela metapodataka federacija i katalog ne mogu da rade.

- Standardi i šeme: DCAT (izvoz ka EU portalima otvorenih podataka) · schema.org (struktuirani podaci na vebu) · DataCite (metapodaci istraživačkih podataka, DOI)
- Kartice resursa: model card i dataset card (svrha, ograničenja, podaci za trening, poznati rizici) — obavezne pre objave
- Lineage i provenance: odakle resurs dolazi i kroz koje transformacije je prošao (bazni model → fine-tune → verzija; izvorni dataset → obrada → verzija)
- Obavezni metapodaci po resursu: owner · license · visibility (public/private/team) · domain · language · provenance/izvor · version · quality status · validation status · GDPR klasa · access policy · publication status
- Kompatibilnost licenci: provera pre objave (da li licenca dataseta dozvoljava objavu izvedenog modela)
- Quality score i PII klasifikacija: automatska oznaka kvaliteta i nivoa osetljivosti, vidljiva u katalogu
- Source of truth: lokalni SAIFA registar je merodavan za lokalne resurse; federisani resursi se ne mešaju u istu bazu bez oznake porekla (vidi blueprint ispod)

## 13. Arhitekturni blueprint (tekstualni — ko sa kim komunicira)

Bez dijagrama, ali sa jasnim tokom i source-of-truth tačkama:

- Korisnik → Frontend → API Gateway (Keycloak JWT validacija ovde) → Backend servisi.
- Policy decisions (RBAC/ABAC) donose se u backend-u pri svakoj operaciji, oslonjene na Keycloak uloge/atribute; pretraga dodatno filtrira po ACL-u (nikad ne prikazuje neodobren resurs).
- Source of truth za metapodatke: PostgreSQL registar (modeli, datasetovi, katalog). Težine i artefakti: MinIO. Identitet: Keycloak. Tajne: Vault. Nijedan drugi servis ne drži paralelnu "istinu".
- HPC posao: backend (job-scheduler) → hpc-bridge → SSH/sbatch ka PARADOX/ITE; status nazad preko sacct; rezultati u MinIO.
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
- Semantička pretraga staje u pgvector → pgvector. Preraste obim/performanse → zaseban vektorski servis (Qdrant/Weaviate).

## 16. Licence i nabavka (presek)

Pošto se insistira na suverenom/self-hosted okruženju, treba razlikovati slične alate po modelu licenciranja i hostinga:

- Feature flags: LaunchDarkly (SaaS, plaća se) ≠ Unleash (self-hosted, open-source) — biramo Unleash/env varijable.
- Experiment tracking: W&B cloud (SaaS) ≠ MLflow (self-hosted) — biramo MLflow.
- API gateway: AWS API Gateway (managed cloud) nije prirodan izbor za self-hosted — koristimo Kong/Traefik/NGINX.
- Eksterni ModelProvider-i (OpenAI, Anthropic, Mistral, Groq, Replicate) su plative eksterne usluge — koriste se kroz proxy uz token budžet, nisu zamena za lokalne modele.

---

Sledeći korak: za svaki sloj testirati navedene opcije (počev od default PoC stacka), mapirati zahteve klijenata na mogućnosti, i dokumentovati odluku sa obrazloženjem (reuse-vs-build, trošak, operativna složenost), uz kriterijum eliminacije zašto opcija nije izabrana.
