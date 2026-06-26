# SAIFA AI Gateway — gotove komponente po slojevima

## 1. Svrha

SAIFA AI Gateway nije skup funkcionalnosti koje se pišu od nule, već **integraciona platforma** koja objedinjuje proverene open-source komponente iza jednog portala. Korisnik vidi jedan portal, jedan identitet, jedinstven katalog, jedinstvene kvote i istoriju aktivnosti, dok ispod rade specijalizovane platforme.

> **Reuse before build. Integrate before replace. Custom samo tamo gde postoji SAIFA-specifična vrednost.**

Ovaj dokument je sažeta odluka „gotovo ili custom" po sloju. Detaljna evaluacija opcija i kriterijumi eliminacije su u `popis.md`, „definition of done" i faze po funkcionalnosti u `funkcionalnosti.md`, a scenariji u `SAIFA_use_cases_final.md`.

## 2. Gotovo vs. custom po sloju

| Sloj / potreba | Gotova komponenta (prvi izbor; alternative) | Šta SAIFA razvija |
|---|---|---|
| Portal i UX | Next.js/React (Nuxt/Vue) | SAIFA UI, BFF, povezivanje alata |
| Identitet i SSO | Keycloak (authentik, WSO2 IS) | mapiranje uloga, ABAC, povezivanje naloga |
| Politike pristupa | Open Policy Agent (Casbin, Cedar) | SAIFA RBAC/ABAC model |
| Kursevi (LMS) | Moodle (Open edX) | veza sa portalom, kvotama i lab vežbama |
| Sveske i IDE | JupyterHub + KubeSpawner (SlurmSpawner, code-server, JupyterLite) | pokretanje iz portala, kvote, dataset mount, audit |
| Dataset katalog | CKAN ili Dataverse (DataHub, OpenMetadata) | objedinjeni katalog, ACL, federacioni adapteri |
| Pretraga | OpenSearch + pgvector (PostgreSQL FTS, Qdrant) | zajednički indeks i ACL filtriranje |
| Model registry i eksperimenti | MLflow (ClearML, Aim, W&B) | model cards, approval, katalog, prava pristupa |
| LLM inference | vLLM (TGI, SGLang, Triton) | API proxy, kvote, routing, audit |
| Model serving orkestracija | KServe (Ray Serve, BentoML) | serving profili i deployment pravila |
| Fine-tuning i trening | Hugging Face TRL/PEFT (Axolotl, LLaMA-Factory) | odobreni šabloni, HPC submit, registracija rezultata |
| HPC poslovi | SLURM na postojećim klasterima; Open OnDemand kao dopuna | HPC bridge i jedinstveni job model |
| Kubernetes / batch | Kubernetes Jobs (Volcano, Kueue) | routing između K8s i SLURM-a |
| Workflow-i | Argo Workflows (Airflow, Flyte, Kubeflow) | jednostavni SAIFA workflow šabloni |
| Kontejneri | Harbor + Trivy + BuildKit/Buildah; Apptainer na HPC (Quay, Kaniko) | odobravanje image-a i HPC konverzija |
| Skladište | PostgreSQL (source of truth), S3 (MinIO/Ceph) za objekte, Valkey/Redis za keš | SAIFA domeni, metadata veze, lifecycle |
| Observability | Prometheus + Grafana + Loki + OpenTelemetry | SAIFA dashboardi i korelacija sa korisnikom/job-om |
| Audit | append-only (PostgreSQL/OpenSearch) + WORM arhiva (immudb) | zajednički audit događaji i policy |
| Tajne | HashiCorp Vault / OpenBao | politike pristupa, rotacija, HPC SSH sertifikati |
| Notifikacije i status | Novu/Gotify + email; Uptime Kuma za status | događaji, preference, javni SAIFA status |
| Ticketing | Zammad ili GLPI (GitLab Issues za PoC) | dugme u portalu i kontekst iz audit loga |
| Kolaboracija i kod | GitLab CE (Gitea/Forgejo) + Nextcloud | prava nad SAIFA resursima i veza repo↔workflow |
| Benchmarking i kvalitet | lm-evaluation-harness + Great Expectations | SAIFA benchmark definicije, leaderboard, pravila |
| Lineage | OpenLineage + Marquez (DataHub) | povezivanje job–dataset–model–rezultat |
| Federacija | DCAT/DCAT-AP + custom adapteri (CKAN Harvester) | Pharos/IT4LIA adapteri i mapiranje šema |
| Kvota i accounting | sopstveni jednostavni model (Waldur) | portal, rezervacije, objedinjeni obračun po fabrici |
| BI i izveštaji | Grafana → Superset/Metabase | SAIFA metrike i dozvole |
| CI/CD i GitOps | GitLab CI + Argo CD (Flux, Tekton) | SAIFA pipeline-i i konfiguracija klastera |
| Backup i DR | Velero + PostgreSQL backup + replikacija | RPO/RTO politike i restore testovi |

## 3. Šta se ne razvija od nule

LMS, notebook/kernel sistem, identity provider, Git server, container registry, monitoring i dashboard engine, general-purpose workflow engine, training framework, LLM inference runtime, object storage, full-text pretraga, helpdesk, editor dokumenata, vulnerability scanner, secrets manager, Kubernetes/HPC scheduler. Sve to već postoji zrelo; razvoj bi potrošio vreme bez SAIFA-specifične vrednosti.

## 4. SAIFA-specifično jezgro

Razvija se samo ono što nosi vrednost Gateway-a: jedinstveni **portal i UX**, **kanonski metadata model** i **objedinjeni katalog**, **RBAC/ABAC** i organizacioni model, **kvote i objedinjeni usage prikaz**, **HPC/Kubernetes routing** i **HPC bridge**, **inference proxy**, **review/approval** tokovi, **audit događaji**, **federacioni adapteri**, **lineage** (dataset–job–model–rezultat), SAIFA **benchmark i fine-tuning šabloni**, te jedinstven **status, notifikacije i podrška** iz portala.

## 5. Preporučeni PoC stack

Portal Next.js/React · IAM Keycloak · politike OPA ili backend modul · backend modularni monolit (FastAPI/Spring/NestJS) · PostgreSQL · S3 (MinIO/Ceph) · pretraga PostgreSQL FTS pa OpenSearch · keš Valkey · Moodle · JupyterHub + KubeSpawner (SlurmSpawner kao zaseban PoC) · CKAN/Dataverse · MLflow · vLLM (KServe/Triton tek kad zatreba) · TRL+PEFT kroz odobren šablon · Argo Workflows · HPC preko SSH + `sbatch/squeue/sacct` · Harbor + Trivy + Apptainer · Prometheus/Grafana/Loki · Vault/OpenBao · GitLab CE · ticketing GitLab Issues (Zammad kasnije) · federacija read-only adapter sa mock partnerom.

Za svaku gotovu platformu PoC mora da dokaže šest tačaka: ko je **source of truth**, kako se korisnik **prijavljuje**, kako se **prenose uloge**, ko **proverava pristup**, gde se vodi **kvota**, kako se beleži **audit** i kako se komponenta prikazuje kroz **jedinstveni SAIFA UX**.

## 6. Konačna preporuka

SAIFA Gateway je **kontrolni i integracioni sloj**, ne monolit koji reimplementira alate. Najrazumnija početna kombinacija: **Keycloak** (identitet), **Moodle** (kursevi), **JupyterHub** (sveske), **CKAN/Dataverse** (datasetovi), **MLflow** (modeli i eksperimenti), **vLLM** (LLM inference), **SLURM** (HPC), **Argo Workflows** (Kubernetes workflow-i), **Harbor + Apptainer** (kontejneri), **PostgreSQL + S3 + OpenSearch** (podaci i katalog), **Prometheus/Grafana/Loki** (observability), **Vault/OpenBao** (tajne), uz **custom SAIFA portal, orchestration, policy, audit i federation adaptere** kao jezgro projekta.

Redosled uvođenja: (1) osnovne platforme — Keycloak, portal, PostgreSQL/S3, Moodle, JupyterHub, MLflow, vLLM, HPC bridge, monitoring; (2) katalog i lifecycle — CKAN/Dataverse, OpenSearch, Harbor, review/approval, lineage, kvote, ticketing; (3) napredna orkestracija — Argo, KServe/Triton, fine-tuning šabloni, benchmark, timski prostori, BI; (4) federacija i production hardening — Pharos/IT4LIA adapteri, token exchange, metadata sync, HA, DR, supply-chain politike, SLO/SLA.
