## 5.1 Treniranje modela nad datasetom

**Naziv:** Treniranje modela nad datasetom iz kataloga

**Akter:** Prijavljen korisnik

**Preduslov:** Korisnik je prijavljen i ima dozvolu pristupa odabranom datasetu; korisnik ima dovoljnu kvotu (GPU sati) za pokretanje posla.

**Osnovni tok:**

1. Prijavljen korisnik otvori ekran **Katalog** i u **Filteri** izabere dataset za trening.
2. Korisnik na ekranu **Detalji resursa** izabere akciju **Pokreni trening**.
3. Sistem prikaže formu **Konfiguracija treninga** sa baznim modelom/arhitekturom, hiperparametrima i izborom okruženja izvršavanja (PARADOX/ITE ili GPU pool).
4. Korisnik popuni konfiguraciju i klikne **Pošalji posao**.
5. Posao se kreira i prati kako je opisano u scenariju *Pokretanje računski zahtevnog posla* (provera kvote i dozvola, kreiranje posla, ekran **Status posla**, prelaz `U redu` → `U toku`, live logovi i napredak).
6. Po prelasku posla u `Završen`, Sistem upiše artefakte treninga u artefakt skladište, poveže ih sa zapisom posla i sa automatski uhvaćenim lineage-om (dataset, konfiguracija, izvršni job), pa pošalje notifikaciju.

**Alternativni tokovi:**

**A1. Nema dozvole za dataset** — u koraku 1: dataset se ne pojavljuje u rezultatima ili je akcija **Pokreni trening** onemogućena; tok se prekida pre slanja.

**A2. Posao ne prođe proveru kvote ili padne / korisnik ga prekine** — obrađeno scenarijem *Pokretanje računski zahtevnog posla* (stanja `Neuspeo` / `Zaustavljanje u toku` → `Prekinut`). Posledica za ovaj tok: ako posao nije `Završen`, artefakti se ne upisuju i model se ne kreira; eventualni delimični checkpoint ostaje vezan za zapis posla kao artefakt posla, ne kao model.

**Rezultat:** Posao treninga je u stanju `Završen`; artefakti modela su u artefakt skladištu, povezani sa zapisom posla i automatskim lineage-om (dataset, konfiguracija, job). Model još nije registrovan (vidi 5.3).

---

## 5.2 Fine-tuning postojećeg modela nad sopstvenim datasetom

**Naziv:** Fine-tuning modela nad sopstvenim labeled datasetom

**Akter:** Prijavljen korisnik

**Preduslov:** Korisnik je prijavljen; postojeći bazni model je dostupan u katalogu; korisnik je već registrovao sopstveni labeled dataset i ima dozvolu pristupa nad njim; korisnik ima dovoljnu kvotu.

> **Napomena (granica 5.1 ↔ 5.2):** treniranje od nule i fine-tuning dele isti tok izvršavanja (računski posao) i razlikuju se samo ulaznom tačkom (dataset-first vs model-first) i sadržajem konfiguracione forme. Tip posla (`trening` / `fine-tuning`) je polje u formi. Zadržani su kao dva scenarija jer je ulazna tačka u UI-ju različita; ako se u implementaciji svedu na jednu formu sa prekidačem, spojiti u jedan parametrizovan scenario.

**Osnovni tok:**

1. Prijavljen korisnik otvori **Katalog**, nađe bazni model i na **Detalji resursa** izabere **Fine-tuning**.
2. Sistem prikaže formu **Konfiguracija fine-tuning-a** sa izborom sopstvenog dataseta, strategijom treninga (pun fine-tuning ili PEFT — LoRA/QLoRA) i hiperparametrima.
3. Korisnik izabere dataset, strategiju i parametre, pa klikne **Pošalji posao**.
4. Posao se kreira i prati kako je opisano u scenariju *Pokretanje računski zahtevnog posla*.
5. Po prelasku posla u `Završen`, Sistem upiše dobijeni checkpoint u artefakt skladište sa automatskim lineage-om (bazni model, dataset, konfiguracija, job) i notifikuje korisnika.

**Alternativni tokovi:**

**A1. Nekompatibilan dataset** — u koraku 3: Sistem prijavi grešku validacije (format/zadatak ne odgovara baznom modelu); posao se ne kreira.

**A2. Posao ne prođe proveru, padne ili korisnik prekine** — obrađeno scenarijem *Pokretanje računski zahtevnog posla*; checkpoint se ne upisuje ako posao nije `Završen`.

**A3. Fine-tuning konvergira u model gori od baznog** — u koraku 5: posao jeste `Završen`, ali evaluacija (5.8) pokazuje pad u odnosu na baseline. Ovo nije greška toka — checkpoint se upisuje normalno; odluka da li ga uopšte registrovati/predložiti za objavu donosi se kasnije, na osnovu evaluacije. Naznačeno ovde da se zna da `Završen` posao ne znači „dobar model".

**Rezultat:** Fine-tune-ovani checkpoint je u artefakt skladištu sa automatskim lineage-om ka baznom modelu i datasetu. Model još nije registrovan (vidi 5.3).

---

## 5.3 Fine-tuning nad osetljivim podacima (compute-to-data) `[osetljivost = atribut]`

**Naziv:** Fine-tuning nad osetljivim podacima koji ne napuštaju kontrolisano okruženje

**Akter:** Prijavljen korisnik

**Preduslov:** Korisnik je prijavljen; dataset je označen kao osetljiv (atribut osetljivosti) i vezan za kontrolisano okruženje; korisnik ima ABAC atribut/sertifikaciju potreban za taj dataset; korisnik ima dovoljnu kvotu na klasteru gde podaci žive.

**Osnovni tok:**

1. Prijavljen korisnik otvori **Katalog** i izabere bazni model, pa **Fine-tuning**.
2. Sistem prikaže formu i ponudi samo ona okruženja izvršavanja koja zadovoljavaju režim osetljivog dataseta (klaster na kom podaci žive); opcija download podataka nije ponuđena.
3. Korisnik izabere osetljivi dataset i parametre, pa klikne **Pošalji posao**.
4. Sistem proveri ABAC atribut, dozvole i kvotu i veže izvršavanje za kontrolisano okruženje; posao se dalje prati kako je opisano u scenariju *Pokretanje računski zahtevnog posla*.
5. Sistem pokrene posao tamo gde podaci žive; korisnik prati napredak i logove, ali nema pristup sirovim podacima.
6. Po prelasku posla u `Završen`, iz kontrolisane zone izlazi **samo artefakt modela (pune težine ili PEFT adapter), nakon prolaska kroz kontrolu izlaska**; Sistem ga upiše u artefakt skladište sa lineage-om i oznakom `compute-to-data`, pa notifikuje korisnika.

**Alternativni tokovi:**

**A1. Korisnik nema potreban ABAC atribut** — u koraku 4: Sistem odbije posao uz poruku da nedostaje sektorska sertifikacija/saglasnost; posao se ne kreira. *(Realno bi se akcija Fine-tuning mogla onemogućiti već u koraku 1; provera je ostavljena i u koraku 4 jer atribut može da nedostaje i pored vidljivosti modela — vidi otvorenu tačku ABAC vidljivosti.)*

**A2. Posao padne ili korisnik prekine** — obrađeno scenarijem *Pokretanje računski zahtevnog posla*; logovi se čuvaju bez izlaganja osetljivih podataka, artefakt ne izlazi iz zone.

**Rezultat:** Artefakt modela je u artefakt skladištu sa lineage-om i oznakom `compute-to-data`; sirovi osetljivi podaci nisu napustili kontrolisano okruženje.


## 5.4 Praćenje i poređenje eksperimenata

**Naziv:** Pregled i poređenje eksperimenata (experiment tracking)

**Akter:** Prijavljen korisnik

**Preduslov:** Korisnik je prijavljen; bar jedan trening/fine-tuning je pokrenut kroz platformu (5.1–5.3) i povezan sa korisnikovim (ili timskim) okruženjem za experiment tracking.

**Osnovni tok:**

1. Prijavljen korisnik otvori ekran **Eksperimenti** i vidi `run`-ove sa parametrima i metrikama (uživo za poslove `U toku`, konačno za `Završen`).
2. Korisnik izabere dva ili više `run`-ova i klikne **Uporedi**.
3. Sistem prikaže uporedni prikaz parametara i metrika (tabela i grafici).
4. Korisnik na osnovu prikaza identifikuje najbolji `run`. *(Izbor „najboljeg" je ljudska procena po metrici koju korisnik bira — sistem ne promoviše automatski.)*

**Alternativni tokovi:**

**A1. Izabran samo jedan `run`** — u koraku 2: dugme **Uporedi** je onemogućeno dok se ne izabere bar dva.

**A2. `run`-ovi nemaju zajedničke metrike** — u koraku 3: Sistem prikaže dostupne metrike i obeleži one koje kod nekog `run`-a nema; poređenje se prikazuje sa prazninama.

**A3. Alat za experiment tracking nedostupan tokom posla na klasteru** — beleženje za vreme nedostupnosti izostane; po vraćanju veze nastavlja se, ili `run` ostaje delimičan uz oznaku. *(Ponašanje preko delimično izolovane veze ka klasteru je eliminacioni kriterijum iz `funkcionalnosti.md` — nije rešeno, vidi otvorene tačke.)*

**Rezultat:** Prikazan je uporedni prikaz eksperimenata; nijedno stanje sistema se ne menja (read-only operacija).


## 5.5 Registracija modela u registar

**Naziv:** Registracija istreniranog artefakta kao modela (verzija 1)

**Akter:** Prijavljen korisnik

**Preduslov:** Korisnik je prijavljen; postoji istreniran model/checkpoint kao artefakt u artefakt skladištu koji pripada korisniku ili njegovom timu.

**Osnovni tok:**

1. Prijavljen korisnik otvori ekran **Moji modeli** i izabere **Registruj model**.
2. Sistem prikaže formu **Registracija modela** sa poljima za metapodatke (naziv, licenca, jezik, tip zadatka, vidljivost public/private/team) i izborom izvornog artefakta.
3. Korisnik poveže artefakt, popuni metapodatke i klikne **Sačuvaj**.
4. Sistem kreira zapis u registru u stanju `Draft`, dodeli mu verziju 1 i poveže lineage.
5. Sistem prikaže **Detalji resursa** novog modela sa stanjem `Draft`, verzijom i vidljivošću.

> **Lineage — dva izvora poverenja:** ako je artefakt nastao kroz platformski posao (5.1–5.3), lineage je **automatski uhvaćen** (dataset, konfiguracija, job) i pouzdan. Ako korisnik ručno povezuje artefakt uvezen spolja, lineage se unosi rukom i obeležava se kao `lineage: nepotvrđen`, jer Sistem ne može da garantuje tačnost ručno unete veze. Ova razlika mora biti vidljiva na **Detalji resursa**.

**Alternativni tokovi:**

**A1. Nepotpuni metapodaci** — u koraku 3: Sistem označi obavezna polja koja nedostaju (npr. licenca); zapis se ne kreira dok se ne popune.

**A2. Model sa istim nazivom već postoji** — u koraku 4: Sistem ponudi da se postojeći model verzioniše (vidi 5.6) umesto kreiranja novog zapisa; korisnik bira.

**A3. Odustajanje** — u bilo kom koraku pre koraka 4: korisnik napusti formu; zapis se ne kreira.

**Rezultat:** Model je registrovan kao verzija 1 u stanju `Draft`, sa metapodacima i lineage-om (potvrđenim ili `nepotvrđen`), vidljiv prema podešenoj vidljivosti.

---

## 5.6 Verzionisanje modela (nova verzija)

**Naziv:** Dodavanje nove verzije postojećeg modela

**Akter:** Prijavljen korisnik

**Preduslov:** Korisnik je prijavljen; model je već registrovan; korisnik ima novi checkpoint/artefakt za istu liniju modela i dozvolu izmene.


**Osnovni tok:**

1. Prijavljen korisnik otvori **Detalji resursa** registrovanog modela i izabere **Dodaj verziju**.
2. Sistem prikaže formu za povezivanje novog artefakta i lineage podataka (dataset, kod, job).
3. Korisnik poveže artefakt, popuni izvor i klikne **Sačuvaj**.
4. Sistem kreira novu verziju (npr. v2) u stanju `Draft` i poveže njen lineage; prethodne verzije ostaju dostupne.
5. Sistem prikaže ažurirane **Detalji resursa** sa listom verzija.

**Alternativni tokovi:**

**A1. Nepotpun lineage** — u koraku 3: Sistem upozori da nedostaje izvor (dataset ili job). Da li se verzija sme sačuvati sa nepotpunim lineage-om zavisi od lineage politike: ako politika to dozvoljava, verzija se snima sa oznakom `lineage: nepotpun`; inače se traži dopuna. *(Governance pravilo — vidi otvorene tačke.)*

**A2. Nema dozvole izmene** — u koraku 1: akcija **Dodaj verziju** je onemogućena.

**Rezultat:** Model ima novu verziju u stanju `Draft` sa povezanim lineage-om; prethodne verzije ostaju dostupne za pregled i rollback.


## 5.7 Promocija checkpoint-a iz eksperimenta u registar

**Naziv:** Promocija `run`-a u verziju registrovanog modela

**Akter:** Prijavljen korisnik

**Preduslov:** Korisnik je prijavljen; izabrani `run` ima validan artefakt modela; korisnik ima dozvolu upisa u registar za taj model.

**Osnovni tok:**

1. Prijavljen korisnik na ekranu **Eksperimenti** otvori izabrani `run` i klikne **Promoviši u registar**.
2. Sistem prikaže formu sa izborom ciljne linije modela (postojeća ili nova).
3. Korisnik potvrdi i klikne **Promoviši**.
4. Sistem kreira verziju modela iz tog `run`-a u stanju `Draft` i poveže automatski lineage ka `run`-u (a preko njega ka datasetu, konfiguraciji i poslu).
5. Sistem prikaže **Detalji resursa** modela sa novom verzijom u stanju `Draft`.


**Alternativni tokovi:**

**A1. `run` nema validan artefakt modela** — u koraku 1: akcija **Promoviši u registar** je onemogućena uz objašnjenje.

**A2. Nema dozvole za upis u registar** — u koraku 3: Sistem odbije promociju uz poruku o dozvoli; verzija se ne kreira.

**A3. Odustajanje** — pre koraka 3: korisnik napusti formu; promocija se ne izvršava.

**Rezultat:** Iz izabranog `run`-a je nastala nova verzija registrovanog modela u stanju `Draft`, sa automatskim lineage-om ka `run`-u.


## 5.8 Pokretanje evaluacije modela

**Naziv:** Evaluacija verzije modela na benchmark setu

**Akter:** Prijavljen korisnik

**Preduslov:** Korisnik je prijavljen; postoji verzija modela za evaluaciju; korisnik ima dovoljnu kvotu.

**Osnovni tok:**

1. Prijavljen korisnik na **Detalji resursa** modela izabere **Pokreni evaluaciju**.
2. Sistem prikaže formu sa izborom benchmark seta i verzije benchmarka (lm-evaluation-harness + sektorski datasetovi).
3. Korisnik izabere benchmark i verziju benchmarka i klikne **Pošalji posao**.
4. Evaluacioni posao se kreira i prati kao platformski posao kroz scenario *Pokretanje računski zahtevnog posla*.
5. Po prelasku posla u `Završen`, Sistem upiše rezultate evaluacije i poveže ih sa verzijom modela i verzijom benchmarka, pa notifikuje korisnika.

**Alternativni tokovi:**

**A1. Izabrani benchmark dataset nije dostupan** — u koraku 2/3: ako sektorski benchmark nije objavljen ili korisnik nema pristup, Sistem ga ne nudi u listi ili blokira slanje uz objašnjenje; posao se ne kreira.

**A2. Posao padne ili korisnik prekine** — obrađeno scenarijem *Pokretanje računski zahtevnog posla*; rezultati se ne upisuju ako posao nije `Završen`.

**Rezultat:** Rezultati evaluacije su sačuvani i povezani sa verzijom modela i verzijom benchmarka, dostupni na **Detalji resursa**.


## 5.9 Slanje verzije modela na recenziju

**Naziv:** Podnošenje verzije modela za recenziju i objavu

**Akter:** Prijavljen korisnik (vlasnik modela / član tima sa dozvolom)

**Preduslov:** Korisnik je prijavljen; postoji verzija modela u stanju `Draft`; verzija ima rezultate evaluacije (5.8).

**Osnovni tok:**

1. Prijavljen korisnik na **Detalji resursa** verzije u stanju `Draft` klikne **Pošalji na recenziju**.
2. Sistem proveri da verzija ima rezultate evaluacije i kompletne obavezne metapodatke.
3. Korisnik (opciono) upiše propratni komentar i potvrdi.
4. Sistem prevede verziju iz `Draft` u `Na recenziji`, stavi je u **Red za pregled** i notifikuje recenzente.

**Alternativni tokovi:**

**A1. Verzija nema evaluaciju** — u koraku 2: Sistem blokira slanje i uputi korisnika na **Pokreni evaluaciju** (5.8); stanje ostaje `Draft`.

**A2. Nepotpuni obavezni metapodaci** — u koraku 2: Sistem označi šta nedostaje; stanje ostaje `Draft`.

**A3. Odustajanje** — pre koraka 4: korisnik napusti tok; stanje ostaje `Draft`.

**Rezultat:** Verzija modela je u stanju `Na recenziji` i nalazi se u **Redu za pregled**, sa zabeleženim podnosiocem i vremenom.


## 5.10 Pregled i odluka recenzenta o objavi

**Naziv:** Pregled evaluacije i odobravanje / odbijanje / vraćanje verzije

**Akter:** Model reviewer (uloga `model-reviewer`)

**Preduslov:** Model reviewer je prijavljen i ima ulogu `model-reviewer`; postoji verzija modela u stanju `Na recenziji` sa rezultatima evaluacije.

**Osnovni tok:**

1. Model reviewer otvori ekran **Red za pregled** i izabere verziju koja čeka odluku.
2. Sistem prikaže **Detalji resursa** sa rezultatima evaluacije, lineage-om i metapodacima.
3. Reviewer pregleda rezultate i upiše komentar.
4. Reviewer klikne **Odobri** ili **Odbij**.
5. Sistem zabeleži odluku, autora i vreme; verzija pređe iz `Na recenziji` u `Odobren` ili `Odbijen`.
6. Sistem notifikuje podnosioca o ishodu.

**Alternativni tokovi:**

**A1. Nepotpuni podaci za odluku** — u koraku 3: reviewer klikne **Vrati podnosiocu** uz komentar šta nedostaje; verzija pređe iz `Na recenziji` u `Vraćeno na doradu` (vraća se na `Draft` na strani podnosioca), bez odluke o objavi.

**A2. Odbijanje** — u koraku 4: reviewer izabere **Odbij** i upiše razlog; verzija pređe u `Odbijen`, ne objavljuje se.

**Rezultat:** Verzija modela je u stanju `Odobren`, `Odbijen` ili `Vraćeno na doradu`, sa zabeleženom odlukom, autorom i komentarom.


## 5.11 Objava modela u lokalni i Pharos katalog

**Naziv:** Objava odobrene verzije lokalno i (opciono) u Pharos katalog

**Akter:** Model reviewer ili Platform administrator (uloga sa dozvolom objave)

**Preduslov:** Postoji verzija modela u stanju `Odobren`.

**Osnovni tok:**

1. Ovlašćeni akter na **Detalji resursa** odobrene verzije klikne **Objavi lokalno**.
2. Sistem prevede verziju u `Objavljen lokalno`, učini je vidljivom u lokalnom katalogu prema podešenoj vidljivosti i zabeleži ko je objavio i kada.
3. Za federaciju: akter dodatno klikne **Objavi u Pharos katalog**.
4. Sistem kroz **adapter ka Pharos-u** mapira metapodatke modela u Pharos šemu kataloga i pošalje zahtev za objavu; po potvrdi, verzija pređe u `Objavljen (Pharos)`.
5. Sistem prikaže **Detalji resursa** sa naznakom statusa objave (lokalno / Pharos) i zabeleženim autorom i vremenom.

**Alternativni tokovi:**

**A1. Verzija nije `Odobren`** — u koraku 1: akcije **Objavi lokalno** / **Objavi u Pharos katalog** su onemogućene.

**A2. Pharos nedostupan ili odbije šemu** — u koraku 4: Sistem ne menja stanje preko `Objavljen lokalno`, prijavi grešku federacije i ponudi ponovni pokušaj; lokalna objava ostaje na snazi. Nijedan kvar partnera ne sme da pokvari lokalno stanje.

**A3. Povlačenje iz upotrebe** — kasnije: ovlašćeni akter klikne **Deprecate**; verzija pređe u `Deprecated` (i dalje dostupna za referencu/lineage, ali označena kao zastarela), a potom opciono u `Arhiviran`.

**Rezultat:** Verzija modela je u stanju `Objavljen lokalno` ili `Objavljen (Pharos)`, vidljiva u odgovarajućem katalogu, sa zabeleženim autorom i vremenom objave.


## 5.12 Rollback servirane verzije modela

**Naziv:** Vraćanje servisnog aliasa na prethodnu verziju

**Akter:** Prijavljen korisnik (sa dozvolom izmene nad modelom)

**Preduslov:** Korisnik je prijavljen; registrovani model ima bar dve verzije; korisnik ima dozvolu izmene nad tim modelom.


**Osnovni tok:**

1. Prijavljen korisnik otvori **Detalji resursa** modela i pređe na karticu **Verzije**.
2. Korisnik izabere raniju verziju i klikne **Vrati na ovu verziju**.
3. Sistem zatraži potvrdu akcije.
4. Korisnik potvrdi.
5. Sistem premesti servisni alias (npr. `production`) na izabranu raniju verziju; sve verzije ostaju sačuvane.
6. Sistem prikaže ažurirane **Detalji resursa** sa naznakom koja je verzija sada servirana.

**Alternativni tokovi:**

**A1. Model ima samo jednu verziju** — u koraku 1: akcija **Vrati na ovu verziju** je onemogućena.

**A2. Odustajanje** — u koraku 4: korisnik odbije potvrdu; servisni alias ostaje nepromenjen.

**A3. Verzija koja se servira vezana je za aktivni deployment** — u koraku 5: Sistem upozori da je verzija u upotrebi i traži dodatnu potvrdu pre prebacivanja. *(Da li rollback dira živu serviranu instancu ili samo pokazivač u registru zavisi od toga koliko model deployment ulazi u MVP — vidi otvorene tačke.)*

**Rezultat:** Servisni alias modela pokazuje na izabranu raniju verziju; nijedna verzija nije obrisana, nijedno lifecycle stanje nije promenjeno.

---

## 5.13 Objava leaderboard-a

**Naziv:** Objava leaderboard-a sa verzionisanjem benchmarka

**Akter:** Platform administrator (uloga `platform-admin`)

**Preduslov:** Platform administrator je prijavljen i ima ulogu `platform-admin`; postoje evaluacioni rezultati više modela na istoj verziji benchmarka.

**Osnovni tok:**

1. Platform administrator otvori ekran **Leaderboard administracija**.
2. Administrator izabere verziju benchmarka i pregleda rezultate kandidata za prikaz.
3. Administrator izabere koje rezultate uključuje i klikne **Objavi leaderboard**.
4. Sistem kreira javno vidljiv leaderboard vezan za tu verziju benchmarka i zabeleži ko ga je objavio i kada.
5. Sistem prikaže objavljeni **Leaderboard** sa naznakom verzije benchmarka.

**Alternativni tokovi:**

**A1. Rezultati sa različitih verzija benchmarka** — u koraku 3: Sistem upozori da se mešaju verzije benchmarka i ne dozvoli objavu dok se ne izabere jedna verzija. *(Da li leaderboard sme da meša verzije benchmarka — pretpostavka je da ne sme jer poređenje gubi smisao; potvrditi.)*

**A2. Odustajanje** — pre koraka 4: administrator napusti ekran; leaderboard se ne objavljuje.

**Rezultat:** Objavljen je leaderboard vezan za tačnu verziju benchmarka, javno vidljiv, sa zabeleženim autorom i vremenom objave.

