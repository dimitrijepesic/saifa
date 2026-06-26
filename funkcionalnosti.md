# SAIFA AI Gateway — Funkcionalnosti, opcije i plan testiranja

Ovaj dokument polazi od funkcionalnosti platforme (šta korisnik radi) i za svaku popisuje opcije realizacije, eliminacione uslove koje opcija mora da ispuni, i plan kako ćemo proveriti koja opcija je najbolja za naš slučaj korišćenja. Zajedno sa `popis_final.md` (popis tehnologija po slojevima infrastrukture) čini osnovu za izbor tehnologija: `popis_final.md` odgovara na pitanje "šta sve postoji", a ovaj dokument na pitanja "čemu to služi kod nas", "šta je eliminacioni uslov" i "kako ćemo proveriti šta nam odgovara".

## Mesto ovog dokumenta u celini

SAIFA dokumentacija je povezan skup u kome svaki dokument odgovara na jedno pitanje; ovaj dokument se oslanja na ostale i njih hrani:

| Dokument | Pitanje na koje odgovara | Veza sa ovim dokumentom |
| --- | --- | --- |
| `popis_final.md` | „Šta sve postoji" — tehnički popis opcija po slojevima | ovaj dokument bira *čemu* te opcije služe i *kako* ćemo ih testirati |
| **`funkcionalnosti.md` (ovaj)** | „Čemu to služi kod nas, kako biramo, šta je eliminacioni uslov" | merodavan izvor **[eliminacionih]** kriterijuma i „definition of done"-a |
| `kriterijumi_izbora_tehnologija.md` | „Prema čemu se meri kandidat po domenu" (test-pitanja) | preuzima eliminacione uslove odavde i razrađuje ih u domenske rubrike |
| `SAIFA_use_cases_final.md` | „Korak po korak, šta korisnik tačno radi" (10 sekcija UC-ova) | svaka funkcionalnost ispod mapira se na sekciju use case-ova |
| `mvp_kriterijumi.md` | „Šta ulazi u MVP" (dimenzije K1–K5, dodela faze) | merodavan je za fazu; tabela prioriteta ispod je ulaz, ne konačna reč |
| `otvorene_odluke.md` | „Šta još nije odlučeno" (OD-1…OD-12) | funkcionalnosti ispod referenciraju OD koji ih blokira |
| `upitnik-zahtevi.md` | „Šta klijent zaista traži" | određuje redosled testiranja i konačni MoSCoW prioritet |

Da bi se izbegao utisak da sve treba raditi odmah: svaka funkcionalnost je svrstana u fazu isporuke (MVP / Pilot / Production / Later) u tabeli prioriteta ispod, ima jasan "definition of done" i popisane rizike. **Konačnu fazu po scenariju ne određuje ovaj dokument nego `mvp_kriterijumi.md`** (primenom K1–K5 i blokada iz `otvorene_odluke.md`); tabela ispod je ciljana faza i ulaz u tu analizu. Konačni redosled i obim određuju odgovori klijenata iz `upitnik-zahtevi.md`.

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

Funkcionalnosti dobijaju smisao kroz konkretne korisnike. Svaki scenario je ujedno i test usvajanja — ako persona ne prođe svoj put bez pomoći, funkcionalnost nije gotova. Korak-po-korak razrada svakog puta je u `SAIFA_use_cases_final.md` (sekcije navedene uz svaku funkcionalnost ispod).

- **Istraživač** želi da fine-tune-uje model na svom datasetu: prijava → nađe bazni model i dataset → pokrene fine-tuning iz šablona na HPC → prati napredak → novi model se registruje. (funkc. 1, 3, 4, 5, 11)
- **Lekar / sektorski stručnjak** želi sandbox nad kontrolisanim zdravstvenim podacima: prijava → otvori sandbox sa read-only pristupom → radi obradu koja ne iznosi podatke van klastera (compute-to-data). (funkc. 4, 5, 6, 11, 12)
- **Student** radi lab vežbu: upis na kurs → otvori svesku → pokrene mali posao na klasteru kroz zaključan šablon → automatska provera → upisan napredak. (funkc. 6, 8, 5)
- **SME / startup** traži gotov inference API: prijava → nađe model u katalogu → dobije OpenAI-kompatibilan endpoint sa token budžetom. (funkc. 2, 7, 11)
- **Analitičar iz javnog sektora** traži dataset i izveštaj: pretraga kataloga → preuzimanje ili API pristup → potrošnja vidljiva u monitoringu. (funkc. 7, 4, 12)
- **EuroHPC partner** uvozi/izvozi resurse: federisani katalog sa oznakom porekla, sync uz odobravanje objave. (funkc. 9, 13)

## Matrica prijave (ko šta koristi)

Svaki tip korisnika ima preporučen metod prijave koji određuje šta sistem automatski preuzima (afilijacija, kvota, federisani pristup). Svi putevi vode do iste Keycloak sesije.

| Tip korisnika | Preporučen metod prijave | Šta se automatski konfiguriše |
| --- | --- | --- |
| Student (akademija) | eduGAIN/AMRES SSO | Nalog iz institucijskog IdP-a; afilijacija preneta; kvota iz pool-a institucije |
| Istraživač (akademija) | eduGAIN/AMRES SSO | Isto kao student; sektorski ABAC atribut dodeljuje admin posle verifikacije |
| SME / startup | Lokalni nalog | Bez IdP-a; kvota iz platformskog pool-a dok organizacija ne bude odobrena |
| EuroHPC korisnik | MyAccessID | Federisani resursi vidljivi odmah; brokering ka Pharos/IT4LIA bez posebnog naloga |
| Zdravstveni / sektorski stručnjak | Lokalni nalog ili SSO + ABAC | Admin ručno dodeljuje sektorski atribut (zdravstvo, javni sektor) posle verifikacije |
| Platform administrator | Lokalni nalog | MFA preporučen; sav pristup kroz admin panel |
| API / programski pristup | API ključ | Izdat kroz portal; scope-ovan, rotirajući, opoziv sa trenutnim dejstvom |

## Zajednički scenariji (visok prioritet — važe za sve persone)

Sledeće funkcionalnosti su preduslov za rad svake persone i imaju automatski visok prioritet bez obzira na klijentski upitnik: prijava na platformu · pretraga kataloga · pregled detalja resursa · praćenje sopstvene kvote i potrošnje · prijem notifikacija o ishodu poslova · pregled lične istorije aktivnosti · prijava problema (ticketing). Sve ovo je MVP za svakog korisnika — ako ovo ne radi, platforma nema vrednost ni za jednu personu.

## Mapiranje: funkcionalnost → use case → domen kriterijuma

Da bi celina bila proverljiva, svaka funkcionalnost se vezuje za sekciju use case-ova (detaljan tok u `SAIFA_use_cases_final.md`) i za domen iz `kriterijumi_izbora_tehnologija.md` (rubrika za izbor tehnologije). Prazna ćelija u koloni domena znači da se izbor rešava kroz decision tree (`popis_final.md` §15), bez zasebne rubrike.

| # | Funkcionalnost | Use case sekcija | Domen kriterijuma | Otvorene odluke |
| --- | --- | --- | --- | --- |
| 1 | Registracija i dodavanje modela | 5 Model lifecycle | 5 Storage | OD-10 |
| 2 | Korišćenje modela (inference / chat) | 3 Inference | 3 Inference serving | OD-3, OD-5, OD-6, OD-7, OD-8 |
| 3 | Fine-tuning modela | 5 Model lifecycle | 6 Experiment tracking | OD-4, OD-9 |
| 4 | Upravljanje datasetovima | 6 Datasetovi | 5 Storage | OD-8 |
| 5 | Pokretanje poslova na HPC | 4 AI factory | 4 HPC orkestracija | OD-4, OD-5 |
| 6 | Interaktivni rad u sveskama | 8 Edukacija (i deo 4) | 7 Edukacija | — |
| 7 | Pretraga i katalog resursa | 2 Katalog | 2 Katalog resursa | OD-8 |
| 8 | Obuke i edukacija (LMS) | 8 Edukacija | 7 Edukacija | — |
| 9 | Federacija sa EuroHPC | 10 Federacija | 8 Federacija | — |
| 10 | Benchmarking i evaluacija | 5 Model lifecycle | 6 Experiment tracking | OD-12 |
| 11 | Upravljanje korisnicima i pristupom | 1 Pristup i nalozi | 1 Autentikacija | OD-5, OD-8 |
| 12 | Administracija, nadzor i usklađenost | 9 Administracija | presek (`popis_final.md` §11) | — |
| 13 | Kolaboracija i timski rad | 7 Kolaboracija | — | OD-1, OD-2 |
| 14 | Workflow orchestration | presek (sek. 5) | — | — |
| 15 | Ticketing i korisnička podrška | presek (sek. 9) | — | — |
| 16 | Upravljanje kontejnerima | sek. 4, sek. 2 (katalog) | — | — |
| 17 | Status platforme | sek. 9 (monitoring) | — | — |
| 18 | Lični dashboard | sek. 9 (lična istorija/potrošnja) | — | — |
| 19 | Admin statistike | sek. 9 (izveštavanje) | — | — |

## Tabela prioriteta (faze isporuke)

"basic" = osnovna verzija dovoljna da scenario prođe; prazno polje = nije u toj fazi. Ovo je **ciljana** faza; `mvp_kriterijumi.md` primenom K1–K5 može da je pomeri — posebno: funkcionalnost koju ovde ciljamo za MVP, a koju blokira otvorena odluka, ostaje **Pilot** dok se odluka ne zatvori (kolona „Blokira").

| Funkcionalnost | MVP | Pilot | Production | Later | Blokira (OD) |
| --- | --- | --- | --- | --- | --- |
| 11 Login / uloge | da | da | da | | OD-8 (ABAC vidljivost) |
| 7 Katalog i pretraga | full-text | + semantička | da | | OD-8 (ABAC-striktna vidljivost) |
| 2 Inference / chat | basic (1 serving) | + sektorski chat | da | | OD-3, OD-5, OD-8 |
| 1 Registracija modela | upload + HF import | + eksterni API, lineage | da | | OD-10 (nepotpun lineage) |
| 5 HPC submit | basic (SSH+sbatch) | + šabloni, kvote | da | | OD-4, OD-5 |
| 4 Dataset upload/verzionisanje | delimično | da | da | | OD-8 |
| 6 Sveske | JupyterHub (KubeSpawner) | + SlurmSpawner | da | | |
| 3 Fine-tuning | basic LoRA šablon | da | da | | OD-4, OD-9 |
| 12 Monitoring / audit | basic | da | da | | |
| 13 Kolaboracija | team vidljivost | + review/approval | da | | OD-1 |
| 10 Benchmarking | basic harness | + leaderboard | da | | OD-12 (leaderboard) |
| 8 LMS | biblioteka materijala | LMS pilot | da | | |
| 9 Federacija | mock / read-only | pilot (sandbox) | da | | |
| 14 Workflow orchestration | basic (linearni lanac) | + grananje/retry (Argo Workflows PoC) | da | generički engine | |
| 15 Ticketing / podrška | MVP forma (email/GitLab issue) | + interni helpdesk | da | | |
| 16 Kontejneri | build + push u Harbor | + zaključani šabloni | da | | |
| 17 Status platforme | status stranica (korisnik) | + health dashboard (admin) | da | | |
| 18 Lični dashboard | usage i istorija aktivnosti | + trendovi, poređenje perioda | da | | |
| 19 Admin statistike | poslovne metrike platforme | + izvozivi izveštaji | da | | |
| Self-service deploj endpointa | | | | da | OD-6, OD-7 |

## Kako ceo pipeline izgleda (kontekst za sve funkcionalnosti)

Bez obzira na to koju funkcionalnost korisnik koristi, put kroz sistem je uvek isti i sve opcije ispod se uklapaju u njega (arhitekturni blueprint: `popis_final.md` §13):

1. Korisnik se prijavljuje (lokalni nalog, univerzitetski nalog ili EuroHPC identitet) i dobija pogled prilagođen svojoj ulozi.
2. U katalogu pronalazi resurs: model, dataset, kurs, svesku ili workflow — lokalni ili federisani (Pharos, IT4LIA).
3. Pokreće akciju: postavlja pitanje modelu, pokreće fine-tuning, šalje posao na superračunar, otvara svesku ili lab vežbu.
4. Platforma posao izvršava tamo gde mu je mesto: na Kubernetes GPU pool-u (brze, interaktivne stvari) ili na PARADOX/ITE klasteru (veliki poslovi), uz proveru kvota. Kratkotrajni **poslovi** (batch, terminalno stanje) i dugotrajni **deployments** (inference endpoint) vode se kroz zasebne entitete sa zasebnim akcijama i stanjima (`popis_final.md` §4, §13).
5. Rezultat se vraća korisniku: odgovor u chatu, novi model u registru, fajlovi u skladištu, položena lab vežba — uz notifikaciju.
6. Sve vreme ispod radi nadzor, merenje potrošnje i audit — svaki korak je zabeležen u append-only audit log (hash-chain).

## Metodologija: kako ćemo birati između opcija

Plan nam je da ne biramo tehnologije na osnovu utisaka i popularnosti, nego da svaku ozbiljnu odluku donesemo posle malog, vremenski ograničenog testa (proof-of-concept od nekoliko dana, u dev okruženju — Docker Compose ili mali k3s klaster), polazeći od podrazumevanog PoC stacka iz `popis_final.md` §14. Svaki test radimo ovako:

1. Pre testa zapišemo šta tačno merimo i šta je kriterijum uspeha (da posle ne pomeramo golove). Za domen koji ima rubriku u `kriterijumi_izbora_tehnologija.md`, polazimo od tamošnjih test-pitanja; **kandidat koji padne na [eliminacionom] kriterijumu ispada iz izbora bez obzira na ostalo.**
2. Testiramo opcije na istom zadatku, sa istim podacima/modelom, da poređenje bude pošteno.
3. Ocenjujemo svaku preostalu opciju (1–5) po šest kriterijuma:
   - iskustvo korisnika — posebno za ne-tehničke korisnike, jer su oni najveći rizik za usvajanje platforme;
   - trud integracije — koliko koda i konfiguracije treba da se uklopi u našu arhitekturu;
   - operativna složenost — da li to mali tim može da održava godinama;
   - performanse i trošak — latencija, propusnost, GPU sati, hardverski zahtevi;
   - zrelost i zajednica — dokumentacija, učestalost release-ova, koliko je lako naći pomoć;
   - usklađenost — GDPR, suverenitet podataka (self-hosted), licence.
4. Odluku zapišemo kao kratak zapisnik (decision record): šta smo testirali, rezultati, šta smo izabrali i zašto. Tako svaka odluka ostaje proverljiva i može da se preispita kad se okolnosti promene. Čisto arhitektonske odluke (ne izbor proizvoda) vode se kao OD u `otvorene_odluke.md`.
5. Gde je moguće, u test uključujemo prave korisnike: za korisničke interfejse barem 3–5 osoba iz ciljne grupe (npr. neko iz zdravstva za sektorski chat), jer naše mišljenje o "jednostavnosti" ne vredi mnogo.

Redosled testiranja pratiće prioritete iz upitnika za klijente (`upitnik-zahtevi.md`): prvo testiramo opcije za funkcionalnosti koje klijenti zaista traže, a egzotičnije opcije ostavljamo za kasnije ili preskačemo.

## 1. Registracija i dodavanje modela

**Use case:** sek. 5 (Model lifecycle) · **Domen:** 5 Storage · **Eliminacioni:** S1 (object store iza platformskog API-ja, bez `git`/LFS), S2 (multipart upload + resumable download), S7 (provera integriteta pre serviranja) · **Otvoreno:** OD-10 (lineage politika).

Opcije:
- Načini dodavanja: import sa HuggingFace Hub-a (po ID-u) · upload sopstvenog checkpoint-a · registracija eksternog API modela (OpenAI, Anthropic, Mistral, Groq, Replicate) · programski kroz svesku/CLI (SDK / REST API)
- Skladištenje težina: MinIO (S3-kompatibilno), resumable multipart upload, SHA-256 provera integriteta
- Metapodaci: registar u PostgreSQL-u (licenca, jezik, tip zadatka, lineage), verzionisanje, vidljivost public/private/team

Kako testiramo: prođemo ceo put registracije za mali model (~1 GB), veliki (7B, ~15 GB) i jedan eksterni API model, uz namerni prekid prenosa i proveru da checksum zaustavlja oštećen fajl. Kriterijum uspeha: ekspert registruje model bez čitanja dokumentacije, a prekinut prenos se sam oporavlja.

Definition of done: korisnik registruje model (upload / HF / eksterni API), dobije verziju i obavezne metapodatke (uklj. licencu), model je vidljiv u katalogu po zadatoj vidljivosti, a prekinut prenos se sam oporavlja.

Rizici: tehnički (veliki fajlovi, integritet) — nizak · compliance (provera licence modela pre objave) — srednji · zavisnost od partnera — niska.

## 2. Korišćenje modela (inference / chat)

**Use case:** sek. 3 (Inference) · **Domen:** 3 Inference serving · **Eliminacioni:** I1 (serving runtime iza gateway-a — korisnik ga nikad ne zove direktno), I4 (kvota i rate limit se troše pre izvršenja) · **Otvoreno:** OD-3 (serving model: deljeni pool vs. self-service), OD-5 (kvotni model), OD-8 (ABAC vidljivost); self-service deploj je OD-6/OD-7 (Later).

Domen je **suštinski otvoren** po pitanju serving modela (OD-3) — opcije ispod ne pretpostavljaju ishod te odluke.

Opcije:
- Interfejsi: chat UI (sektorski prilagođen) · REST API (OpenAI-kompatibilan) · Anthropic-kompatibilan API i spoljni agentski/coding klijenti (Claude Code, Cursor, Copilot) preko gateway-a · poziv iz sveske (`upitnik-zahtevi.md` B8)
- Serving apstrakcija (K8s sloj iznad runtime-a): KServe (`InferenceService` CRD) · Ray Serve · goli Helm/K8s manifest
- Serving runtime: vLLM · Triton Inference Server · HuggingFace TGI · ONNX Runtime (laki CPU modeli) · eksterni provajderi kroz proxy
- Serving topologija je metapodatak modela (single-GPU / single-node multi-GPU / multi-node), promenljiva bez izmene klijenta (`popis_final.md` §6)
- Prateće: SSE streaming · rate limiting i token budžeti · keširanje (Valkey/Redis) · merenje potrošnje sa atribucijom po korisniku

Kako testiramo: isti 7B model podignemo na vLLM i TGI na istom GPU-u i pustimo load test (1/10/50 korisnika), mereći vreme do prvog tokena, p95 latenciju, propusnost i ponašanje pod preopterećenjem; chat UI testiramo sa ne-tehničkim korisnicima. Kriterijum: stabilan streaming pod opterećenjem i chat koji ne-tehnički korisnik koristi bez pomoći.

Definition of done: model se servira preko OpenAI-kompatibilnog API-ja i chat UI-ja, sa streamingom, rate limitom, token budžetom i merenjem potrošnje; sav inference saobraćaj ide kroz gateway (runtime nedostupan direktno); ne-tehnički korisnik dobije odgovor bez pomoći. Ako klijent traži spoljne agentske klijente, gateway izlaže i provider-kompatibilne facade (`/v1/chat/completions`, `/v1/messages`).

Rizici: tehnički (GPU kapacitet, latencija pod opterećenjem) — srednji · operativni (održavanje serving stacka) — srednji.

## 3. Fine-tuning modela

**Use case:** sek. 5 (Model lifecycle) · **Domen:** 6 Experiment tracking · **Eliminacioni:** E1 (beleženje metrika iz posla na klasteru preko delimično izolovane veze) · **Otvoreno:** OD-4 (prag rutiranja K8s ↔ SLURM), OD-9 (ponašanje trackinga pri prekidu veze).

Opcije:
- Pokretanje: vizard sa hiperparametrima · YAML šabloni · custom skripta iz sveske · API
- Stack: HuggingFace Transformers + PEFT (LoRA/QLoRA) · Axolotl · Unsloth · LLaMA-Factory
- Izvršavanje: SLURM job na PARADOX/ITE · Kubernetes Job za manje poslove
- Praćenje: MLflow · Weights & Biases · ClearML

Kako testiramo: isti bazni model i mali srpski instruction set provučemo kroz Axolotl, Unsloth i LLaMA-Factory, mereći trud konfiguracije, VRAM, trajanje i kvalitet pre/posle, i poredimo LoRA vs QLoRA. MLflow proveravamo preko veze ka klasteru (eliminacioni kriterijum); W&B samo ako MLflow podbaci. Kriterijum: ceo ciklus (config → trening → eval → registracija) prolazi bez ručnih intervencija.

Definition of done: korisnik pokrene fine-tuning iz šablona, posao ode na HPC/Kubernetes, prati napredak uživo, a rezultujući model se automatski registruje sa lineage-om ka baznom modelu i datasetu.

Rizici: tehnički (GPU sati, stabilnost treninga) — srednji · organizacioni (ko odobrava trošak GPU sati) — srednji.

## 4. Upravljanje datasetovima

**Use case:** sek. 6 (Datasetovi) · **Domen:** 5 Storage · **Eliminacioni:** S1 (upload/preuzimanje bez `git`/LFS — lekar ili agronom bez ijednog CLI koraka), S2 (multipart + resumable), S5 (nepromenljive ranije verzije) · **Otvoreno:** OD-8 (ABAC vidljivost osetljivih datasetova).

Opcije:
- Upload: portal (presigned multipart u MinIO) · SDK/CLI · povezivanje izvora (Nacionalni data centar Kragujevac)
- Verzionisanje: DVC · LakeFS
- Validacija: provera formata, enkodiranja, heuristički PII sken
- GDPR: klasifikacija (4 nivoa) · saglasnosti · anonimizacija · lineage

Kako testiramo: za DVC i LakeFS odigramo upload → nova verzija → vraćanje na staru (i paralelnu anotaciju gde se razlikuju), uz uslov da korisnik ne vidi nijednu komandu alata. PII sken merimo na probnom datasetu sa namerno ubačenim ličnim podacima. Kriterijum: verzionisanje koje korisnik razume kao "verzija 1, verzija 2" bez ikakvog znanja o alatu ispod.

Definition of done: korisnik upload-uje dataset, dobije verziju, vrati se na staru, a sistem klasifikuje podatke i prijavi mogući PII pre nego što dataset može da se objavi ili podeli.

Rizici: compliance (PII, GDPR, osetljivi/zdravstveni podaci) — visok · tehnički (veliki datasetovi, anotacije) — srednji · organizacioni (ko je vlasnik dataseta) — srednji.

## 5. Pokretanje poslova na HPC (PARADOX, ITE)

**Use case:** sek. 4 (AI factory i pokretanje poslova) · **Domen:** 4 HPC orkestracija · **Eliminacioni:** H1 (rad preko SSH+sbatch/squeue/sacct, bez slurmrestd-a), H2 (apstrakcija SLURM-a od korisnika — nijedna linija SLURM sintakse), H4 (provera i rezervacija kvote pre zauzeća čvora) · **Otvoreno:** OD-4 (prag rutiranja), OD-5 (kvotni model).

Rutiranje Kubernetes ↔ SLURM je **suštinski otvoreno** (OD-4) — opcije mere sposobnost rutiranja, ne fiksiraju prag.

Opcije:
- Interfejsi: forma u portalu · sveska sa SDK · raw API · zaključani šabloni za studente
- Konekcija: SSH + sbatch/squeue/sacct · slurmrestd REST API · Open OnDemand (komplementarno)
- Prenos: rsync (veliki fajlovi) · SFTP (skripte) · S3/MinIO na HPC strani (staging)
- Kontejneri na klasteru: Singularity/Apptainer (build na platformi → push → pull; vidi funkc. 16)
- Bezbednost: Vault SSH sertifikati · izolovan bridge servis · per-resource secret za pisanje logova/statusa posla (`popis_final.md` §4)
- Kvote: dvonivovski model (pool organizacije → lična kvota člana) sa rezervacijom i obračunom (`popis_final.md` §8a; vidi funkc. 11), prag/model su OD-5

Kako testiramo: razvijamo prvo protiv lokalnog mock SLURM-a, pa na pravom klasteru, sa fokusom na otpornost: prekid SSH veze između sbatch-a i čitanja job ID-a (dupli posao?), restart bridge servisa dok poslovi traju (ponovno usvajanje preko sacct?) i prekid rsync prenosa (nastavak?). slurmrestd testiramo samo ako ga operateri omoguće — SSH putanja je primarna. Kriterijum: nijedan prekid ne sme da proizvede dupli posao ili izgubljen rezultat.

Definition of done: korisnik pokrene job iz portala (poslovi su zaseban entitet od deployments — `popis_final.md` §4), vidi status i logove uživo, preuzme rezultat, a sistem sprečava dupli submit posle prekida veze i oporavlja stanje poslova posle restarta bridge servisa. Prag rutiranja i model kvote ostaju otvoreni (OD-4, OD-5) dok ih obim iz `upitnik-zahtevi.md` C2 ne potvrdi.

Rizici: zavisnost od operatera klastera (slurmrestd, S3/MinIO, kvote nisu zagarantovani) — visoka · tehnički (otpornost na prekide veze) — srednji · bezbednosni (pristup klasteru) — srednji.

## 6. Interaktivni rad u sveskama (eksperti i studenti)

**Use case:** sek. 8 (Edukacija) i deo sek. 4 · **Domen:** 7 Edukacija · **Eliminacioni:** D1 (izolovana sesija po korisniku sa CPU/RAM/GPU limitima i trajanjem), D2 (idle culling — neaktivne GPU sveske se same gase).

Opcije:
- JupyterHub (SSO preko Keycloak-a): KubeSpawner · SlurmSpawner
- Alternative: JupyterLite (browser, bez servera) · code-server (pun IDE) · Marimo (edukativne sveske) · Google Colab (samo javni edukativni sadržaj)
- Politike: limiti po korisniku, gašenje neaktivnih sesija, GPU po ulozi

Kako testiramo: merimo vreme od „Otvori svesku" do prve ćelije za KubeSpawner (sekunde) i SlurmSpawner (minute) — to bira podrazumevani, i simuliramo pun klaster radi poruke korisniku. Istu lab vežbu damo u JupyterLite i JupyterHub-u, code-server probnim naprednim korisnicima, i testiramo gašenje neaktivnih GPU sesija (rad se ne sme izgubiti). Kriterijum: sveska se otvara dovoljno brzo da ne prekida tok rada, a neaktivne GPU sesije se oslobađaju bez gubitka rada.

Definition of done: sveska se otvara dovoljno brzo da ne prekida tok rada, izolovana je po korisniku sa kvotama, a neaktivne GPU sesije se oslobađaju bez gubitka rada.

Rizici: operativni (GPU kvote, idle culling) — srednji · tehnički (spawn latencija na SLURM-u) — srednji.

## 7. Pretraga i katalog resursa

**Use case:** sek. 2 (Katalog) · **Domen:** 2 Katalog resursa · **Eliminacioni:** K1 (ACL/ABAC filter na nivou indeksa — neodobren resurs ne ulazi u rezultate, da se ne otkrije ni postojanje) · **Otvoreno:** OD-8 (da li vidljivost uračunava ABAC, ili se proverava tek pri akciji).

Tipovi resursa u katalogu (svi indeksirani u jednom upitu, sa oznakom tipa i porekla):
- **Modeli** — težine, metapodaci, model card, serving status
- **Datasetovi** — artefakti u MinIO, dataset card, GDPR klasa, verzije
- **Kontejneri** — Docker/Apptainer slike u Harbor-u, tagovi, digest
- **Jupyter sveske / šabloni** — kurirani notebook-ovi i zaključani studentski šabloni
- **Workflow šabloni (pipeline definicije)** — redosled koraka, parametri, zavisnosti
- **Benchmark setovi** — evaluacioni setovi sa versioning-om i leaderboard vezom
- **Kursevi / learning pathovi** — LMS sadržaj, upis, sertifikati
- **Šabloni poslova (job šabloni)** — parametri, targeting klaster, lock za studente
- **Alati / plugin-ovi** — eksterni ModelProvider-i i ComputeBackend-i registrovani na platformi

Opcije pretrage i prikaza:
- Pretraga: OpenSearch (full-text, faceti) · pgvector (semantička) · Qdrant/Weaviate (ako semantika preraste pgvector)
- Vidljivost resursa: `private` (samo vlasnik) · `team` (tim/projekat) · `platform` (svi prijavljeni korisnici platforme) · `public` (i anonimni posetioci)
- Sektorski pogledi: konfiguracioni filteri nad istim katalogom
- Provenance: oznaka izvora (lokalno / Pharos / IT4LIA / CKAN)

Kako testiramo: napunimo probni katalog sa 100+ stavki i ~30 realnih upita, obavezno na srpskom i ćirilicom i latinicom („модел за здравство" = „model za zdravstvo"), pa poredimo čist full-text (OpenSearch) sa hibridom + semantika (pgvector) — ocena: da li je pravi rezultat u prvih 5. Kriterijum: pretraga radi podjednako na oba pisma i jezika, a sektorski filter nikad ne propusti neodobren resurs.

Definition of done: pretraga vraća tačan rezultat u prvih 5 na oba pisma i oba jezika, svaki resurs ima jasnu oznaku porekla, a sektorski filter nikad ne propusti neodobren resurs.

Rizici: tehnički (kvalitet semantike na srpskom) — srednji · bezbednosni (filter pristupa u pretrazi) — srednji.

## 8. Obuke i edukacija (LMS)

**Use case:** sek. 8 (Edukacija) · **Domen:** 7 Edukacija · **Kriterijumi:** D6 (LMS iza platformskog API-ja — korisnik ne izlazi iz portala), D3 (zaključan šablon posla za studenta), D4 (read-only montaža pravih podataka), D5 (zbir studentskih sesija unutar pool-a institucije), D7 (sadržaj na sr/en). Pretežno Pilot; infrastruktura sveske (funkc. 6) je MVP.

Opcije:
- LMS: Open edX · Moodle — iza SAIFA API-ja
- Lab vežbe: kurirane sveske + pravi podaci iz MinIO + zaključani SLURM šabloni na ITE
- Jezici: srpski (ćir./lat.) i engleski

Kako testiramo: isti probni kurs (lekcije + jedna hands-on lab vežba) postavimo na Open edX i Moodle i merimo trošak integracije (upis, napredak, sertifikat kroz API, bez izlaska iz portala) i ocenu probnih studenata; lab vežbu testiramo ceo lanac upis → sveska → dataset → posao na klasteru → provera → napredak. Kriterijum: student završi vežbu bez pomoći, a mi procenimo koji LMS traži manje održavanja (Moodle lakši za hosting, Open edX bogatiji).

Definition of done: student završi lab vežbu kroz portal bez izlaska iz SAIFA okruženja, sa upisanim napretkom i (opciono) sertifikatom.

Rizici: organizacioni (obim kurseva nepoznat dok klijenti ne kažu) — srednji · operativni (održavanje LMS-a) — srednji.

## 9. Federacija sa EuroHPC (Pharos, IT4LIA)

**Use case:** sek. 10 (Federacija) · **Domen:** 8 Federacija · **Eliminacioni:** F1 (federacija iza CatalogueSource apstrakcije — novi partner = nov adapter, ne izmena jezgra), F2 (uvezeni resurs ulazi isključivo sa oznakom porekla i ne prepisuje lokalni source of truth). Cela grupa je Pilot (MVP samo mock/read-only); zrelost partnerskih API-ja je najveća nepoznanica projekta.

Opcije:
- Sync kataloga: zakazan + event-driven · izvoz modela/datasetova/benchmarka · uvoz sa oznakom porekla
- Apstrakcija: adapter po partneru (Pharos API, IT4LIA API, CKAN, Dataverse)
- Identitet: Keycloak brokering ka EuroHPC AAI · eduGAIN preko AMRES-a

Kako testiramo: u dva koraka — prvo mock Pharos po objavljenoj šemi (uključujući namerno pokvarene odgovore: sync ne sme upisati ništa polovično), pa test protiv pravog Pharos sandbox-a čim ga dobijemo (spremnost partnera je najveća nepoznanica). Kriterijum: model označen za objavu pojavi se u (mock) partnerskom katalogu bez ručne intervencije, a nijedan kvar partnera ne ošteti naš katalog.

Definition of done: model označen za objavu pojavi se u (mock) partnerskom katalogu bez ručne intervencije, uvezeni resursi nose oznaku porekla, a nijedan kvar partnera ne ošteti naš katalog.

Rizici: zavisnost od partnera (zrelost Pharos/IT4LIA API-ja i metapodataka) — visoka · tehnički (validacija šeme, otpornost na kvarove) — srednji.

## 10. Benchmarking i evaluacija

**Use case:** sek. 5 (Model lifecycle — evaluacija) · **Domen:** 6 Experiment tracking · **Kriterijumi:** E6 (reproduktivnost — bez nje leaderboard nema smisla) · **Otvoreno:** OD-12 (mešanje verzija benchmarka na leaderboard-u).

Opcije:
- Harness: lm-evaluation-harness + sektorski benchmark datasetovi (srpski jezik)
- Izvršavanje: platformski job na HPC ili GPU pool-u
- Leaderboard: objava po verziji benchmarka, uz odobrenje

Kako testiramo: lm-evaluation-harness na dva poznata modela, proveravajući reproduktivnost (dva pokretanja → praktično identično) i cenu (GPU sati po evaluaciji), i merimo koliko je posla uvesti custom srpski task. Kriterijum: evaluacija je ponovljiva, automatizovana kao običan platformski posao i košta predvidivo.

Definition of done: evaluacija je ponovljiva, pokreće se kao običan platformski posao, košta predvidivo, a rezultat ide na leaderboard tek uz odobrenje (vidi funkc. 13). OD-12: jedan leaderboard = jedna verzija benchmarka (preporučeno, bez mešanja).

Rizici: tehnički (reproduktivnost) — srednji · trošak (GPU sati po evaluaciji) — srednji.

## 11. Upravljanje korisnicima i pristupom

**Use case:** sek. 1 (Pristup platformi i nalozi) · **Domen:** 1 Autentikacija · **Eliminacioni:** A1 (provider istovremeno broker-uje SAML2 ka AMRES-u i OIDC ka EuroHPC AAI i svodi ih na jedan interni identitet), A3 (opoziv API ključa i sesije deluje odmah, ne tek po isteku tokena) · **Otvoreno:** OD-5 (korisnički podlimiti kvote), OD-8 (ABAC vidljivost).

Opcije:
- Identity provider: Keycloak (OIDC/OAuth2/SAML2) · prijava lokalna, eduGAIN/AMRES, EuroHPC identitet (MyAccessID)
- Uloge (RBAC): platform-admin, data-admin, model-reviewer, ai-expert, advanced-user, basic-user, student, mentor/instruktor, viewer, federation-partner
- Atributi (ABAC): sektorske sertifikacije, afilijacija → kvote, GDPR saglasnosti, tenant
- Organizacije i dvonivovska kvota: pool organizacije (dodeljuje platform-admin) → lična kvota člana (raspodeljuje predstavnik); nezavisni korisnik dobija ličnu kvotu iz platformskog pool-a (`popis_final.md` §8a). Project/grant accounting: sopstveni model vs. Waldur (koristi ga Pharos) — otvoreno, traži zaseban ADR
- API ključevi: generisanje, scope-ovanje, rotacija, opoziv

Kako testiramo: Keycloak realm sa svim ulogama + automatizovani testovi izolacije koji za svaku ulogu pokušavaju svaku operaciju nad tuđim resursima (sve mora „zabranjeno") i ostaju trajno u CI-ju; eduGAIN sa AMRES test identitetom, i ciklus „napravi → koristi → opozovi API ključ" (opoziv deluje odmah). Kriterijum: nijedan test izolacije ne prolazi pogrešno, ni jednom.

Definition of done: svaka uloga vidi i radi samo ono što sme, nijedan test izolacije ne prolazi pogrešno, a opoziv API ključa deluje odmah.

Rizici: bezbednosni (curenje između tenanta) — visok · organizacioni (eduGAIN/AMRES i EuroHPC AAI pristup) — srednji.

### Admin hijerarhija (nivo ovlašćenja i granice)

| Nivo | Uloga | Može | Ne može bez višeg odobrenja |
| --- | --- | --- | --- |
| 1 | platform-admin | Sve administratorske akcije; blokira naloge, menja kvote organizacija, odobrava organizacije, konfiguriše federaciju | — |
| 2 | data-admin | Odobrava datasetove, GDPR pregled, DPIA, sektorska usklađenost dataseta | Menja uloge korisnika, blokira naloge, pristupa admin panelu korisnika |
| 2 | model-reviewer | Odobrava modele za objavu, vraća na doradu, kontroliše leaderboard objavu | Menja kvote, blokira naloge, GDPR operacije |
| 3 | Predstavnik organizacije | Dodaje/uklanja članove, raspodeljuje kvotu unutar organizacije, kreira timove | Odobrava resurse, pristupa globalnom admin panelu, vidi tuđe organizacije |
| 4 | mentor/instruktor | Kreira kurseve i lab vežbe, zaključava šablone, prati napredak studenata | Menja kvote, pristup admin panelu, odobrava resurse za objavu |
| 5 | Sve ostale uloge | Koriste platformu u okviru dodeljene uloge i kvote | Odobravanje, administracija, vidljivost tuđih resursa van dozvoljene vidljivosti |

Napomena: `data-admin` i `model-reviewer` su ravnopravni na nivou 2 — nijedan ne može uticati na drugi domen (data-admin ne odobrava modele, model-reviewer ne dira GDPR). Oba odgovaraju `platform-admin`-u.

## 12. Administracija, nadzor i usklađenost

**Use case:** sek. 9 (Administracija, monitoring i usklađenost) · **Domen:** presek kroz sve slojeve (`popis_final.md` §11).

Opcije:
- Monitoring: Prometheus + Grafana (HPC redovi, latencija, GPU iskorišćenost, kvote, GPU sati i energija) · Loki (logovi) · OpenTelemetry tracing · Sentry
- Mrežni nadzor pristupnih tačaka i linkova ka HPC: NetFlow · IPFIX · sFlow — lanac exporter (ruter/firewall) → collector (nfdump, GoFlow) → analyzer (ntopng, ElastiFlow + Grafana)
- Audit: append-only log sa hash-chain-om svih operacija, retencija 2+ godine, compliance izveštaji
- GDPR alati: DSAR izvoz, pravo na brisanje, DPIA procedura
- Notifikacije: email + in-app
- Feature flags: Unleash · env varijable

Kako testiramo: monitoring stack dižemo već u dev okruženju i proveravamo da li iz njega odgovaramo na EuroHPC pitanja (GPU sati po projektu, energija, preuzimanja dataseta) — ako ne možemo, fali metrika; mrežni nadzor proveravamo malim NetFlow ogledom (softflowd → nfdump → Grafana), a audit pokušajem izmene/brisanja zapisa (ne sme uspeti ni kao admin baze). Kriterijum: svaki compliance izveštaj dobija se iz sistema bez ručnog kopanja po logovima.

Definition of done: svaki izveštaj koji compliance/EuroHPC traži dobija se iz sistema bez ručnog kopanja po logovima, a audit zapis ne može da se izmeni ni obriše čak ni kao admin baze.

Rizici: compliance (retencija, integritet audita) — srednji · operativni (održavanje monitoring stacka) — nizak.

## 13. Kolaboracija i timski rad

**Use case:** sek. 7 (Kolaboracija i timski rad) · **Domen:** — · **Otvoreno:** OD-1 (zasebno terminalno stanje `rejected` ≠ `draft`), OD-2 (masovna moderacija vidljivosti).

Opcije:
- Projekti i timski workspace-ovi: deljenje modela, datasetova i rezultata unutar tima (`team` vidljivost — `popis_final.md` §8c)
- Review/approval: jedinstvena mašina stanja resursa `Draft → Na recenziji → Odobren → Objavljen…` (`popis_final.md` §11a), uloge Data administrator (datasetovi) i Model reviewer (modeli), odluke `Odobri` / `Odbij` / `Vrati na doradu`; komentari, validation reports
- Kod-kolaboracija: interni GitLab/Gitea (merge request, review) — već potreban za DVC

Kako testiramo: prototip team vidljivosti (model vidljiv timu, nevidljiv ostalima) damo istraživačkoj grupi (3–5 ljudi) da prođe ciklus jedan trenira / drugi pregleda / treći odobrava, krećući od minimuma i dodajući samo što grupa zatraži. Kriterijum: review ciklus od treninga do odobrene objave prođe kroz platformu bez „sa strane" komunikacije.

Definition of done: review ciklus od treninga do odobrene objave prođe kroz platformu, sa jasnim ko-šta-odobrava (vidi RACI ispod), bez komunikacije "sa strane".

Rizici: organizacioni (definisanje ownership i approval procesa) — srednji · tehnički (team vidljivost) — nizak.

## 14. Workflow orchestration (ML pipeline-ovi)

**Use case:** presek kroz model lifecycle (sek. 5) · **Domen:** — (decision tree `popis_final.md` §15).

Opcije:
- Kubeflow Pipelines · Apache Airflow · Argo Workflows · sopstveni job-scheduler (dovoljan za jednostavne lance, već u arhitekturi)

Kako testiramo: referentni pipeline od četiri koraka (preprocesiranje → trening → evaluacija → registracija) prvo izvedemo sopstvenim job-scheduler-om sa ulančavanjem preko događaja; ako je čitljivo i pouzdano, generički engine ne uvodimo. Tek ako klijenti traže složenije lance (grananje, retry, paralelizam) radimo PoC sa Argo Workflows. Kriterijum: odluku vodi složenost koju klijenti stvarno traže u upitniku, ne zamišljena.

Definition of done: referentni pipeline (preprocesiranje → trening → evaluacija → registracija) prođe kroz sopstveni job-scheduler bez ručnih intervencija; generički engine se uvodi samo ako ga klijentske potrebe opravdaju.

Rizici: tehnički (složenost koju klijenti traže nepoznata) — srednji · rizik prekomerne izgradnje (uvođenje generičkog engine-a bez potrebe) — srednji.

## Ownership i odobravanje (RACI)

Sledeća matrica definiše vlasništvo nad ključnim koracima životnog ciklusa resursa, usklađeno sa mašinom stanja iz `popis_final.md` §11a. R = radi, A = krajnje odgovoran (jedan po redu), C = konsultuje se, I = obaveštava se.

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

## 15. Ticketing i korisnička podrška

**Use case:** presek (operativna podrška uz sek. 9) · **Domen:** — · Vezano za audit log (sek. 12) radi konteksta zahteva.

Opcije:
- MVP: forma „Prijavi problem" u portalu → šalje email platformskom timu i kreira GitLab Issue u internom projektu
- Pilot: interni helpdesk (Zammad, Freshdesk self-hosted) sa ticketima, prioritetima, SLA praćenjem i istorijom po korisniku
- Integracija sa audit logom: ticket automatski dobija ID sesije i poslednje akcije korisnika, da tim ne mora ručno da reproduced problem

Kako testiramo: simuliramo realan incident — korisnik ne može da pokrene posao — i gledamo koliko traje put od prijave do rešenja bez ticketing sistema (email lančić) vs. sa njim. Kriterijum: tim vidi sve otvorene zahteve na jednom mestu, bez gubljenja po email inboxu.

Definition of done: korisnik može da prijavi problem iz portala jednim klikom; platformski tim vidi zahtev sa kontekstom (uloga korisnika, poslednje akcije iz audit loga, poruka o grešci).

Rizici: operativni (ko odgovara na tickete, SLA) — srednji · tehnički — nizak.

## 16. Upravljanje kontejnerima

**Use case:** sek. 4 (AI factory), sek. 2 (kontejneri u katalogu) · **Domen:** — (`popis_final.md` §5 HPC kontejneri, §9 Harbor). Vezano za funkc. 5 (Apptainer na klasteru) i funkc. 7 (tip resursa „Kontejneri").

Opcije:
- Build okruženje: Kaniko (bez Docker daemon-a, u Kubernetes podu) · Buildah (rootless) · priprema Dockerfile-a iz šablona
- Registry: Harbor (self-hosted, skeniranje ranjivosti, potpisivanje, RBAC po projektu)
- HPC: push u Harbor → pull na klasteru kao Singularity/Apptainer image (Harbor-Apptainer integracija)
- Zaključani šabloni kontejnera za studente: fiksiran bazni image, korisnik samo dodaje kod/podatke

Kako testiramo: prolazimo ceo lanac — upload Dockerfile-a ili izbor baznog šablona → automatski build u Kaniku → push u Harbor → pull na PARADOX/ITE kao Apptainer → pokretanje posla koji koristi taj image. Merimo vreme (od upload do ready-to-run) i vidimo da li Harbor skeniranje blokira nešto neopravdano.

Definition of done: korisnik registruje kontejner bez lokalne instalacije Docker-a, image je dostupan i na Kubernetes pool-u i na HPC klasteru, a Harbor skeniranje je aktivno.

Rizici: bezbednosni (ranjivosti u kontejnerima) — srednji · operativni (održavanje Harbor-a i build pipeline-a) — srednji.

## 17. Status platforme (vidljivost servisa)

**Use case:** sek. 9 (Administracija, monitoring) · **Domen:** — · Oslanja se na monitoring stack iz funkc. 12 (Prometheus/Grafana alarmi).

Opcije:
- Javna status stranica (bez prijave): prikazuje da li su ključni servisi živi — inference pool, HPC bridge, JupyterHub, katalog, federacija
- Admin health dashboard: detaljni prikaz latencija, grešaka po servisu, zadnji uspešan HPC ping, zadnji sync federacije
- Integracija sa Prometheus/Grafana alertima: status stranica se automatski ažurira kad alarm pukne

Kako testiramo: namerno ugasimo jedan servis (mock) i proverimo: (a) koliko traje do ažuriranja status stranice, (b) da li korisnik vidi jasnu poruku zašto mu inference ne radi. Kriterijum: korisnik sam zaključi "sistem je dole, ne čekam" bez kontaktiranja podrške.

Definition of done: status stranica prikazuje ispravan status svakog ključnog servisa u roku od 60 sekundi od promene stanja; admin vidi širi health dashboard sa metrikama.

Rizici: operativni (lažni alarmi koji zabrinjuju korisnike) — nizak.

## 18. Lični dashboard (individualna statistika)

**Use case:** sek. 9 („Pregled lične istorije aktivnosti", „Praćenje potrošnje") · **Domen:** — · Prikazuje obe vrste kvote iz `upitnik-zahtevi.md` B13 (compute GPU/CPU sati i inference token budžet).

Opcije:
- Pregled potrošnje: token budžet (inference) i compute kvota (GPU/CPU sati) — potrošeno vs. ostalo, po vremenskom prozoru
- Istorija aktivnosti: svi poslovi, inference pozivi, preuzimanja, objave resursa — sa statusom i trajanjem
- Trendovi: potrošnja po nedelji/mesecu, trend koji pomeru pored limita (upozorenje pre iscrpljivanja kvote)
- Quick actions: ponovi posao, preuzmi rezultat, prijavi problem — sve sa lične stranice

Kako testiramo: dajemo dashboard korisnicima posle mesec dana korišćenja platforme i pitamo: "Da li možeš odatle da zaključiš koliko ti kvote ostaje i šta si radio prošle nedelje?" Kriterijum: korisnik odgovara bez otvaranja admin panela ili slanja emaila.

Definition of done: korisnik vidi potrošnju po tipu resursa, istoriju poslova poslednjih 30 dana, i upozorenje kad kvota padne ispod 20%.

Rizici: operativni (količina podataka za prikazivanje, retencija) — nizak.

## 19. Admin statistike (poslovne metrike platforme)

**Use case:** sek. 9 (Administracija, izveštavanje) · **Domen:** — · Direktno nosi cilj „EuroHPC integration" (izveštaj o GPU satima i energiji — `mvp_kriterijumi.md` K4).

Opcije:
- Korisničke metrike: broj registrovanih/aktivnih korisnika po periodu, onboarding trend, distribucija po tipu/sektoru/afilijaciji
- Resursne metrike: broj objavljenih modela/dataseta, inference pozivi po modelu, GPU sati po projektu/organizaciji, energija
- EuroHPC izveštaji: agregatni izveštaji po projektu i organizaciji (izvozivi u CSV/PDF) za compliance i grant accounting
- Anomalije i alarmi: korisnici sa neobičnom potrošnjom, neaktivne organizacije, resurs bez saobraćaja mesecima

Kako testiramo: uzimamo listu pitanja koja EuroHPC traži u polugodišnjem izveštaju i proveravamo da li svako od njih može da se odgovori iz jednog dashboarda bez ručnog SQL-a. Kriterijum: ceo izveštaj se izvozi za 5 minuta, bez ručnih intervencija.

Definition of done: platform-admin generiše EuroHPC izveštaj za izabrani period bez SQL-a; anomalije u potrošnji se pojavljuju automatski.

Rizici: compliance (tačnost podataka u izveštaju) — srednji · operativni (GDPR pri agregiranju po korisniku) — srednji.

---

Sledeći korak: prioritete testiranja određujemo prema odgovorima klijenata iz `upitnik-zahtevi.md`; fazu po scenariju daje `mvp_kriterijumi.md` (K1–K5), a blokade `otvorene_odluke.md` (OD-1…OD-12). Svaki završen test dobija kratak zapisnik odluke (šta smo testirali, rezultati po kriterijumima iz `kriterijumi_izbora_tehnologija.md`, izbor i obrazloženje), koji čuvamo uz ostatak dokumentacije.
