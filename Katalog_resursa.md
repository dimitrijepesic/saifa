**Naziv:** Pretraga i filtriranje javnog kataloga
**Akter:** Anonimni posetilac
**Preduslov:** Korisnik nije prijavljen (ne postoji Keycloak sesija); ima pristup SAIFA portalu; anonimnom posetiocu su u katalogu vidljivi samo resursi sa vidljivošću `public`.

**Osnovni tok:**
1. Anonimni posetilac otvori SAIFA portal i klikne **„Katalog"** u zaglavlju.
2. Sistem prikaže listu javnih resursa (modeli, datasetovi, kursevi, workflow-i) sa vidljivošću `public`, uz polje za pretragu i panel **„Filteri"**.
3. Korisnik upiše pojam u polje za pretragu (npr. „model za zdravstvo") i potvrdi.
4. Sistem prikaže javne resurse koji odgovaraju pojmu, uz broj rezultata.
5. Korisnik otvori **„Filteri"** i izabere vrednosti za jezik, licencu, domen i tip zadatka.
6. Sistem kombinuje pojam i izabrane filtere i prikaže suženu listu (samo `public`), uz osvežen brojač rezultata.
7. Korisnik klikne stavku iz liste i otvori **„Detalji resursa"**.
8. Sistem prikaže javne metapodatke resursa (naziv, opis, jezik, licenca, domen, tip zadatka, poreklo, verzija) i, umesto dugmadi za preuzimanje/pokretanje, prikaže **„Prijava za pristup"** poziv na prijavu.

**Alternativni tokovi:**
A1. **Nema rezultata** — u koraku 4 ili 6: kombinacija pojma i/ili filtera ne vraća nijedan javni resurs; sistem prikaže poruku da nema rezultata i predloži uklanjanje dela filtera; korisnik menja kriterijume i tok se vraća na korak 3 ili 5.
A2. **Radnja zahteva prijavu** — u koraku 8: korisnik klikne **„Prijava za pristup"**; sistem ga preusmeri na Keycloak prijavu/registraciju; po prijavi korisnik nastavlja kao prijavljen korisnik (izlazi iz ove tačke).
A3. **Poništavanje filtera** — u koraku 6: korisnik klikne **„Poništi filtere"**; sistem ukloni izabrane vrednosti i vrati punu listu javnih resursa; uneti pojam pretrage se zadržava.
A4. **Federisani resurs** — u koraku 7/8: resurs potiče iz Pharos/IT4LIA/CKAN; **„Detalji resursa"** prikažu oznaku porekla i link ka izvoru (metapodaci mogu biti delimični, koliko partnerski katalog izlaže); izvršne radnje vode na prijavu kao u A2.
A5. **Resurs je zastareo (`deprecated`)** — u koraku 8: sistem prikaže metapodatke sa upozorenjem o zastarelosti i bez poziva na preuzimanje/pokretanje.
A6. **Direktan link ka nejavnom ili nepostojećem resursu** — u koraku 7: korisnik dođe direktnim URL-om do resursa sa vidljivošću `private`/`team`, ili do `archived`/uklonjenog resursa; sistem u svim slučajevima prikaže istovetnu poruku **„Resurs nije pronađen"** (404), bez razlikovanja da li resurs postoji ali je skriven — da se ne oda postojanje nejavnog resursa.

**Rezultat:** Posetiocu je prikazana lista javnih resursa sužena prema unetom pojmu i izabranim filterima, a otvaranjem jedne stavke vidi njene javne metapodatke (model card). Nije kreirana Keycloak sesija; nijedan `private`/`team` resurs nije izlistan ni otvoren; nijedna radnja nad resursima nije izvršena.


**Naziv:** Pretraga i filtriranje kataloga resursa
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen preko Keycloak-a i ima ulogu koja mu daje pristup katalogu; katalog sadrži lokalne i federisane resurse.

**Osnovni tok:**
1. Prijavljen korisnik otvori ekran **„Katalog"**.
2. Sistem prikaže listu resursa vidljivih korisniku prema njegovoj ulozi i ABAC atributima, sa oznakom porekla (lokalni / Pharos / IT4LIA) na svakom federisanom resursu. Resursi za koje korisnik nema pristup detaljima ne prikazuju se u listi.
3. Korisnik unese pojam u polje **„Pretraga"**.
4. Sistem prikaže rezultate koji odgovaraju pojmu, i dalje filtrirane po vidljivosti.
5. Korisnik otvori **„Filteri"** i izabere vrednosti (tip resursa, sektorski tag, jezik, poreklo).
6. Sistem osveži listu prema izabranim filterima i pretrazi.
7. Korisnik izabere resurs sa liste i otvori **„Detalji resursa"**.
8. Sistem prikaže metapodatke resursa (vlasnik, licenca, jezik, domen, poreklo, verzija, status).

**Alternativni tokovi:**
A1. **Nema rezultata** — u koraku 4: pretraga ne vraća nijedan resurs; sistem prikaže prazno stanje sa porukom i predlogom da se ublaže filteri ili izmeni pojam; korisnik se vraća na korak 3 ili 5.
A2. **Federisani izvor nedostupan** — u koraku 2: partnerski katalog (Pharos/IT4LIA) ne odgovara; sistem prikaže samo lokalne resurse uz poruku da federisani izvori trenutno nisu dostupni; tok se nastavlja normalno.
A3. **Poništavanje filtera** — u koraku 6: korisnik odustane od suženja i izabere **„Poništi filtere"**; sistem vrati pun spisak vidljivih resursa; tok se nastavlja od koraka 3.

**Rezultat:** Korisniku je prikazan ekran **„Detalji resursa"** sa metapodacima izabranog resursa, ili filtrirana lista kataloga uklj. oznake porekla; nijedan resurs van korisnikove vidljivosti nije prikazan ni otvoren.


**Naziv:** Objava resursa u katalog uz slanje na pregled
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen preko Keycloak-a i ima dozvolu da objavljuje resurse; resurs (model ili dataset) je već otpremljen u MinIO ili registrovan kao eksterni izvor i nalazi se u stanju **„Draft"**.

**Osnovni tok:**
1. Prijavljen korisnik otvori **„Detalji resursa"** za resurs koji je registrovao i izabere **„Objavi u katalog"**.
2. Sistem prikaže formu **„Objava resursa"** sa poljima za opis, tagove, sektorske tagove, licencu i vidljivost.
3. Korisnik unese opis i izabere tagove i sektorske tagove.
4. Korisnik izabere licencu iz liste **„Licenca"**.
5. Korisnik izabere vidljivost (`private` / `team` / `public`).
6. Korisnik izabere **„Pošalji na pregled"**.
7. Sistem proveri da su obavezna polja popunjena.
8. Sistem postavi resurs u stanje **„Poslat"** (`submitted`), dodeli ga review toku (Data administrator za dataset, Model reviewer za model) i prebaci ga u **„Na pregledu"** (`under review`) kada recenzent preuzme stavku.
9. Sistem prikaže potvrdu da je resurs poslat na pregled i obavesti nadležnog recenzenta.

**Alternativni tokovi:**
A1. **Nedostaju obavezna polja** — u koraku 7: opis, licenca ili tag nisu popunjeni; sistem označi prazna polja i ne šalje resurs; korisnik se vraća na korak 3.
A2. **Resurs već poslat ili objavljen** — u koraku 1: resurs je već u stanju **„Poslat"**, **„Na pregledu"** ili **„Objavljen (lokalno)"**; sistem ne nudi ponovnu objavu, već prikaže trenutni status; tok se prekida.
A3. **Odustajanje** — u koraku 3 ili 5: korisnik napusti formu bez slanja; sistem sačuva uneto i resurs ostaje u stanju **„Draft"**; review se ne pokreće; resurs ostaje nevidljiv u katalogu.
A4. **Resurs vraćen na doradu** — nakon koraka 9: recenzent odbije ili zatraži izmene (vidi Scenario „Pregled i odluka o objavi resursa"); resurs se vraća u **„Draft"**, korisnik dobija notifikaciju sa komentarom i može da ponovi tok od koraka 1.

**Rezultat:** Resurs je u stanju **„Na pregledu"**, dodeljen nadležnom recenzentu (Data administrator ili Model reviewer), sa unetim opisom, tagovima, licencom i vidljivošću; još nije vidljiv u javnom katalogu.


**Naziv:** Pregled i odluka o objavi resursa
**Akter:** Recenzent (Model reviewer za model, Data administrator za dataset)
**Preduslov:** Recenzent je prijavljen preko Keycloak-a i ima dozvolu za pregled resursa odgovarajućeg tipa; u redu za pregled postoji bar jedan resurs u stanju **„Na pregledu"**.

**Osnovni tok:**
1. Recenzent otvori ekran **„Red za pregled"** i izabere resurs iz liste.
2. Sistem prikaže **„Detalji resursa"** sa metapodacima (opis, tagovi, licenca, vidljivost, poreklo, verzija) i pristupom samom artefaktu u MinIO.
3. Recenzent pregleda metapodatke i sadržaj resursa, uključujući ručnu procenu da li je izabrana licenca u skladu sa poreklom resursa (upstream model/dataset). Sistem ne proverava licencu automatski — to je recenzentova odgovornost.
4. Recenzent izabere **„Odobri"**.
5. Sistem zatraži potvrdu i opcioni komentar.
6. Recenzent potvrdi odluku.
7. Sistem postavi resurs u stanje **„Odobren"** (`approved`) i učini ga vidljivim u katalogu prema vidljivosti koju je objavljivač izabrao (`private` / `team` / `public`). Ako je objavljivač izabrao `public`, resurs prelazi u **„Objavljen (lokalno)"** i postaje vidljiv anonimnom posetiocu.
8. Sistem obavesti objavljivača da je resurs odobren i prikaže recenzentu potvrdu.

**Alternativni tokovi:**
A1. **Vraćanje na doradu / odbijanje** — u koraku 4: recenzent izabere **„Vrati na doradu"** (tražene izmene) ili **„Odbij"** (resurs neprihvatljiv); sistem u oba slučaja zahteva obavezan komentar sa razlogom, vraća resurs u stanje **„Draft"** i obaveštava objavljivača sa komentarom; resurs se ne objavljuje; tok se završava. (Razlika između dve opcije je samo semantika poruke objavljivaču — lifecycle posledica je ista: povratak u **„Draft"**.)
> ⚠ Ako se uvede terminalno stanje `rejected` (odbijen bez mogućnosti ponovnog slanja) različito od `draft` (vraćen, može ponovo) — razdvojiti „Odbij" i „Vrati na doradu" u dve grane sa različitim ciljnim stanjem. Trenutno obe vode u `draft`, što za audit ne beleži da je resurs bio odbijen.
A2. **Nekompatibilna licenca ili sporno poreklo** — u koraku 3: recenzent ručno utvrdi da licenca nije u skladu sa poreklom resursa; tretira se kao razlog za odbijanje i nastavlja kao A1.
A3. **Resurs povučen tokom pregleda** — u koraku 3: objavljivač je u međuvremenu povukao resurs ili ga prebacio u **„Draft"**; sistem prikaže da resurs više nije na pregledu i uklanja ga iz reda; recenzent se vraća na korak 1.
A4. **Odustajanje od odluke** — u koraku 5: recenzent napusti ekran bez potvrde; resurs ostaje u stanju **„Na pregledu"** u redu; nijedna promena se ne upisuje.

**Rezultat:** Resurs je u stanju **„Odobren"** (ili **„Objavljen (lokalno)"** ako je vidljivost `public`) i vidljiv u katalogu prema vidljivosti koju je objavljivač izabrao, sa zabeleženom odlukom recenzenta (i opcionim komentarom); objavljivač je obavešten.


**Naziv:** Skidanje javnog resursa sa kataloga (moderacija)
**Akter:** Platform administrator
**Preduslov:** Platform administrator je prijavljen preko Keycloak-a i ima dozvolu za moderaciju kataloga; resurs je trenutno javno vidljiv (vidljivost `public`, u stanju **„Objavljen (lokalno)"** ili **„Objavljen (Pharos)"**).

**Osnovni tok:**
1. Platform administrator otvori ekran **„Katalog"** i izabere javno vidljiv resurs iz liste.
2. Sistem prikaže **„Detalji resursa"** sa trenutnom vidljivošću i statusom.
3. Administrator izabere **„Skini sa javnog kataloga"**.
4. Sistem zatraži obavezan razlog (moderacija mora biti zabeležena u audit log).
5. Administrator unese razlog i potvrdi.
6. Sistem promeni vidljivost resursa sa `public` na `private`, ukloni ga iz javnog dela kataloga (više nije vidljiv anonimnom posetiocu) i zabeleži ko, kada i zašto.
7. Sistem obavesti objavljivača da je resurs skinut sa javnog kataloga, sa unetim razlogom.

**Alternativni tokovi:**
A1. **Resurs nije javan** — u koraku 1: izabrani resurs je već `private`/`team` ili nije objavljen; sistem ne nudi opciju skidanja; tok se prekida.
A2. **Odustajanje** — u koraku 4 ili 5: administrator napusti ekran bez potvrde; nijedna promena se ne upisuje; resurs ostaje javan.
> ⚠ **ODLUKA 5 (otvorena):** Masovno skidanje/podešavanje vidljivosti (selekcija više resursa odjednom, grupna primena, izveštaj o preskočenim resursima) je zasebna funkcionalnost sa sopstvenim ekranom i logikom, prevelika za alternativnu granu. Nije uključena ovde; ako je potrebna u MVP-u, modelovati je kao zaseban use-case „Masovno podešavanje vidljivosti kataloga".

**Rezultat:** Resurs je skinut sa javnog kataloga (vidljivost `private`), više nije vidljiv anonimnom posetiocu, a razlog skidanja je zabeležen u audit log; objavljivač je obavešten.
