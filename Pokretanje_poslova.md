
**Naziv:** Pokretanje posla iz zaključanog šablona (student)
**Akter:** Student
**Preduslov:** Student je prijavljen preko Keycloak-a i ima ulogu Student; postoji bar jedan zaključan šablon posla dodeljen njegovom kursu/grupi; student ima dodeljenu ličnu kvotu sa dovoljno preostalih resursa za izvršavanje; izlazni parametri i resursi šablona su unapred fiksirani i student ih ne menja.

**Osnovni tok:**
1. Student otvori ekran **„Šabloni poslova"** i izabere dodeljeni šablon iz liste.
2. Sistem prikaže **„Detalji šablona"**: opis posla, fiksirane parametre i polja koja je dozvoljeno popuniti (npr. izbor ulaznog dataseta iz unapred odobrene liste).
3. Student popuni dozvoljena polja i izabere **„Pokreni"**.
4. Sistem proveri ulogu studenta, proveri i rezerviše preostalu ličnu kvotu studenta (da paralelni submit ne potroši isti budžet) i izvrši posao na unapred određenom okruženju zapisanom u šablonu (Kubernetes Job ili SLURM job na ITE/PARADOX — student ne bira).
5. Sistem kreira zapis posla u stanju **„U redu"** i prikaže potvrdu sa identifikatorom posla.

**Alternativni tokovi:**
A1. **Nedovoljna kvota** — u koraku 4: preostala lična kvota studenta ne pokriva procenu posla; sistem odbije pokretanje uz poruku koliko kvote nedostaje i kome da se obrati (mentor/predstavnik); posao se ne kreira i ništa se ne rezerviše.
A2. **Obavezno polje nije popunjeno** — u koraku 3: dozvoljeno polje (npr. ulazni dataset) nije izabrano; sistem označi prazno polje i ne pokreće posao; uneto se zadržava i student se vraća na korak 3.
A3. **Šablon povučen ili istekao** — u koraku 1: šablon više nije aktivan (mentor ga povukao ili je rok kursa istekao); sistem prikaže da šablon nije dostupan i uklanja ga iz liste.
A4. **Odustajanje** — u koraku 3: student napusti ekran bez pokretanja; nijedan posao se ne kreira.

**Rezultat:** Kreiran je posao iz zaključanog šablona u stanju **„U redu"**, vezan za studenta, sa rezervisanom ličnom kvotom i parametrima u granicama koje šablon dozvoljava; posao čeka izvršavanje na unapred određenom okruženju; dalji životni ciklus prati se kroz scenario *Praćenje statusa i logova posla*; kreiranje je upisano u audit log. Student nije imao pristup pisanju skripte niti izboru klastera.


**Naziv:** Pokretanje računski zahtevnog posla
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen preko Keycloak-a i ima pravo na pokretanje poslova; postoje ulazni resursi (model i/ili dataset u MinIO) nad kojima korisnik ima pravo pristupa; korisnik ima dodeljenu ličnu kvotu sa dovoljno preostalih resursa.

**Osnovni tok:**
1. Prijavljen korisnik otvori ekran **„Novi posao"** (alternativno: pokreće isti posao iz sveske preko SAIFA SDK-a ili REST API-ja — vidi A5).
2. Sistem prikaže formu **„Novi posao"** sa izborom tipa posla (fine-tuning / trening / batch obrada), ulaznih resursa, parametara i odredišta rezultata u MinIO.
3. Korisnik izabere tip posla, ulazne resurse (iz resursa nad kojima ima pravo pristupa), podesi parametre i odredište i izabere **„Pokreni posao"**.
4. Sistem proveri da su obavezna polja popunjena; ako jesu, prosledi zahtev na proveru prava i kvote.
5. Sistem proveri pristup korisnika ulaznim resursima (uloga, ABAC) i atomično proveri i rezerviše procenjenu ličnu kvotu korisnika (da paralelni submit ne potroši isti budžet).
6. Sistem odredi izvršno okruženje prema pravilu rutiranja: ako ograničenje pristupa podacima vezuje dataset za određeni klaster (compute-to-data), posao ide tamo; inače se okruženje bira po preostalim kriterijumima rutiranja, a kada je više klastera dopušteno, korisnik bira između ponuđenih.
7. Sistem stavi posao u red, kreira zapis posla u stanju **„U redu"** i prikaže potvrdu sa identifikatorom posla.

**Alternativni tokovi:**
A1. **Nedostaju obavezna polja** — u koraku 4: ulazni resurs ili odredište nisu izabrani; sistem označi prazna polja i ne pokreće posao; forma zadržava uneto i korisnik se vraća na korak 3.
A2. **Nema pristup ulaznom resursu** — u koraku 5: uloga ili ABAC atributi ne pokrivaju model ili dataset (npr. sektorska sertifikacija za zdravstvo); sistem odbije pokretanje uz poruku o nedostatku dozvole; posao se ne kreira.
A3. **Nedovoljna kvota** — u koraku 5: preostala lična kvota ne pokriva procenu posla; sistem odbije pokretanje i prikaže koliko kvote nedostaje, uz uput na **„Zatraži povećanje kvote"**; posao se ne kreira i ništa se ne rezerviše.
A4. **Kvota potrošena tokom rezervacije** — u koraku 5: provera prođe ali rezervacija ne uspe jer je paralelni posao u međuvremenu potrošio preostalu kvotu; sistem odbije pokretanje uz istu poruku kao A3; posao se ne kreira.
A5. **Podaci vezani za klaster koji je nedostupan** — u koraku 6: compute-to-data ograničenje zahteva klaster (npr. ITE) koji trenutno ne prihvata poslove; sistem odbije pokretanje uz poruku da odredišni klaster nije dostupan, da se posao ne može preusmeriti jer podaci ne smeju da ga napuste, i da pokuša ponovo kasnije; rezervisana kvota se oslobađa; posao se ne kreira.
A6. **Pokretanje iz sveske / API-ja** — u koraku 1–3: korisnik umesto forme šalje posao preko SAIFA SDK-a iz sveske ili REST poziva; iste provere (koraci 4–6) važe identično, autentikacija ide preko sesije sveske ili API ključa; ishod je isti zapis posla.
A7. **Odustajanje** — u koraku 3: korisnik napusti formu bez pokretanja; nijedan posao se ne kreira; uneto u formi se ne čuva (za razliku od A1, gde se unos zadržava jer se korisnik vraća da dopuni).

**Rezultat:** Kreiran je posao u stanju **„U redu"**, vezan za korisnika i ulazne resurse, sa rezervisanom ličnom kvotom (korak 5); posao čeka izvršavanje na Kubernetes/SLURM okruženju određenom u koraku 6; kreiranje je upisano u audit log.


**Naziv:** Praćenje statusa i logova posla
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen preko Keycloak-a; postoji bar jedan posao koji je korisnik pokrenuo, u nekom od stanja izvršavanja (**„U redu"**, **„U toku"**, **„Završen"**, **„Neuspeo"**, **„Prekinut"**).

**Osnovni tok:**
1. Prijavljen korisnik otvori ekran **„Moji poslovi"**.
2. Sistem prikaže listu korisnikovih poslova sa tipom, statusom i vremenom pokretanja.
3. Korisnik izabere posao iz liste.
4. Sistem prikaže **„Detalji posla"**: trenutni status, poziciju u redu (ako je u redu), procenu potrošnje resursa do tada (ako je dostupna iz scheduler-a) i live log izvršavanja.
5. Korisnik prati osvežavanje statusa i logova dok posao traje.
6. Po prelasku posla u završno stanje, sistem prikaže konačni status (**„Završen"** sa linkom ka rezultatima u MinIO, ili **„Neuspeo"** / **„Prekinut"**).

**Alternativni tokovi:**
A1. **Posao još u redu** — u koraku 4: posao nije počeo da se izvršava; sistem prikaže poziciju u redu i procenu čekanja (ako je dostupna iz SLURM-a/scheduler-a); logovi izvršavanja još ne postoje.
A2. **Posao neuspeo** — u koraku 6: izvršavanje se završilo greškom; sistem prikaže status **„Neuspeo"** sa porukom o grešci i poslednjim logovima; rezultata nema; korisnik može da pokrene posao ponovo iz **„Detalji posla"** sa istim parametrima, pri čemu re-run prolazi iste provere pristupa i kvote (koraci 4–6 scenarija *Pokretanje računski zahtevnog posla*) bez ponovnog popunjavanja forme.
A3. **Kašnjenje live log streama** — u koraku 5: prikaz logova zaostaje za stvarnim izvršavanjem; sistem nastavlja da dopunjuje logove kad stignu; status posla ostaje verodostojan jer se čita iz scheduler-a, ne iz log streama.
A4. **Posao prisilno zaustavljen spolja** — u koraku 5/6: posao je zaustavio administrator platforme (zbog stabilnosti klastera) ili je zaustavljen pri deaktivaciji naloga; sistem prikaže status **„Prekinut"** sa naznakom da zaustavljanje nije pokrenuo sam korisnik.

**Rezultat:** Korisnik je video aktuelni status, potrošnju i logove posla; za završen posao ima link ka rezultatima u MinIO. Pregled ne menja stanje posla. Prekid posla od strane korisnika obrađen je u zasebnom scenariju.


**Naziv:** Preuzimanje rezultata posla
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen preko Keycloak-a; postoji posao koji je korisnik pokrenuo u stanju **„Završen"**; rezultati su upisani u odredišni prostor u MinIO i korisnik nad njima ima pravo čitanja.

**Osnovni tok:**
1. Prijavljen korisnik otvori **„Detalji posla"** za završen posao.
2. Sistem prikaže status **„Završen"** i odeljak **„Rezultati"** sa listom izlaznih artefakata (fajlovi, log, metrika) u MinIO.
3. Korisnik izabere **„Preuzmi rezultate"** (ceo skup) ili pojedinačni artefakt iz liste.
4. Sistem generiše privremeni (presigned) link i započne preuzimanje izabranih artefakata u korisnikovo lokalno okruženje.
5. Sistem zabeleži preuzimanje rezultata u audit log.

**Alternativni tokovi:**
A1. **Rezultat je novi resurs u registru** — u koraku 2: izlaz posla je model (npr. fine-tune rezultat); umesto preuzimanja fajlova korisnik izabere **„Registruj kao resurs"** i model se upisuje u registar u stanju **„Draft"** (objava ide kroz zaseban tok registracije i objave resursa); preuzimanje fajlova i dalje ostaje moguće.
A2. **Posao nije završen** — u koraku 1: posao je u stanju **„U redu"** / **„U toku"** / **„Neuspeo"** / **„Prekinut"**; sistem ne nudi preuzimanje rezultata (za neuspeo/prekinut posao nudi samo logove); tok se prekida.
A3. **Rezultati istekli ili obrisani** — u koraku 2: artefakti su prošli retenciju ili su obrisani; sistem prikaže poruku da rezultati više nisu dostupni i da posao treba ponoviti; preuzimanje nije moguće.
A4. **Prekid preuzimanja** — u koraku 4: veza se prekine tokom preuzimanja velikog artefakta; preuzimanje se nastavlja/ponavlja (resumable), original u MinIO ostaje netaknut.

**Rezultat:** Izabrani rezultati posla preuzeti su u korisnikovo okruženje (ili je izlaz registrovan kao novi resurs); originali ostaju u MinIO; preuzimanje je upisano u audit log. Stanje posla nije promenjeno.


**Naziv:** Prijem notifikacije o ishodu posla
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik ima bar jedan pokrenut posao; u podešavanjima profila notifikacije su uključene (in-app podrazumevano, email opciono).

**Osnovni tok:**
1. Posao korisnika pređe u završno stanje (**„Završen"**, **„Neuspeo"** ili **„Prekinut"**), što sistem detektuje.
2. Sistem kreira notifikaciju sa tipom posla, identifikatorom, konačnim statusom i linkom ka **„Detalji posla"**.
3. Sistem prikaže notifikaciju u zvonu **„Obaveštenja"** u portalu i, ako je email uključen, pošalje email.
4. Korisnik otvori **„Obaveštenja"** i izabere notifikaciju.
5. Sistem otvori **„Detalji posla"** sa konačnim statusom (i linkom ka rezultatima ako je posao završen uspešno).

**Alternativni tokovi:**
A1. **Email isporuka ne uspe** — u koraku 3: email se ne može isporučiti (nevažeća adresa, odbijen); in-app notifikacija ostaje vidljiva u portalu; sistem ne ponavlja email u nedogled.
A2. **Korisnik je isključio email notifikacije** — u koraku 3: prema podešavanjima profila šalje se samo in-app notifikacija; email se ne šalje.
A3. **Notifikacija pročitana** — u koraku 4: korisnik označi notifikaciju kao pročitanu (ili je otvori); sistem skloni indikator nepročitanog; notifikacija ostaje u istoriji obaveštenja.

**Rezultat:** Korisnik je obavešten o ishodu posla kroz in-app notifikaciju (i email ako je uključen) i iz nje može direktno otvoriti detalje posla. Stanje posla nije promenjeno notifikacijom.


**Naziv:** Zahtev za povećanje kvote
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen preko Keycloak-a i ima dodeljenu ličnu kvotu; korisnik vidi sopstvenu potrošnju i preostali deo dodeljene kvote.

**Osnovni tok:**
1. Prijavljen korisnik otvori ekran **„Moja kvota"**.
2. Sistem prikaže dodeljenu kvotu, dosadašnju potrošnju i preostali deo (CPU sati, GPU sati, prostor).
3. Korisnik izabere **„Zatraži povećanje kvote"**.
4. Sistem prikaže formu **„Zahtev za kvotu"** sa poljima za traženi iznos (po tipu resursa) i obrazloženje; uz formu prikaže trenutnu potrošnju kao kontekst za odlučioca.
5. Korisnik unese traženi iznos i obrazloženje i izabere **„Pošalji zahtev"**.
6. Sistem proveri da su iznos i obrazloženje uneti, kreira zahtev u stanju **„Na čekanju"** i prosledi ga vlasniku odluke: predstavniku organizacije ako je korisnik član organizacije, odnosno platform administratoru ako je korisnik Nezavisni korisnik.
7. Sistem prikaže potvrdu da je zahtev poslat i obavesti odlučioca.

**Alternativni tokovi:**
A1. **Nedostaje iznos ili obrazloženje** — u koraku 6: obavezno polje nije popunjeno; sistem označi prazno polje i ne šalje zahtev; uneto se zadržava i korisnik se vraća na korak 5.
A2. **Već postoji zahtev na čekanju** — u koraku 3: korisnik ima nerešen zahtev za kvotu; sistem ne nudi novi, već prikaže status postojećeg; tok se prekida.
A3. **Odustajanje** — u koraku 4 ili 5: korisnik napusti formu bez slanja; nijedan zahtev se ne kreira.

**Rezultat:** Kreiran je zahtev za povećanje kvote u stanju **„Na čekanju"**, vezan za korisnika i prosleđen predstavniku organizacije (ili platform administratoru za Nezavisnog korisnika), sa traženim iznosom i obrazloženjem; slanje je upisano u audit log. Lična kvota korisnika još nije promenjena.


**Naziv:** Odluka o zahtevu člana za povećanje kvote
**Akter:** Predstavnik organizacije
**Preduslov:** Predstavnik je prijavljen preko Keycloak-a i ima aktivnu organizaciju; postoji bar jedan zahtev člana za povećanje kvote u stanju **„Na čekanju"**.

**Osnovni tok:**
1. Predstavnik organizacije otvori **„Moja organizacija"** → **„Zahtevi za kvotu"** i izabere zahtev iz liste.
2. Sistem prikaže detalje zahteva: ime člana, traženi iznos po tipu resursa, obrazloženje, dosadašnju potrošnju člana i preostali kapacitet pool-a organizacije.
3. Predstavnik pregleda zahtev i izabere **„Odobri"**.
4. Sistem zatraži potvrdu i prikaže koliko će se oduzeti iz pool-a organizacije i koliko ostaje.
5. Predstavnik potvrdi (uz mogućnost da odobri manji iznos od traženog).
6. Sistem prebaci odobreni iznos iz pool-a organizacije na ličnu kvotu člana, postavi zahtev u stanje **„Odobren"** i obavesti člana.
7. Sistem prikaže ažuriranu raspodelu kvote u organizaciji.

**Alternativni tokovi:**
A1. **Odbijanje zahteva** — u koraku 3: predstavnik izabere **„Odbij"**; sistem zatraži obavezan razlog, postavi zahtev u stanje **„Odbijen"**, obavesti člana sa razlogom; kvota se ne menja; tok se završava.
A2. **Pool organizacije nema dovoljno kvote** — u koraku 4: traženi (ili umanjeni) iznos prelazi preostali kapacitet pool-a; sistem upozori da odobravanje nije moguće bez prethodnog povećanja kvote organizacije i uputi predstavnika da zatraži povećanje od platform administratora; odobravanje se ne izvršava.
A3. **Zahtev u međuvremenu povučen ili nevažeći** — u koraku 3: član je povukao zahtev ili je deaktiviran; sistem prikaže da zahtev više nije aktivan i uklanja ga iz liste; predstavnik se vraća na korak 1.
A4. **Odustajanje** — u koraku 4 ili 5: predstavnik napusti ekran bez potvrde; zahtev ostaje u stanju **„Na čekanju"**; nijedna promena se ne upisuje.

**Rezultat:** Zahtev je u stanju **„Odobren"** (uz prenos kvote iz pool-a organizacije na ličnu kvotu člana) ili **„Odbijen"** (bez promene kvote); član je obavešten o ishodu; odluka i prenos kvote upisani su u audit log.


**Naziv:** Prisilno zaustavljanje posla koji ugrožava stabilnost klastera
**Akter:** Platform administrator
**Preduslov:** Platform administrator je prijavljen preko Keycloak-a i ima dozvolu za upravljanje poslovima na klasteru; postoji posao u stanju **„U toku"** za koji nadzor pokazuje da ugrožava stabilnost klastera (npr. prekoračenje resursa, zaglavljen posao, opterećenje koje blokira druge).

**Osnovni tok:**
1. Platform administrator otvori ekran **„Svi poslovi"** (administratorski pregled svih aktivnih poslova na PARADOX/ITE i Kubernetes pool-u).
2. Sistem prikaže listu aktivnih poslova sa vlasnikom, klasterom, potrošnjom resursa i statusom.
3. Administrator izabere problematičan posao i otvori **„Detalji posla"**.
4. Administrator izabere **„Prisilno zaustavi posao"**.
5. Sistem zatraži obavezan razlog (zaustavljanje mora biti zabeleženo u audit log).
6. Administrator unese razlog i potvrdi.
7. Sistem pošalje signal za zaustavljanje poslu na odgovarajućem klasteru (preko SLURM-a ili Kubernetes-a) i postavi posao u prelazno stanje **„Zaustavljanje u toku"**.
8. Po potvrdi klastera da je posao zaustavljen, sistem oslobodi rezervisanu kvotu i obračuna stvarnu potrošnju do prekida, postavi posao u stanje **„Prekinut"**, obavesti vlasnika posla sa razlogom i prikaže administratoru potvrdu.

**Alternativni tokovi:**
A1. **Posao se u međuvremenu završio** — u koraku 4: posao je prešao u **„Završen"** / **„Neuspeo"** pre zaustavljanja; sistem prikaže da posao više nije aktivan i ne preduzima ništa; administrator se vraća na korak 1.
A2. **Signal za zaustavljanje ne prolazi odmah** — u koraku 8: klaster ne potvrdi zaustavljanje u očekivanom roku; sistem zadrži posao u stanju **„Zaustavljanje u toku"**, ponavlja/eskalira signal i ne prebacuje posao u **„Prekinut"** dok klaster ne potvrdi; administrator vidi da prekid još nije potvrđen.
A3. **Odustajanje** — u koraku 5 ili 6: administrator napusti ekran bez potvrde; nijedna promena se ne upisuje; posao nastavlja da se izvršava.

**Rezultat:** Posao je u stanju **„Prekinut"**, rezervisana kvota oslobođena a stvarna potrošnja do prekida obračunata; vlasnik je obavešten; ko, kada i zašto je zaustavio posao zabeleženo je u audit log.
