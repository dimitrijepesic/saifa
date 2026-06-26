## 1. Pristup platformi i nalozi


**Naziv:** Registracija lokalnim nalogom
**Akter:** Anonimni posetilac
**Preduslov:** Korisnik nije prijavljen; ima pristup SAIFA portalu i važeću email adresu.

**Osnovni tok:**
1. Anonimni posetilac otvori SAIFA portal i klikne dugme **„Prijavi se"** u zaglavlju.
2. Keycloak login stranica se prikaže; korisnik klikne link **„Registruj se"** ispod forme za prijavu.
3. Korisnik popuni registracioni formular: ime, prezime, email adresa, lozinka, potvrda lozinke; prihvati uslove korišćenja kvačicom i klikne **„Kreiraj nalog"**.
4. Sistem validira unos i šalje email sa vezom za potvrdu na unetu adresu.
5. Korisnik otvori email i klikne vezu za potvrdu.
6. Portal prikaže poruku o uspešnoj potvrdi i automatski prijavi korisnika.
7. Korisnik se preusmeri na početnu stranicu portala sa ulogom **Nezavisni korisnik** (bez organizacije, bez kvote dok je administrator ne dodeli).

**Alternativni tokovi:**
A1. **Email već postoji** — u koraku 4: sistem prikaže grešku „Nalog sa ovom email adresom već postoji"; korisnik se vraća na formular i može kliknuti **„Zaboravljena lozinka"** ili uneti drugu adresu.
A2. **Lozinka ne zadovoljava zahteve** — u koraku 4: sistem prikaže inline grešku ispod polja (npr. „Lozinka mora imati najmanje 8 karaktera i jedan specijalni znak"); korisnik ispravlja lozinku bez gubitka ostalih unosa.
A3. **Veza za potvrdu istekla** — u koraku 5: portal prikaže poruku „Veza je istekla"; korisnik klikne **„Pošalji ponovo"** i dobija novi email; nalog ostaje neaktivan do uspešne potvrde.
A4. **Korisnik odustane** — u bilo kom koraku pre slanja formulara: zatvori stranicu; nalog nije kreiran.
A5. **Institucijski korisnik pokušava lokalnu registraciju** — u koraku 3: korisnik unese email institucionalnog domena (npr. `@etf.bg.ac.rs`); sistem prikaže obaveštenje „Za pristup putem vaše institucije koristite SSO prijavu" i ponudi link ka eduGAIN/AMRES opciji.

**Rezultat:** U sistemu postoji aktivan lokalni nalog sa ulogom Nezavisni korisnik; korisnik je prijavljen i može pretraživati katalog i koristiti platformu u okviru podrazumevane (ili nulte) kvote dok je platform administrator ne prilagodi.



**Naziv:** Prijava institucionalnim SSO-om (eduGAIN/AMRES)
**Akter:** Anonimni posetilac
**Preduslov:** Korisnik nije prijavljen; njegova matična institucija je član AMRES/eduGAIN federacije i ima aktivan institucionalni nalog.

**Osnovni tok:**
1. Anonimni posetilac otvori SAIFA portal i klikne **„Prijavi se"** u zaglavlju.
2. Keycloak login stranica se prikaže; korisnik klikne opciju **„Prijava putem institucije (eduGAIN/AMRES)"**.
3. Prikaže se pretraživač institucija (Discovery Service); korisnik ukuca naziv ili odabere svoju instituciju sa liste i klikne **„Nastavi"**.
4. Portal preusmeri korisnika na login stranicu matične institucije (IdP); korisnik unese institucionalne kredencijale (korisnički nalog + lozinka po pravilima institucije) i potvrdi prijavu.
5. Institucija autentifikuje korisnika i šalje SAML/OIDC assertion ka Keycloak-u sa atributima (email, ime, prezime, afilijacija).
6. Keycloak prima assertion i proverava atribute: ako nalog sa tim emailom ne postoji, kreira ga automatski i dodeljuje ulogu **Prijavljen korisnik / akademski** pod odgovarajućom organizacijom (ako je institucija već registrovana na platformi) ili kao nezavisni akademski korisnik.
7. Korisnik se preusmeri na početnu stranicu portala, prijavljen sa pogledom prilagođenim svojoj ulozi.

**Alternativni tokovi:**
A1. **Institucija nije u federaciji** — u koraku 3: korisnik ne pronađe svoju instituciju na listi; prikaže se poruka „Vaša institucija nije dostupna putem AMRES/eduGAIN — registrujte se lokalnim nalogom"; korisnik dobija link ka lokalnoj registraciji.
A2. **Autentifikacija kod institucije ne uspe** — u koraku 4: institucijski IdP odbije kredencijale i prikaže svoju grešku; korisnik ostaje na stranici institucije; može pokušati ponovo ili se vratiti na SAIFA portal klikom „Nazad".
A3. **IdP ne vrati obavezne atribute** — u koraku 5: Keycloak ne dobije email ili ime; portal prikaže poruku „Vaša institucija nije prosledila sve potrebne podatke — obratite se IT podršci institucije"; sesija se ne uspostavlja.
A4. **Email već postoji kao lokalni nalog** — u koraku 6: Keycloak detektuje koliziju; portal prikaže obaveštenje „Nalog sa ovom adresom već postoji kao lokalni nalog — prijavite se lozinkom ili kontaktirajte administratora za spajanje naloga"; SSO prijava se ne završava.
A5. **Korisnik odustane na stranici institucije** — u koraku 4: korisnik zatvori prozor ili klikne „Otkaži"; Keycloak primi grešku otkazivanja; portal vrati korisnika na login ekran bez poruke o grešci.

**Rezultat:** Korisnik je prijavljen sa nalogom vezanim za institucijski identitet; atributi (institucija, afilijacija) su upisani u profil i koriste se za ABAC politike; korisnik može pristupiti punom katalogu i funkcijama platforme u okviru svoje uloge i kvote organizacije.



**Naziv:** Prijava EuroHPC identitetom
**Akter:** Anonimni posetilac
**Preduslov:** Korisnik nije prijavljen; ima aktivan MyAccessID nalog registrovan na EuroHPC korisničkom portalu.

**Osnovni tok:**
1. Anonimni posetilac otvori SAIFA portal i klikne **„Prijavi se"** u zaglavlju.
2. Keycloak login stranica se prikaže; korisnik klikne opciju **„Prijava putem EuroHPC"**.
3. Portal preusmeri korisnika na EuroHPC login stranicu; korisnik unese svoje EuroHPC kredencijale i potvrdi prijavu.
4. MyAccessID autentifikuje korisnika i šalje OIDC token ka Keycloak-u sa atributima (email, ime, prezime, EuroHPC membership status).
5. Keycloak prima token i proverava atribute: ako nalog sa tim emailom ne postoji, kreira ga i dodeljuje ulogu **Prijavljen korisnik** sa oznakom federisanog identiteta (`euroHPC`).
6. Korisnik se preusmeri na početnu stranicu portala, prijavljen sa pogledom koji uključuje federisane resurse (modeli i datasetovi sa Pharos/IT4LIA platformi vidljivi su u katalogu sa oznakom porekla).

**Alternativni tokovi:**
A1. **Korisnik nema MyAccessID nalog** — u koraku 3: korisnik klikne link **„Registruj se na EuroHPC portalu"** koji ga vodi na `eurohpc-ju.eu`; SAIFA sesija se ne uspostavlja; korisnik se vraća na SAIFA login tek nakon što kreira nalog.
A2. **Autentifikacija na MyAccessID ne uspe** — u koraku 3: MyAccessID odbije kredencijale i prikaže svoju grešku; korisnik ostaje na MyAccessID stranici; može pokušati ponovo ili se vratiti na SAIFA portal.
A3. **MyAccessID ne vrati obavezne atribute** — u koraku 4: Keycloak ne dobije email ili membership status; portal prikaže poruku „EuroHPC nije prosledilo sve potrebne podatke — proverite da li je vaš MyAccessID profil potpun"; sesija se ne uspostavlja.
A4. **Email već postoji kao lokalni ili eduGAIN nalog** — u koraku 5: Keycloak detektuje koliziju; portal prikaže obaveštenje „Nalog sa ovom adresom već postoji — prijavite se originalnom metodom ili kontaktirajte administratora za spajanje naloga"; EuroHPC prijava se ne završava.
A5. **Korisnik odustane na MyAccessID stranici** — u koraku 3: korisnik klikne „Otkaži" ili zatvori prozor; Keycloak primi grešku otkazivanja; portal vrati korisnika na login ekran bez poruke o grešci.

**Rezultat:** Korisnik je prijavljen sa nalogom vezanim za EuroHPC identitet; u profilu je upisana oznaka `euroHPC`; korisnik vidi federisane resurse u katalogu (označene oznakom porekla) i može im pristupiti bez posebnog naloga na partnerskoj platformi, kroz Keycloak identity brokering.



**Naziv:** Kreiranje i upravljanje API ključem
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen preko Keycloak-a i ima ulogu sa pravom na programski pristup (inference i/ili pokretanje poslova).

**Osnovni tok:**
1. Prijavljen korisnik otvori **„Podešavanja"** i izabere karticu **„API ključevi"**.
2. Sistem prikaže listu postojećih ključeva korisnika (naziv, scope, datum izdavanja, rok važenja, status).
3. Korisnik klikne **„Novi ključ"**.
4. Sistem prikaže formu; korisnik unese naziv ključa, izabere scope (npr. inference / pokretanje poslova) i rok važenja.
5. Korisnik klikne **„Izdaj ključ"**.
6. Sistem generiše ključ, prikaže ga jednom u celosti i ponudi dugme **„Kopiraj"**; korisnik kopira ključ i sačuva ga na bezbedno mesto.

**Alternativni tokovi:**
A1. **Ključ sa istim nazivom već postoji i aktivan je** — u koraku 5: sistem upozori da već postoji aktivan ključ pod tim nazivom i predloži drugi naziv ili korišćenje postojećeg ključa; novi ključ se ne izdaje.
A2. **Korisnik rotira ključ** — u koraku 2: korisnik izabere postojeći ključ i klikne **„Rotiraj"**; sistem izda novu vrednost ključa, poništi staru i prikaže novu vrednost jednom; pozivi sa starom vrednošću prestaju da važe.
A3. **Korisnik opoziva ključ** — u koraku 2: korisnik izabere ključ i klikne **„Opozovi"**; sistem zatraži potvrdu i trajno onemogući ključ; svi budući pozivi sa tim ključem vraćaju grešku autentikacije.
A4. **Odustajanje** — u koraku 4 ili 5: korisnik napusti formu; nijedan ključ se ne izdaje.

**Rezultat:** Korisnik ima aktivan API ključ za programski pristup platformi (REST/SDK), sa izabranim scope-om i rokom važenja; izdavanje, rotacija i opoziv ključa zabeleženi su u audit logu.



**Naziv:** Registracija organizacije i zahtev za odobrenje
**Akter:** Predstavnik organizacije
**Preduslov:** Predstavnik ima aktivan nalog na platformi (lokalni, eduGAIN ili MyAccessID); nije još uvek pridružen nijednoj organizaciji; zna zvanični naziv, tip i sektor organizacije.

**Osnovni tok:**
1. Predstavnik se prijavi na SAIFA portal i na početnoj stranici ili u meniju naloga klikne **„Registruj organizaciju"**.
2. Prikaže se formular za registraciju; predstavnik popuni obavezna polja: naziv organizacije, tip (akademska institucija / istraživačka institucija / kompanija/SME), sektor (zdravstvo / poljoprivreda / jezik i kultura / održivost / opšte), kontakt email organizacije i kratak opis svrhe korišćenja platforme.
3. Opcionalno: predstavnik priloži dokumentaciju (rešenje o registraciji, pismo institucije i sl.) klikom na **„Priloži dokument"**.
4. Predstavnik prihvati uslove korišćenja za organizacije kvačicom i klikne **„Pošalji zahtev"**.
5. Sistem validira unos, kreira organizaciju u statusu **`Na čekanju`** i automatski šalje notifikaciju platform administratoru o novom zahtevu.
6. Portal prikaže potvrdu: „Vaš zahtev je primljen i čeka odobrenje administratora. Obaveštenje ćete dobiti emailom."
7. Predstavnik se vraća na početnu stranicu; u zaglavlju ili bočnom meniju vidi oznaku **„Organizacija: na čekanju"**.

**Alternativni tokovi:**
A1. **Organizacija sa istim nazivom već postoji** — u koraku 5: sistem prikaže upozorenje „Organizacija s ovim nazivom već je registrovana — ako ste član te organizacije, obratite se njenom predstavniku ili platform administratoru"; formular ostaje popunjen, zahtev se ne šalje.
A2. **Obavezno polje nije popunjeno** — u koraku 5: sistem prikaže inline grešku pored praznog polja; predstavnik ispravlja unos bez gubitka ostatka formulara.
A3. **Predstavnik već pripada organizaciji** — u koraku 1: sistem detektuje da nalog ima aktivnu organizacijsku vezu i prikaže poruku „Već ste pridruženi organizaciji [naziv] — ne možete registrovati novu dok god ste njen član"; dugme **„Registruj organizaciju"** nije dostupno.
A4. **Korisnik odustane** — u bilo kom koraku pre slanja: klikne „Otkaži" ili napusti stranicu; formular se ne čuva, zahtev se ne kreira.

**Rezultat:** U sistemu postoji organizacija u statusu **`Na čekanju`** sa svim unetim podacima i priloženom dokumentacijom; platform administrator ima notifikaciju o zahtevu i može pristupiti pregledu i odlučivanju; predstavnik čeka emailom potvrdu o ishodu.



**Naziv:** Dodavanje člana u organizaciju
**Akter:** Predstavnik organizacije
**Preduslov:** Predstavnik je prijavljen i ima aktivnu organizaciju; osoba koja se dodaje ima nalog na platformi; organizacija ima slobodnog kapaciteta u okviru dodeljene kvote.

**Osnovni tok:**
1. Predstavnik organizacije otvori portal i u bočnom meniju klikne **„Moja organizacija"** → **„Članovi"**.
2. Prikaže se lista trenutnih članova sa njihovim ulogama i potrošnjom kvote; predstavnik klikne **„Dodaj člana"**.
3. Prikaže se pretraga; predstavnik ukuca ime ili email adresu osobe i klikne **„Pretraži"**.
4. Sistem prikaže rezultat; predstavnik potvrdi da je pronašao pravu osobu i klikne **„Dodaj"** pored njenog imena.
5. Prikaže se dijalog za konfiguraciju: predstavnik odabere ulogu novog člana unutar organizacije (Student / Akademski istraživač / Mentor / Industrijski korisnik / ML praktičar) i opcionalno odmah dodeli deo kvote (CPU sati, GPU sati, prostor).
6. Predstavnik klikne **„Potvrdi"**.
7. Sistem doda osobu u organizaciju, primeni odabranu ulogu i kvotu, i pošalje joj email obaveštenje da je dodata u organizaciju [naziv] sa navedenom ulogom.
8. Lista članova se osvežava; nova osoba je vidljiva sa dodeljenom ulogom i kvotom.

**Alternativni tokovi:**
A1. **Osoba nema nalog na platformi** — u koraku 4: pretraga ne vrati rezultate; predstavnik klikne **„Pozovi osobu"**, unese email adresu i pošalje pozivnicu; sistem šalje email sa linkom za registraciju; osoba se dodaje u organizaciju automatski nakon što kreira nalog i prihvati poziv.
A2. **Osoba je već član druge organizacije** — u koraku 4: pored rezultata pretrage prikaže se oznaka „Već u organizaciji [naziv]"; predstavnik ne može dodati tu osobu; ako je potrebno, mora se obratiti platform administratoru za transfer.
A3. **Organizacija je dostigla limit kvote** — u koraku 2: dugme **„Dodaj člana"** prikazuje se zasivljeno sa napomenom „Dostignut limit članova — zatražite povećanje kvote od administratora"; predstavnik ne može dodati novog člana dok kvota ne bude proširena.
A4. **Predstavnik ne dodeljuje kvotu odmah** — u koraku 5: ostavi polja kvote prazna i klikne **„Potvrdi"**; član se dodaje bez dodeljene kvote i ne može pokretati poslove dok mu predstavnik naknadno ne dodeli resurse putem **„Uredi kvotu"** na stranici člana.
A5. **Korisnik odustane** — u bilo kom koraku pre potvrde: klikne „Otkaži"; lista članova ostaje nepromenjena.

**Rezultat:** Nova osoba je član organizacije sa dodeljenom ulogom i (opciono) kvotom; može odmah pristupiti resursima i pokretati poslove u okviru dodeljenih resursa; dodavanje je zabeleženo u audit logu sa identitetom predstavnika i vremenskom oznakom.



**Naziv:** Uklanjanje člana iz organizacije
**Akter:** Predstavnik organizacije
**Preduslov:** Predstavnik je prijavljen i ima aktivnu organizaciju; osoba koja se uklanja je trenutni član te organizacije.

**Osnovni tok:**
1. Predstavnik organizacije otvori portal i u bočnom meniju klikne **„Moja organizacija"** → **„Članovi"**.
2. Prikaže se lista trenutnih članova sa ulogama i potrošnjom kvote; predstavnik pronađe željenu osobu i klikne **„Uredi"** pored njenog imena.
3. Prikaže se stranica detalja člana sa pregledom dodeljene kvote, aktivnih poslova i resursa u vlasništvu; predstavnik klikne **„Ukloni iz organizacije"**.
4. Prikaže se dijalog za potvrdu koji prikazuje: ime člana, broj aktivnih poslova (ako postoje) i napomenu šta se dešava s resursima nakon uklanjanja; predstavnik klikne **„Potvrdi uklanjanje"**.
5. Sistem ukloni osobu iz organizacije, povuče dodeljenu kvotu, prebaci njen nalog u status Nezavisni korisnik i pošalje joj email obaveštenje da je uklonjena iz organizacije [naziv].
6. Lista članova se osvežava; uklonjena osoba više nije vidljiva na listi.

**Alternativni tokovi:**
A1. **Član ima aktivne pokrenute AI factory poslove** — u koraku 4: dijalog eksplicitno navede broj aktivnih poslova i upozori „Pokrenuti poslovi će nastaviti do završetka, ali novi neće moći da se pokreću"; predstavnik potvrđuje ili odustaje.
A2. **Član je vlasnik zajedničkih resursa tima** — u koraku 4: dijalog prikaže upozorenje „Ovaj korisnik je vlasnik [N] resursa deljenih sa organizacijom — resursi ostaju u katalogu, ali vlasništvo prelazi na vas kao predstavnika"; predstavnik potvrđuje ili odustaje.
A3. **Predstavnik pokušava ukloniti samog sebe** — u koraku 2: opcija **„Ukloni iz organizacije"** nije dostupna pored sopstvenog imena; predstavnik mora zatražiti od platform administratora da promeni predstavnika pre nego što napusti organizaciju.
A4. **Predstavnik odustane** — u koraku 4: klikne „Otkaži" u dijalogu; lista članova ostaje nepromenjena.

**Rezultat:** Osoba više nije član organizacije; njena dodeljena kvota je vraćena u pool organizacije; nalog joj ostaje aktivan kao Nezavisni korisnik i može se prijavljivati na platformu; uklanjanje je zabeleženo u audit logu sa identitetom predstavnika i vremenskom oznakom.



**Naziv:** Odobravanje ili odbijanje registracije organizacije
**Akter:** Platform administrator
**Preduslov:** Postoji najmanje jedna organizacija u statusu **`Na čekanju`**; platform administrator je prijavljen.

**Osnovni tok:**
1. Platform administrator otvori administratorski panel i u bočnom meniju klikne **„Organizacije"** → **„Na čekanju"**.
2. Prikaže se lista zahteva za registraciju; administrator klikne na naziv organizacije da otvori njen detaljan pregled.
3. Pregled prikazuje: naziv, tip, sektor, kontakt email, opis svrhe korišćenja, priloženu dokumentaciju i podatke o nalogu predstavnika koji je podneo zahtev.
4. Administrator pregleda priloženu dokumentaciju klikom na **„Preuzmi dokument"** (ako postoji).
5. Administrator klikne **„Odobri"**; prikaže se dijalog za potvrdu koji traži dodelu početne ukupne kvote organizaciji (CPU sati, GPU sati, prostor); administrator unese vrednosti i klikne **„Potvrdi"**.
6. Sistem postavi status organizacije na **`Aktivna`**, dodeli kvotu, poveže nalog predstavnika sa ulogom Predstavnik organizacije i pošalje mu email o odobrenju.
7. Administrator se vrati na listu; organizacija više ne figuriše u redu **„Na čekanju"**.

**Alternativni tokovi:**
A1. **Administrator odbija zahtev** — u koraku 5: administrator klikne **„Odbij"** umesto **„Odobri"**; prikaže se dijalog koji traži obavezan razlog odbijanja; administrator unese obrazloženje i klikne **„Potvrdi odbijanje"**; sistem postavi status organizacije na **`Odbijeno`**, pošalje predstavniku email sa razlogom; organizacija ostaje u sistemu kao neaktivna i vidljiva administratoru u arhivi.
A2. **Administrator traži dopunu dokumentacije** — u koraku 4: administrator klikne **„Traži dopunu"**, unese komentar šta nedostaje i klikne **„Pošalji"**; sistem obavesti predstavnika emailom; zahtev ostaje u statusu **`Na čekanju — tražena dopuna`** dok predstavnik ne pošalje izmenu.
A3. **Administrator odustane** — u bilo kom koraku pre potvrde: klikne „Otkaži" u dijalogu ili napusti stranicu; status organizacije ostaje **`Na čekanju`**, nikakva akcija se ne beleži.
A4. **Lista zahteva je prazna** — u koraku 1: sistem prikaže poruku „Nema zahteva na čekanju"; administrator nema šta da pregleda.

**Rezultat:** Organizacija je aktivna u sistemu sa dodeljenom ukupnom kvotom; predstavnik organizacije ima odgovarajuću ulogu i može početi da dodaje članove i raspodeljuje kvotu; ishod (odobrenje ili odbijanje) je zabeležen u audit logu.



**Naziv:** Dodeljivanje uloge predstavnika organizacije
**Akter:** Platform administrator
**Preduslov:** Organizacija je aktivna u sistemu; postoji nalog osobe kojoj se dodeljuje uloga i ta osoba je član te organizacije (ili nema još uvek organizacijsku vezu); organizacija trenutno nema predstavnika, ili se vrši zamena postojećeg.

**Osnovni tok:**
1. Platform administrator otvori administratorski panel i u bočnom meniju klikne **„Organizacije"** → **„Aktivne"**.
2. Administrator pronađe željenu organizaciju u listi i klikne na njen naziv da otvori stranicu sa detaljima.
3. Na stranici organizacije administrator klikne karticu **„Članovi"**; prikaže se lista članova sa njihovim ulogama.
4. Administrator pronađe željenu osobu u listi i klikne **„Uredi ulogu"** pored njenog imena.
5. U padajućem meniju uloga odabere **„Predstavnik organizacije"** i klikne **„Sačuvaj"**.
6. Prikaže se dijalog za potvrdu: „Da li ste sigurni? Ova osoba preuzima punu upravljačku ulogu nad organizacijom [naziv]."; administrator klikne **„Potvrdi"**.
7. Sistem dodeli ulogu, automatski ukloni ulogu Predstavnika od prethodnog nosioca (ako je postojao) i pošalje email obaveštenje i novom i starom predstavniku.
8. Stranica organizacije se osvežava; pored imena osobe prikazuje se oznaka **„Predstavnik"**.

**Alternativni tokovi:**
A1. **Osoba nije član organizacije** — u koraku 3: administrator ne pronalazi željenu osobu na listi članova; klikne **„Dodaj člana"**, pretražuje po emailu ili imenu, odabere osobu i u istom toku direktno dodeli ulogu Predstavnika (bez prethodnog dodavanja kao obični član).
A2. **Osoba je već predstavnik druge organizacije** — u koraku 5: sistem prikaže upozorenje „Ova osoba je već predstavnik organizacije [naziv druge org.] — jedna osoba ne može biti predstavnik više organizacija"; akcija se blokira.
A3. **Osoba nema nalog na platformi** — u koraku 4: administrator ne pronađe osobu u listi; može kliknuti **„Pozovi osobu"** i uneti email; sistem šalje pozivnicu; uloga se može dodeliti tek nakon što osoba kreira nalog i prihvati poziv.
A4. **Administrator odustane** — u koraku 6: klikne „Otkaži" u dijalogu za potvrdu; uloge ostaju nepromenjene.

**Rezultat:** Odabrana osoba ima ulogu Predstavnika organizacije i može odmah raspodeljuje kvotu i upravljati članovima; promena uloge je zabeležena u audit logu sa vremenskom oznakom i identitetom administratora koji je izvršio izmenu.



**Naziv:** Kreiranje naloga od strane administratora
**Akter:** Platform administrator
**Preduslov:** Platform administrator je prijavljen; zna email adresu i osnovne podatke osobe za koju kreira nalog.

**Osnovni tok:**
1. Platform administrator otvori administratorski panel i u bočnom meniju klikne **„Korisnici"** → **„Svi korisnici"**.
2. Administrator klikne dugme **„Novi korisnik"** u gornjem desnom uglu.
3. Prikaže se formular; administrator popuni obavezna polja: ime, prezime, email adresa, i odabere tip naloga iz padajućeg menija (**lokalni nalog / eduGAIN / MyAccessID**).
4. Administrator odabere početnu ulogu korisnika (Prijavljen korisnik / Student / Mentor / Model reviewer / Data administrator) i opcionalno ga odmah pridruži postojećoj organizaciji iz padajuće liste.
5. Administrator klikne **„Kreiraj nalog"**.
6. Sistem kreira nalog, generiše privremenu lozinku (za lokalni nalog) i šalje korisniku email sa uputstvima za prvu prijavu i linkom za postavljanje lozinke.
7. Portal prikaže potvrdu „Nalog je uspešno kreiran" i vrati administratora na stranicu detalja novog korisnika, gde su vidljivi dodeljeni atributi i uloga.

**Alternativni tokovi:**
A1. **Email već postoji u sistemu** — u koraku 5: sistem prikaže grešku „Nalog sa ovom email adresom već postoji"; formular ostaje popunjen; administrator može pretražiti postojeći nalog ili uneti drugu adresu.
A2. **Obavezno polje nije popunjeno** — u koraku 5: sistem prikaže inline grešku pored praznog polja; administrator ispravlja unos bez gubitka ostatka formulara.
A3. **Odabrana organizacija nema slobodnih mesta u kvoti** — u koraku 4: sistem prikaže upozorenje pored padajuće liste organizacija „Ova organizacija je dostigla limit članova u okviru dodeljene kvote"; administrator može nastaviti bez pridruživanja organizaciji ili prethodno povećati kvotu organizacije.
A4. **Administrator odustane** — u bilo kom koraku pre slanja: klikne „Otkaži" ili napusti stranicu; nalog se ne kreira.

**Rezultat:** Nov nalog postoji u sistemu sa dodeljenom ulogom i atributima; korisnik je dobio email sa uputstvima za prvu prijavu; kreiranje naloga je zabeleženo u audit logu sa identitetom administratora i vremenskom oznakom.



**Naziv:** Uređivanje naloga od strane administratora
**Akter:** Platform administrator
**Preduslov:** Platform administrator je prijavljen; nalog koji se uređuje postoji i aktivan je u sistemu.

**Osnovni tok:**
1. Platform administrator otvori administratorski panel i u bočnom meniju klikne **„Korisnici"** → **„Svi korisnici"**.
2. Administrator pronađe nalog pretragom po imenu ili email adresi i klikne na njega.
3. Prikaže se stranica detalja korisnika sa svim trenutnim podacima: ime, prezime, email, uloga, organizacija, ABAC atributi, status naloga.
4. Administrator klikne **„Uredi"** u gornjem desnom uglu stranice.
5. Formular postane editabilan; administrator izmeni željene podatke — ime/prezime, email, ulogu (iz padajućeg menija), organizacijsku vezu ili ABAC atribute.
6. Administrator klikne **„Sačuvaj izmene"**.
7. Sistem validira unos, primeni promene i prikaže potvrdu „Nalog je uspešno ažuriran"; stranica se osvežava sa novim vrednostima.
8. Ako je promenjena uloga ili organizacija, sistem automatski ažurira korisnikov prikaz i dozvole pri sledećoj prijavi; korisnik dobija email obaveštenje o izmeni naloga.

**Alternativni tokovi:**
A1. **Nova email adresa već postoji u sistemu** — u koraku 6: sistem prikaže grešku „Nalog sa ovom email adresom već postoji"; formular ostaje popunjen sa unetim vrednostima; administrator ispravlja adresu ili odustaje.
A2. **Obavezno polje je obrisano** — u koraku 6: sistem prikaže inline grešku pored praznog obaveznog polja; izmene se ne čuvaju dok polje nije popunjeno.
A3. **Promena uloge utiče na aktivne resurse** — u koraku 6: sistem detektuje da korisnik ima pokrenute poslove ili aktivne endpointe koji zahtevaju višu ulogu od nove; prikaže se upozorenje „Promena uloge može uticati na aktivne resurse korisnika — da li želite da nastavite?"; administrator bira **„Nastavi"** ili **„Otkaži"**.
A4. **Administrator odustane** — u bilo kom koraku pre čuvanja: klikne **„Otkaži"** ili napusti stranicu; sve izmene se odbacuju, nalog ostaje nepromenjen.

**Rezultat:** Nalog je ažuriran sa novim vrednostima; izmena je zabeležena u audit logu sa identitetom administratora, vremenskom oznakom i listom promenjenih polja (stara i nova vrednost).



**Naziv:** Deaktivacija naloga od strane administratora
**Akter:** Platform administrator
**Preduslov:** Platform administrator je prijavljen; nalog koji se deaktivira postoji i trenutno je aktivan.

**Osnovni tok:**
1. Platform administrator otvori administratorski panel i u bočnom meniju klikne **„Korisnici"** → **„Svi korisnici"**.
2. Administrator pronađe nalog pretragom po imenu ili email adresi i klikne na njega.
3. Prikaže se stranica detalja korisnika; administrator pregleda trenutni status, organizaciju i aktivne resurse korisnika (aktivni poslovi, endpointi, sesije).
4. Administrator klikne **„Deaktiviraj nalog"** u gornjem desnom uglu stranice.
5. Prikaže se dijalog za potvrdu koji prikazuje upozorenje o aktivnim resursima korisnika (ako postoje) i traži unos razloga deaktivacije; administrator unese razlog i klikne **„Potvrdi deaktivaciju"**.
6. Sistem postavi status naloga na **`Neaktivan`**, trenutno prisilno prekine sve aktivne sesije korisnika, zaustavi pokrenute poslove i suspenduje aktivne inference endpointe.
7. Korisnik dobija email obaveštenje da je njegov nalog deaktiviran, sa navedenim razlogom.
8. Stranica detalja korisnika se osvežava; pored imena prikazuje se oznaka **`Neaktivan`**.

**Alternativni tokovi:**
A1. **Korisnik je jedini predstavnik aktivne organizacije** — u koraku 5: sistem prikaže blokirajuće upozorenje „Ovaj korisnik je jedini predstavnik organizacije [naziv] — deaktivacija nije moguća dok organizacija nema drugog predstavnika"; administrator mora prvo dodeliti novog predstavnika, pa tek onda deaktivirati nalog.
A2. **Korisnik ima pokrenute AI factory poslove** — u koraku 5: dijalog eksplicitno navede broj aktivnih poslova i upozori da će biti zaustavljeni; administrator odlučuje da nastavi ili odustane.
A3. **Administrator odustane** — u koraku 5: klikne „Otkaži" u dijalogu; nalog ostaje aktivan, nikakva akcija se ne beleži.
A4. **Nalog je već neaktivan** — u koraku 3: dugme **„Deaktiviraj nalog"** nije prikazano; umesto toga prikazuje se **„Aktiviraj nalog"** i datum prethodne deaktivacije.

**Rezultat:** Nalog je neaktivan; korisnik se ne može prijaviti niti koristiti platformu; njegovi resursi (modeli, datasetovi, rezultati poslova) ostaju sačuvani u sistemu i nisu obrisani; deaktivacija je zabeležena u audit logu sa identitetom administratora, vremenskom oznakom i razlogom.



**Naziv:** Dodeljivanje uloga i ABAC atributa korisniku
**Akter:** Platform administrator
**Preduslov:** Platform administrator je prijavljen; nalog korisnika kome se dodeljuju uloga i atributi postoji i aktivan je u sistemu.

**Osnovni tok:**
1. Platform administrator otvori administratorski panel i u bočnom meniju klikne **„Korisnici"** → **„Svi korisnici"**.
2. Administrator pronađe korisnika pretragom po imenu ili email adresi i klikne na njega.
3. Prikaže se stranica detalja korisnika; administrator klikne karticu **„Uloge i atributi"**.
4. Administrator klikne **„Uredi"**; prikaže se formular sa dva odeljka: **Uloga** i **ABAC atributi**.
5. U odeljku **Uloga** administrator odabere jednu ulogu iz padajućeg menija (Prijavljen korisnik / Student / Mentor / Model reviewer / Data administrator / Platform administrator).
6. U odeljku **ABAC atributi** administrator postavi vrednosti za svaki atribut iz ponuđenih padajućih menija:
   - **Tip institucije:** akademska / kompanija/SME / istraživačka
   - **Sektor:** zdravstvo / poljoprivreda / jezik i kultura / održivost / opšte
   - **Osetljivost podataka:** javno / interno / osetljivo (compute-to-data)
7. Administrator klikne **„Sačuvaj"**.
8. Sistem primeni novu ulogu i atribute; prikaže potvrdu „Uloga i atributi su uspešno ažurirani"; promene stupaju na snagu pri sledećem zahtevu korisnika (bez potrebe za ponovnom prijavom).

**Alternativni tokovi:**
A1. **Konflikt između uloge i atributa** — u koraku 7: sistem detektuje nekompatibilnu kombinaciju (npr. uloga Student uz atribut tip institucije = kompanija/SME); prikaže upozorenje „Odabrana kombinacija uloge i atributa nije uobičajena — da li želite da nastavite?"; administrator potvrđuje ili ispravlja unos.
A2. **Atribut osetljivost = compute-to-data, a korisnik nema organizaciju** — u koraku 6: sistem prikaže inline upozorenje „Compute-to-data pristup zahteva da korisnik bude član registrovane istraživačke ili kliničke organizacije"; administrator može nastaviti ali se atribut neće aktivirati dok korisnik ne bude pridružen odgovarajućoj organizaciji.
A3. **Administrator uklanja ulogu višeg nivoa dok korisnik ima aktivne resurse** — u koraku 7: sistem prikaže upozorenje sa listom resursa koji zahtevaju višu ulogu (npr. aktivni inference endpointi rezervisani za ML praktičara); administrator bira **„Nastavi"** ili **„Otkaži"**.
A4. **Administrator odustane** — u bilo kom koraku pre čuvanja: klikne **„Otkaži"**; uloga i atributi ostaju nepromenjeni.

**Rezultat:** Korisnik ima ažuriranu ulogu i ABAC atribute; sistem primenjuje nova pravila pristupa na sve buduće zahteve tog korisnika (pristup katalogu, dozvoljena osetljivost podataka, sektorski resursi); izmena je zabeležena u audit logu sa starim i novim vrednostima, identitetom administratora i vremenskom oznakom.



**Naziv:** Prisilno opozivanje sesije korisnika pri bezbednosnom incidentu
**Akter:** Platform administrator
**Preduslov:** Platform administrator je prijavljen; korisnik čija se sesija opoziva ima aktivnu sesiju na platformi; postoji bezbednosni razlog za hitno prekidanje pristupa (sumnja na kompromitovan nalog, neovlašćen pristup, neuobičajena aktivnost).

**Osnovni tok:**
1. Platform administrator otvori administratorski panel i u bočnom meniju klikne **„Korisnici"** → **„Aktivne sesije"**.
2. Prikaže se lista svih korisnika sa trenutno aktivnim sesijama, sa kolonama: korisnik, IP adresa, vreme prijave, poslednja aktivnost, broj aktivnih poslova.
3. Administrator pronađe korisnika o kome je reč (pretragom po imenu ili direktnim pregledom liste) i klikne **„Detalji sesije"** pored njega.
4. Prikaže se pregled sesije: IP adresa, lokacija, uređaj, trajanje, lista aktivnih resursa (pokrenuti poslovi, otvorene sveske, aktivni endpointi).
5. Administrator klikne **„Prisilno prekini sesiju"**.
6. Prikaže se dijalog koji traži: izbor tipa akcije (**samo prekid sesije** / **prekid sesije + privremena blokada naloga**) i unos razloga; administrator popuni i klikne **„Potvrdi"**.
7. Sistem trenutno poništi sve Keycloak tokene korisnika, zatvori aktivne sveske i suspenduje aktivne inference endpointe; AI factory poslovi koji su već u izvršavanju nastavljaju do završetka (osim ako administrator nije odabrao opciju prisilnog zaustavljanja poslova u dijalogu).
8. Korisnik pri sledećem zahtevu dobija poruku „Vaša sesija je istekla — prijavite se ponovo"; ako je nalog privremeno blokiran, prijava nije moguća.
9. Portal prikaže administratoru potvrdu „Sesija je uspešno prekinuta" i zabeleži incident u audit log.

**Alternativni tokovi:**
A1. **Administrator odlučuje da i zaustavi aktivne AI factory poslove** — u koraku 6: u dijalogu postavi kvačicu **„Zaustavi aktivne AI factory poslove"**; sistem nakon prekida sesije pošalje signal za zaustavljanje svim poslovima korisnika na PARADOX/ITE; korisnik dobija email obaveštenje o zaustavljenim poslovima.
A2. **Korisnik nema aktivnih sesija** — u koraku 2: korisnik se ne prikazuje na listi aktivnih sesija; administrator može pronaći korisnika u **„Svi korisnici"** i direktno deaktivirati nalog ako je potrebno.
A3. **Administrator želi da opozove sesiju s korisničke stranice, ne iz liste sesija** — u koraku 1: administrator umesto liste sesija ode na **„Korisnici"** → pronađe korisnika → otvori detalje naloga → klikne karticu **„Aktivne sesije"** → klikne **„Prisilno prekini sesiju"**; dalje isto od koraka 6.
A4. **Administrator odustane** — u koraku 6: klikne „Otkaži" u dijalogu; sesija ostaje aktivna, nikakva akcija se ne beleži.

**Rezultat:** Sesija korisnika je prekinuta i svi tokeni poništeni; korisnik ne može nastaviti rad bez ponovne prijave (ili uopšte, ako je nalog blokiran); incident je zabeležen u append-only audit logu sa identitetom administratora, vremenskom oznakom, IP adresom korisnika i navedenim razlogom.


## 2. Katalog resursa


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
**Preduslov:** Korisnik je prijavljen preko Keycloak-a i ima dozvolu da objavljuje resurse; resurs (model ili dataset) je već otpremljen u MinIO ili registrovan kao eksterni izvor i nalazi se u stanju **`Draft`**.

**Osnovni tok:**
1. Prijavljen korisnik otvori **„Detalji resursa"** za resurs koji je registrovao i izabere **„Objavi u katalog"**.
2. Sistem prikaže formu **„Objava resursa"** sa poljima za opis, tagove, sektorske tagove, licencu i vidljivost.
3. Korisnik unese opis i izabere tagove i sektorske tagove.
4. Korisnik izabere licencu iz liste **„Licenca"**.
5. Korisnik izabere vidljivost (`private` / `team` / `public`).
6. Korisnik izabere **„Pošalji na pregled"**.
7. Sistem proveri da su obavezna polja popunjena.
8. Sistem postavi resurs u stanje **`Poslat`** (`submitted`), dodeli ga review toku (Data administrator za dataset, Model reviewer za model) i prebaci ga u **`Na pregledu`** (`under review`) kada recenzent preuzme stavku.
9. Sistem prikaže potvrdu da je resurs poslat na pregled i obavesti nadležnog recenzenta.

**Alternativni tokovi:**
A1. **Nedostaju obavezna polja** — u koraku 7: opis, licenca ili tag nisu popunjeni; sistem označi prazna polja i ne šalje resurs; korisnik se vraća na korak 3.
A2. **Resurs već poslat ili objavljen** — u koraku 1: resurs je već u stanju **`Poslat`**, **`Na pregledu`** ili **`Objavljen lokalno`**; sistem ne nudi ponovnu objavu, već prikaže trenutni status; tok se prekida.
A3. **Odustajanje** — u koraku 3 ili 5: korisnik napusti formu bez slanja; sistem sačuva uneto i resurs ostaje u stanju **`Draft`**; review se ne pokreće; resurs ostaje nevidljiv u katalogu.
A4. **Resurs vraćen na doradu** — nakon koraka 9: recenzent odbije ili zatraži izmene; resurs se vraća u **`Draft`**, korisnik dobija notifikaciju sa komentarom i može da ponovi tok od koraka 1. → vidi: *Pregled i odluka o objavi dataseta* (za datasetove), odnosno *Pregled evaluacije i odobravanje / odbijanje / vraćanje verzije* (za modele).

**Rezultat:** Resurs je u stanju **`Na pregledu`**, dodeljen nadležnom recenzentu (Data administrator ili Model reviewer), sa unetim opisom, tagovima, licencom i vidljivošću; još nije vidljiv u javnom katalogu.



**Naziv:** Pregled i odluka o objavi dataseta
**Akter:** Data administrator
**Preduslov:** Data administrator je prijavljen preko Keycloak-a i ima dozvolu za pregled **datasetova**; u redu za pregled postoji bar jedan **dataset** u stanju **`Na pregledu`**. Ovaj scenario pokriva isključivo datasetove — pregled i odluka o objavi **modela** obrađeni su zasebno → vidi: *Pregled evaluacije i odobravanje / odbijanje / vraćanje verzije*.

**Osnovni tok:**
1. Data administrator otvori ekran **„Red za pregled"** i izabere dataset iz liste.
2. Sistem prikaže **„Detalji resursa"** sa metapodacima (opis, tagovi, licenca, vidljivost, poreklo, verzija) i pristupom samom artefaktu u MinIO.
3. Data administrator pregleda metapodatke i sadržaj dataseta, uključujući ručnu procenu da li je izabrana licenca u skladu sa poreklom dataseta (upstream dataset). Sistem ne proverava licencu automatski — to je odgovornost data administratora.
4. Data administrator izabere **„Odobri"**.
5. Sistem zatraži potvrdu i opcioni komentar.
6. Data administrator potvrdi odluku.
7. Sistem postavi dataset u stanje **`Odobren`** i učini ga vidljivim u katalogu prema vidljivosti koju je objavljivač izabrao (`private` / `team` / `public`). Ako je objavljivač izabrao `public`, dataset prelazi u **`Objavljen lokalno`** i postaje vidljiv anonimnom posetiocu.
8. Sistem obavesti objavljivača da je dataset odobren i prikaže data administratoru potvrdu.

**Alternativni tokovi:**
A1. **Vraćanje na doradu / odbijanje** — u koraku 4: data administrator izabere **„Vrati na doradu"** (tražene izmene) ili **„Odbij"** (dataset neprihvatljiv); sistem u oba slučaja zahteva obavezan komentar sa razlogom, vraća dataset u stanje **`Draft`** i obaveštava objavljivača sa komentarom; dataset se ne objavljuje; tok se završava. (Razlika između dve opcije je samo semantika poruke objavljivaču — lifecycle posledica je ista: povratak u **`Draft`**.)
> ⚠ Ako se uvede terminalno stanje `rejected` (odbijen bez mogućnosti ponovnog slanja) različito od `draft` (vraćen, može ponovo) — razdvojiti „Odbij" i „Vrati na doradu" u dve grane sa različitim ciljnim stanjem. Trenutno obe vode u `draft`, što za audit ne beleži da je resurs bio odbijen.
A2. **Nekompatibilna licenca ili sporno poreklo** — u koraku 3: data administrator ručno utvrdi da licenca nije u skladu sa poreklom dataseta; tretira se kao razlog za odbijanje i nastavlja kao A1.
A3. **Dataset povučen tokom pregleda** — u koraku 3: objavljivač je u međuvremenu povukao dataset ili ga prebacio u **`Draft`**; sistem prikaže da dataset više nije na pregledu i uklanja ga iz reda; data administrator se vraća na korak 1.
A4. **Odustajanje od odluke** — u koraku 5: data administrator napusti ekran bez potvrde; dataset ostaje u stanju **`Na pregledu`** u redu; nijedna promena se ne upisuje.

**Rezultat:** Dataset je u stanju **`Odobren`** (ili **`Objavljen lokalno`** ako je vidljivost `public`) i vidljiv u katalogu prema vidljivosti koju je objavljivač izabrao, sa zabeleženom odlukom data administratora (i opcionim komentarom); objavljivač je obavešten.



**Naziv:** Skidanje javnog resursa sa kataloga (moderacija)
**Akter:** Platform administrator
**Preduslov:** Platform administrator je prijavljen preko Keycloak-a i ima dozvolu za moderaciju kataloga; resurs je trenutno javno vidljiv (vidljivost `public`, u stanju **`Objavljen lokalno`** ili **`Objavljen (Pharos)`**).

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


## 3. Inference i korišćenje modela


**Naziv:** Pozivanje modela (REST API ili Jupyter sveska)
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen preko Keycloak-a i ima ulogu sa pravom na inference; postoji model dostupan korisniku prema ulozi i ABAC atributima, podignut na deljenom serving pool-u (model je poslužen, korisnik ga ne diže sam); korisnik ima način autentikacije za izabranu ulaznu tačku — izdat API ključ za REST, ili aktivnu sesiju za svesku.

**Osnovni tok:**
1. Korisnik izabere ulaznu tačku. Za REST: sa ekrana **„Detalji resursa"** modela očita identifikator modela i URL inference endpointa, a sa ekrana **„Moji API ključevi"** kopira postojeći ključ (ili ga, ako ga nema, izda kroz **„Novi API ključ"**). Za svesku: otvori svesku (podizanje JupyterHub okruženja je zaseban tok) i pozove SAIFA SDK. → vidi: *Kreiranje i upravljanje API ključem*
2. Korisnik pošalje HTTP zahtev na inference endpoint u OpenAI-kompatibilnom formatu, sa identifikatorom modela u telu. Za REST ručno postavi API ključ u zaglavlje; za svesku SDK koristi korisnikovu sesiju/token (korisnik ne kuca ključ ručno u svesci).
3. Sistem (API Gateway) autentifikuje zahtev i proveri ulogu i ABAC atribute korisnika za traženi model.
4. Sistem proveri kvotu za tip zahteva `inference` kroz celu hijerarhiju — budžet institucije (nivo afilijacije) i, ako postoji, podlimit korisnika unutar institucije; zahtev prolazi samo ako nijedan nivo nije prekoračen.
5. Sistem prosledi zahtev internom serving sloju (vLLM/TGI) na kom je model poslužen. Serving endpoint je interan i korisnik mu nikad ne pristupa direktno, već uvek kroz gateway.
6. Sistem vrati odgovor modela u OpenAI-kompatibilnom formatu i zabeleži potrošnju tokena na korisnikov nalog i na budžet institucije iz kog korisnik troši.

**Alternativni tokovi:**
A1. **Neispravan ili opozvan API ključ / nevažeća sesija** — u koraku 3: ključ ne postoji, istekao je ili je opozvan (za svesku: sesija je istekla); sistem vrati grešku autentikacije (401) i ne prosleđuje zahtev; tok se prekida (za svesku korisnik ponovo otvara svesku i podiže okruženje).
A2. **Nema pristup modelu** — u koraku 3: korisnikova uloga ili ABAC atributi ne pokrivaju traženi model (npr. sektorska sertifikacija za zdravstvo); sistem vrati grešku autorizacije (403); tok se prekida.
A3. **Prekoračena kvota** — u koraku 4: neki nivo hijerarhije (institucijski budžet ili korisnički podlimit) je iscrpeo dozvoljeni broj tokena za tip `inference`; sistem vrati grešku (429) sa podatkom koji je nivo prekoračen, trenutnom potrošnjom i vremenom kada limit ističe; zahtev se ne izvršava.
A4. **Model nije poslužen / serving ne odgovara** — u koraku 5: serving sloj za traženi model ne odgovara; sistem vrati grešku (503). U modelu deljenog pool-a model bi trebalo da je uvek poslužen, pa 503 ukazuje na grešku infrastrukture koju treba prijaviti operateru, a ne na stanje koje korisnik sam rešava ponovnim pokušajem.
A5. **Streaming odgovor** — u koraku 2 i 6: korisnik je u zahtevu tražio streaming (`stream: true`); sistem vraća odgovor inkrementalno preko SSE umesto kao jedan blok; potrošnja se obračuna tačno jednom, po završetku streama.
> ⚠ Pretpostavka serving modela: ceo preduslov i koraci 5–6 pretpostavljaju deljeni upravljani serving pool (korisnik ne diže model, samo poziva gateway). Ako se umesto toga usvoji dedicated/self-service serving, menja se značenje A4 (503) i potreban je prethodni korak deploja. Potvrditi serving model pre finalizacije.

**Rezultat:** Korisnik je dobio odgovor modela (preko REST API-ja ili u Jupyter svesci); potrošnja tokena je zabeležena na njegov nalog i na budžet institucije; poziv je upisan u audit log. Stanje modela, kataloga i serving sloja nije promenjeno.



**Naziv:** Pokretanje batch inference posla nad datasetom
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen preko Keycloak-a i ima pravo na inference i na pokretanje poslova; postoji model dostupan korisniku i dataset u MinIO nad kojim korisnik ima pravo čitanja; korisnik (kroz budžet svoje institucije) ima dovoljnu kvotu za izvršavanje.

**Osnovni tok:**
1. Prijavljen korisnik otvori **„Detalji resursa"** modela i izabere **„Batch inference"**.
2. Sistem prikaže formu **„Batch inference"** sa izborom ulaznog dataseta, parametara inference-a i odredišta za rezultate.
3. Korisnik izabere dataset iz liste (resursi nad kojima ima pravo čitanja), podesi parametre i potvrdi odredište rezultata u MinIO.
4. Korisnik izabere **„Pokreni posao"**.
5. Sistem proveri pristup korisnika modelu i datasetu (uloga, ABAC) i raspoloživu kvotu kroz hijerarhiju (institucija → korisnik).
6. Sistem rezerviše procenjenu kvotu za posao, da paralelni submit ne potroši isti budžet.
7. Sistem odredi izvršno okruženje prema veličini posla (Kubernetes Job za manje poslove ili SLURM job na PARADOX/ITE za velike) i stavi posao u red.
> ⚠ Pravilo rutiranja „prema veličini posla" nije definisano u referentnim dokumentima. Otvoreno je po čemu se meri veličina (broj redova dataseta, procenjeni GPU sati, veličina ulaza) i gde je prag K8s/SLURM. Vezati za otvoreno pitanje rutiranja između klastera (popis.md, sekcija 5).
8. Sistem kreira zapis posla u stanju **`U redu`** i prikaže potvrdu sa identifikatorom posla. → vidi: *Praćenje statusa i logova posla*

**Alternativni tokovi:**
A1. **Nedostaju obavezna polja** — u koraku 5: dataset ili odredište nisu izabrani; sistem označi prazna polja i ne pokreće posao; forma zadržava već uneto i korisnik se vraća na korak 3.
A2. **Nema pristup modelu ili datasetu** — u koraku 5: uloga ili ABAC atributi ne pokrivaju model ili dataset; sistem odbije pokretanje uz poruku o nedostatku dozvole; posao se ne kreira.
A3. **Nedovoljna kvota** — u koraku 5: preostala kvota (CPU/GPU sati na nekom nivou hijerarhije) ne pokriva procenu posla; sistem odbije pokretanje i prikaže koja kvota i koliko nedostaje; posao se ne kreira i ništa se ne rezerviše.
A4. **Odustajanje** — u koraku 3 ili 4: korisnik napusti formu bez pokretanja; nijedan posao se ne kreira; uneto u formi se ne čuva (za razliku od A1, gde se unos zadržava jer se korisnik vraća da dopuni).

**Rezultat:** Kreiran je batch inference posao u stanju **`U redu`**, vezan za korisnika, model i dataset, sa rezervisanom kvotom (korak 6); posao čeka izvršavanje na Kubernetes/SLURM-u; kreiranje je upisano u audit log.



**Naziv:** Podešavanje rate limitinga (kvote)
**Akter:** Platform administrator
**Preduslov:** Platform administrator je prijavljen preko Keycloak-a i ima dozvolu za administraciju kvota; postoji entitet za koji se limit podešava — institucija (nivo afilijacije), korisnik unutar institucije, ili tip zahteva (`inference`, `AI factory submit`, `federacija`).

**Osnovni tok:**
1. Platform administrator otvori ekran **„Rate limiting"** i izabere nivo na kom podešava: karticu **„Po instituciji"**, **„Po korisniku"** ili **„Po tipu zahteva"**.
2. Sistem prikaže postojeća pravila za izabrani nivo.
3. Administrator izabere entitet (instituciju / korisnika / tip zahteva) i izabere **„Izmeni limit"**.
4. Sistem prikaže formu sa poljima za broj zahteva i token budžet po vremenskom prozoru; dozvoljeno je više prozora (npr. `5h`, `1w`, `1M`).
5. Administrator unese vrednosti i izabere **„Sačuvaj"**.
6. Sistem proveri ispravnost vrednosti (pozitivni brojevi, dosledan vremenski prozor, i da korisnički podlimit ne prelazi budžet institucije kojoj korisnik pripada).
7. Sistem upiše novo pravilo, primeni ga na buduće zahteve i zabeleži ko, kada i šta je promenio.

**Alternativni tokovi:**
A1. **Neispravna vrednost** — u koraku 6: unete vrednosti nisu validne (negativan broj, prazno polje, ili korisnički podlimit veći od budžeta institucije); sistem označi polje i ne čuva; administrator se vraća na korak 5.
A2. **Odustajanje** — u koraku 4 ili 5: administrator napusti formu; postojeće pravilo ostaje nepromenjeno.
A3. **Vraćanje na podrazumevano** — u koraku 4: administrator izabere **„Vrati na podrazumevano"**; sistem ukloni specifično pravilo i vrati entitet na podrazumevani limit za njegov nivo (npr. korisnika na podrazumevanu raspodelu unutar institucije).
> ⚠ Kompozicija limita: za pojedinačni zahtev svaki definisan nivo hijerarhije (institucija, korisnik) i svaki vremenski prozor moraju nezavisno da prođu — bilo koji prekoračen prozor blokira zahtev. Ovo nije „stroži od dva limita pobeđuje", nego „svi se evaluiraju". Potvrditi da li se uopšte uvode korisnički podlimiti ili samo budžet institucije uz atribuciju potrošnje po korisniku u logovima.

**Rezultat:** Za izabrani entitet (instituciju, korisnika ili tip zahteva) važi novo rate limit pravilo, primenjeno na buduće zahteve; promena je zabeležena u audit log. Za pojedinačni zahtev i dalje važi da svi nivoi hijerarhije i svi prozori moraju da prođu.



> ⚠ **Napomena o serving modelu:** Scenariji „Deploy sopstvenog modela" i „Zamena verzije na endpointu" pretpostavljaju različite aktere i nisu međusobno usklađeni jer arhitektonska odluka o serving modelu (deljeni upravljani pool vs. self-service deploy) još nije doneta. Oba scenarija se zadržavaju kao kandidati; pre finalizacije MVP-a jedan će biti uklonjen ili prilagođen.


**Naziv:** Deploy sopstvenog modela kao inference endpoint
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen preko Keycloak-a i ima pravo da deploy-uje modele; model je registrovan i otpremljen u MinIO i nalazi se u stanju **`Odobren`** (`approved`) ili **`Objavljen lokalno`** (`published locally`); postoji raspoloživ GPU kapacitet u serving pool-u.
> ⚠ Ceo scenario je pretpostavka. Self-service deploy modela kao endpoint ne pominje se u referentnim dokumentima (funkcionalnosti.md, popis.md), a u referentnom stack-u (Themelio/GRNET) deploj velikih modela je eksplicitno odobren platformski servis, ne self-service akcija. Ovaj scenario ima smisla samo ako se usvoji dedicated/self-service serving model umesto deljenog upravljanog pool-a (vidi pretpostavku u scenariju „Pozivanje modela"). Do te odluke ovo je kandidat za kasniju fazu, ne za MVP. Pre finalizacije razrešiti i: (a) da li deploy zahteva odobrenje administratora, jer je GPU pool deljen i ograničen resurs; (b) da endpoint ne sme biti vidljiviji od modela (npr. `team` model iza `public` endpointa odao bi pristup modelu).

**Osnovni tok:**
1. Prijavljen korisnik otvori **„Detalji resursa"** svog modela i izabere **„Deploy kao endpoint"**.
2. Sistem prikaže formu **„Deploy modela"** sa izborom serving engine-a (vLLM/TGI), veličine resursa i vidljivosti endpointa.
3. Korisnik podesi parametre i izabere **„Pokreni deploy"**.
4. Sistem proveri pravo korisnika na deploy i raspoloživ GPU kapacitet u serving pool-u.
5. Sistem podigne serving instancu sa izabranim modelom i kreira inference endpoint.
6. Sistem prikaže potvrdu sa URL-om endpointa i identifikatorom modela za pozive.

**Alternativni tokovi:**
A1. **Nedovoljan kapacitet** — u koraku 4: nema slobodnog GPU resursa za traženu veličinu; sistem odbije deploy i predloži manju veličinu ili kasniji pokušaj; endpoint se ne kreira.
A2. **Nema dozvolu za deploy** — u koraku 4: korisnikova uloga ne pokriva deploy; sistem vrati poruku o nedostatku dozvole; tok se prekida.
A3. **Model nije u dozvoljenom stanju** — u koraku 1: model je u stanju **`Draft`** (`draft`) ili **`Na pregledu`** (`under review`); sistem ne nudi deploy dok model ne bude odobren; tok se prekida.
A4. **Neuspeo deploy** — u koraku 5: serving instanca ne uspe da podigne model (npr. nekompatibilan format težina); sistem prikaže grešku sa logom i ne ostavlja pola-podignut endpoint; korisnik se vraća na korak 2.
A5. **Odustajanje** — u koraku 2 ili 3: korisnik napusti formu; endpoint se ne kreira; nijedan resurs nije zauzet.

**Rezultat:** Model je podignut kao aktivan inference endpoint sa dodeljenim URL-om, vidljiv prema izabranoj vidljivosti (ne vidljiviji od samog modela); zauzet je GPU resurs serving pool-a; deploy je upisan u audit log.



**Naziv:** Zamena verzije modela na endpointu bez prekida
**Akter:** Platform administrator
**Preduslov:** Platform administrator je prijavljen preko Keycloak-a i ima dozvolu za upravljanje serving slojem; postoji aktivan inference endpoint sa tekućom verzijom modela; nova verzija modela je u stanju **`Odobren`** (`approved`) ili **`Objavljen lokalno`** (`published locally`) i kompatibilna je sa istim API-jem.
> ⚠ Scenario zavisi od serving modela i nosi nerešenu podelu odgovornosti. Ako deploj endpointa radi korisnik (vidi scenario „Deploy sopstvenog modela"), nedosledno je da zamenu verzije na istom endpointu radi administrator — ceo serving lifecycle treba da pripadne istom akteru. Razrešiti zajedno sa odlukom o serving modelu. Takođe potvrditi ko proverava API-kompatibilnost nove verzije: automatski (uz deklarisanu ulazno/izlaznu šemu po verziji modela) ili ručno (administrator zna), jer od toga zavisi grana A3.

**Osnovni tok:**
1. Platform administrator otvori ekran **„Inference endpointi"** i izabere endpoint čiji model menja.
2. Sistem prikaže **„Detalji endpointa"** sa tekućom verzijom modela i listom dostupnih verzija.
3. Administrator izabere novu verziju i izabere **„Zameni verziju"**.
4. Sistem zatraži potvrdu i prikaže da će se zamena izvesti bez prekida za pozivaoce (nova replika se podiže pre gašenja stare).
5. Administrator potvrdi.
6. Sistem podigne novu verziju kao paralelnu repliku, sačeka da bude spremna, prebaci saobraćaj na nju i tek onda ugasi staru verziju.
7. Sistem prikaže potvrdu da je endpoint sada na novoj verziji i zabeleži ko, kada i sa koje na koju verziju je prešao.

**Alternativni tokovi:**
A1. **Nova verzija se ne podiže** — u koraku 6: nova replika ne postane spremna (greška pri podizanju); sistem zadrži staru verziju aktivnom (saobraćaj se ne prebacuje) i prijavi grešku; endpoint ostaje na tekućoj verziji.
A2. **Nedovoljan kapacitet za paralelnu repliku** — u koraku 6: nema GPU resursa da nova i stara verzija rade istovremeno; sistem prekine zamenu uz poruku i ne gasi staru verziju; administrator se vraća na korak 2.
A3. **Nekompatibilan API nove verzije** — u koraku 3: nova verzija ne izlaže isti API kao tekuća; sistem upozori da zamena bez prekida nije bezbedna i ne dozvoli „hot-swap" (zahteva se zaseban deploy kao novi endpoint).
A4. **Odustajanje** — u koraku 4: administrator ne potvrdi; nijedna promena se ne upisuje; endpoint ostaje na tekućoj verziji.

**Rezultat:** Inference endpoint poslužuje novu verziju modela; pozivaoci nisu imali prekid (stara verzija ugašena tek po prebacivanju saobraćaja); zamena je upisana u audit log sa starom i novom verzijom.


## 4. AI factory i pokretanje poslova


**Naziv:** Pokretanje računski zahtevnog posla
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen preko Keycloak-a i ima pravo na pokretanje poslova; postoje ulazni resursi (model i/ili dataset u MinIO) nad kojima korisnik ima pravo pristupa; korisnik ima dodeljenu ličnu kvotu sa dovoljno preostalih resursa.

**Osnovni tok:**
1. Prijavljen korisnik otvori ekran **„Novi posao"** (alternativno: pokreće isti posao iz sveske preko SAIFA SDK-a ili REST API-ja — vidi A6).
2. Sistem prikaže formu **„Novi posao"** sa izborom tipa posla (fine-tuning / trening / batch obrada), ulaznih resursa, parametara i odredišta rezultata u MinIO.
3. Korisnik izabere tip posla, ulazne resurse (iz resursa nad kojima ima pravo pristupa), podesi parametre i odredište i izabere **„Pokreni posao"**.
4. Sistem proveri da su obavezna polja popunjena; ako jesu, prosledi zahtev na proveru prava i kvote.
5. Sistem proveri pristup korisnika ulaznim resursima (uloga, ABAC) i atomično proveri i rezerviše procenjenu ličnu kvotu korisnika (da paralelni submit ne potroši isti budžet).
6. Sistem odredi izvršno okruženje prema pravilu rutiranja: ako ograničenje pristupa podacima vezuje dataset za određeni klaster (compute-to-data), posao ide tamo; inače se okruženje bira po preostalim kriterijumima rutiranja, a kada je više klastera dopušteno, korisnik bira između ponuđenih.
7. Sistem stavi posao u red, kreira zapis posla u stanju **`U redu`** i prikaže potvrdu sa identifikatorom posla.

**Alternativni tokovi:**
A1. **Nedostaju obavezna polja** — u koraku 4: ulazni resurs ili odredište nisu izabrani; sistem označi prazna polja i ne pokreće posao; forma zadržava uneto i korisnik se vraća na korak 3.
A2. **Nema pristup ulaznom resursu** — u koraku 5: uloga ili ABAC atributi ne pokrivaju model ili dataset (npr. sektorska sertifikacija za zdravstvo); sistem odbije pokretanje uz poruku o nedostatku dozvole; posao se ne kreira.
A3. **Nedovoljna kvota** — u koraku 5: preostala lična kvota ne pokriva procenu posla; sistem odbije pokretanje i prikaže koliko kvote nedostaje, uz uput na **„Zatraži povećanje kvote"**; posao se ne kreira i ništa se ne rezerviše.
A4. **Kvota potrošena tokom rezervacije** — u koraku 5: provera prođe ali rezervacija ne uspe jer je paralelni posao u međuvremenu potrošio preostalu kvotu; sistem odbije pokretanje uz istu poruku kao A3; posao se ne kreira.
A5. **Podaci vezani za klaster koji je nedostupan** — u koraku 6: compute-to-data ograničenje zahteva klaster (npr. ITE) koji trenutno ne prihvata poslove; sistem odbije pokretanje uz poruku da odredišni klaster nije dostupan, da se posao ne može preusmeriti jer podaci ne smeju da ga napuste, i da pokuša ponovo kasnije; rezervisana kvota se oslobađa; posao se ne kreira.
A6. **Pokretanje iz sveske / API-ja** — u koraku 1–3: korisnik umesto forme šalje posao preko SAIFA SDK-a iz sveske ili REST poziva; iste provere (koraci 4–6) važe identično, autentikacija ide preko sesije sveske ili API ključa; ishod je isti zapis posla.
A7. **Odustajanje** — u koraku 3: korisnik napusti formu bez pokretanja; nijedan posao se ne kreira; uneto u formi se ne čuva (za razliku od A1, gde se unos zadržava jer se korisnik vraća da dopuni).

**Rezultat:** Kreiran je posao u stanju **`U redu`**, vezan za korisnika i ulazne resurse, sa rezervisanom ličnom kvotom (korak 5); posao čeka izvršavanje na Kubernetes/SLURM okruženju određenom u koraku 6; kreiranje je upisano u audit log.



**Naziv:** Praćenje statusa i logova posla
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen preko Keycloak-a; postoji bar jedan posao koji je korisnik pokrenuo, u nekom od stanja izvršavanja (**`U redu`**, **`U toku`**, **`Završen`**, **`Neuspeo`**, **`Prekinut`**).

**Osnovni tok:**
1. Prijavljen korisnik otvori ekran **„Moji poslovi"**.
2. Sistem prikaže listu korisnikovih poslova sa tipom, statusom i vremenom pokretanja.
3. Korisnik izabere posao iz liste.
4. Sistem prikaže **„Detalji posla"**: trenutni status, poziciju u redu (ako je u redu), procenu potrošnje resursa do tada (ako je dostupna iz scheduler-a) i live log izvršavanja.
5. Korisnik prati osvežavanje statusa i logova dok posao traje.
6. Po prelasku posla u završno stanje, sistem prikaže konačni status (**`Završen`** sa linkom ka rezultatima u MinIO, ili **`Neuspeo`** / **`Prekinut`**).

**Alternativni tokovi:**
A1. **Posao još u redu** — u koraku 4: posao nije počeo da se izvršava; sistem prikaže poziciju u redu i procenu čekanja (ako je dostupna iz SLURM-a/scheduler-a); logovi izvršavanja još ne postoje.
A2. **Posao neuspeo** — u koraku 6: izvršavanje se završilo greškom; sistem prikaže status **`Neuspeo`** sa porukom o grešci i poslednjim logovima; rezultata nema; korisnik može da pokrene posao ponovo iz **„Detalji posla"** sa istim parametrima, pri čemu re-run prolazi iste provere pristupa i kvote (koraci 4–6 scenarija → vidi: *Pokretanje računski zahtevnog posla*) bez ponovnog popunjavanja forme.
A3. **Kašnjenje live log streama** — u koraku 5: prikaz logova zaostaje za stvarnim izvršavanjem; sistem nastavlja da dopunjuje logove kad stignu; status posla ostaje verodostojan jer se čita iz scheduler-a, ne iz log streama.
A4. **Posao prisilno zaustavljen spolja** — u koraku 5/6: posao je zaustavio administrator platforme (zbog stabilnosti klastera) ili je zaustavljen pri deaktivaciji naloga; sistem prikaže status **`Prekinut`** sa naznakom da zaustavljanje nije pokrenuo sam korisnik.

**Rezultat:** Korisnik je video aktuelni status, potrošnju i logove posla; za završen posao ima link ka rezultatima u MinIO. Pregled ne menja stanje posla. Prekid posla od strane korisnika obrađen je u zasebnom scenariju.



**Naziv:** Preuzimanje rezultata posla
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen preko Keycloak-a; postoji posao koji je korisnik pokrenuo u stanju **`Završen`**; rezultati su upisani u odredišni prostor u MinIO i korisnik nad njima ima pravo čitanja.

**Osnovni tok:**
1. Prijavljen korisnik otvori **„Detalji posla"** za završen posao.
2. Sistem prikaže status **`Završen`** i odeljak **„Rezultati"** sa listom izlaznih artefakata (fajlovi, log, metrika) u MinIO.
3. Korisnik izabere **„Preuzmi rezultate"** (ceo skup) ili pojedinačni artefakt iz liste.
4. Sistem generiše privremeni (presigned) link i započne preuzimanje izabranih artefakata u korisnikovo lokalno okruženje.
5. Sistem zabeleži preuzimanje rezultata u audit log.

**Alternativni tokovi:**
A1. **Rezultat je novi resurs u registru** — u koraku 2: izlaz posla je model (npr. fine-tune rezultat); umesto preuzimanja fajlova korisnik izabere **„Registruj kao resurs"** i model se upisuje u registar u stanju **`Draft`** (objava ide kroz zaseban tok registracije i objave resursa); preuzimanje fajlova i dalje ostaje moguće.
A2. **Posao nije završen** — u koraku 1: posao je u stanju **`U redu`** / **`U toku`** / **`Neuspeo`** / **`Prekinut`**; sistem ne nudi preuzimanje rezultata (za neuspeo/prekinut posao nudi samo logove); tok se prekida.
A3. **Rezultati istekli ili obrisani** — u koraku 2: artefakti su prošli retenciju ili su obrisani; sistem prikaže poruku da rezultati više nisu dostupni i da posao treba ponoviti; preuzimanje nije moguće.
A4. **Prekid preuzimanja** — u koraku 4: veza se prekine tokom preuzimanja velikog artefakta; preuzimanje se nastavlja/ponavlja (resumable), original u MinIO ostaje netaknut.

**Rezultat:** Izabrani rezultati posla preuzeti su u korisnikovo okruženje (ili je izlaz registrovan kao novi resurs); originali ostaju u MinIO; preuzimanje je upisano u audit log. Stanje posla nije promenjeno.



**Naziv:** Prijem notifikacije o ishodu posla
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik ima bar jedan pokrenut posao; u podešavanjima profila notifikacije su uključene (in-app podrazumevano, email opciono).

**Osnovni tok:**
1. Posao korisnika pređe u završno stanje (**`Završen`**, **`Neuspeo`** ili **`Prekinut`**), što sistem detektuje.
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
6. Sistem proveri da su iznos i obrazloženje uneti, kreira zahtev u stanju **`Na čekanju`** i prosledi ga vlasniku odluke: predstavniku organizacije ako je korisnik član organizacije, odnosno platform administratoru ako je korisnik Nezavisni korisnik.
7. Sistem prikaže potvrdu da je zahtev poslat i obavesti odlučioca.

**Alternativni tokovi:**
A1. **Nedostaje iznos ili obrazloženje** — u koraku 6: obavezno polje nije popunjeno; sistem označi prazno polje i ne šalje zahtev; uneto se zadržava i korisnik se vraća na korak 5.
A2. **Već postoji zahtev na čekanju** — u koraku 3: korisnik ima nerešen zahtev za kvotu; sistem ne nudi novi, već prikaže status postojećeg; tok se prekida.
A3. **Odustajanje** — u koraku 4 ili 5: korisnik napusti formu bez slanja; nijedan zahtev se ne kreira.

**Rezultat:** Kreiran je zahtev za povećanje kvote u stanju **`Na čekanju`**, vezan za korisnika i prosleđen predstavniku organizacije (ili platform administratoru za Nezavisnog korisnika), sa traženim iznosom i obrazloženjem; slanje je upisano u audit log. Lična kvota korisnika još nije promenjena.



**Naziv:** Pokretanje posla iz zaključanog šablona (student)
**Akter:** Student
**Preduslov:** Student je prijavljen preko Keycloak-a i ima ulogu Student; postoji bar jedan zaključan šablon posla dodeljen njegovom kursu/grupi; student ima dodeljenu ličnu kvotu sa dovoljno preostalih resursa za izvršavanje; izlazni parametri i resursi šablona su unapred fiksirani i student ih ne menja.

**Osnovni tok:**
1. Student otvori ekran **„Šabloni poslova"** i izabere dodeljeni šablon iz liste.
2. Sistem prikaže **„Detalji šablona"**: opis posla, fiksirane parametre i polja koja je dozvoljeno popuniti (npr. izbor ulaznog dataseta iz unapred odobrene liste).
3. Student popuni dozvoljena polja i izabere **„Pokreni"**.
4. Sistem proveri ulogu studenta, proveri i rezerviše preostalu ličnu kvotu studenta (da paralelni submit ne potroši isti budžet) i izvrši posao na unapred određenom okruženju zapisanom u šablonu (Kubernetes Job ili SLURM job na ITE/PARADOX — student ne bira).
5. Sistem kreira zapis posla u stanju **`U redu`** i prikaže potvrdu sa identifikatorom posla.

**Alternativni tokovi:**
A1. **Nedovoljna kvota** — u koraku 4: preostala lična kvota studenta ne pokriva procenu posla; sistem odbije pokretanje uz poruku koliko kvote nedostaje i kome da se obrati (mentor/predstavnik); posao se ne kreira i ništa se ne rezerviše.
A2. **Obavezno polje nije popunjeno** — u koraku 3: dozvoljeno polje (npr. ulazni dataset) nije izabrano; sistem označi prazno polje i ne pokreće posao; uneto se zadržava i student se vraća na korak 3.
A3. **Šablon povučen ili istekao** — u koraku 1: šablon više nije aktivan (mentor ga povukao ili je rok kursa istekao); sistem prikaže da šablon nije dostupan i uklanja ga iz liste.
A4. **Odustajanje** — u koraku 3: student napusti ekran bez pokretanja; nijedan posao se ne kreira.

**Rezultat:** Kreiran je posao iz zaključanog šablona u stanju **`U redu`**, vezan za studenta, sa rezervisanom ličnom kvotom i parametrima u granicama koje šablon dozvoljava; posao čeka izvršavanje na unapred određenom okruženju; dalji životni ciklus prati se → vidi: *Praćenje statusa i logova posla*; kreiranje je upisano u audit log. Student nije imao pristup pisanju skripte niti izboru klastera.



**Naziv:** Odluka o zahtevu člana za povećanje kvote
**Akter:** Predstavnik organizacije
**Preduslov:** Predstavnik je prijavljen preko Keycloak-a i ima aktivnu organizaciju; postoji bar jedan zahtev člana za povećanje kvote u stanju **`Na čekanju`**.

**Osnovni tok:**
1. Predstavnik organizacije otvori **„Moja organizacija"** → **„Zahtevi za kvotu"** i izabere zahtev iz liste.
2. Sistem prikaže detalje zahteva: ime člana, traženi iznos po tipu resursa, obrazloženje, dosadašnju potrošnju člana i preostali kapacitet pool-a organizacije.
3. Predstavnik pregleda zahtev i izabere **„Odobri"**.
4. Sistem zatraži potvrdu i prikaže koliko će se oduzeti iz pool-a organizacije i koliko ostaje.
5. Predstavnik potvrdi (uz mogućnost da odobri manji iznos od traženog).
6. Sistem prebaci odobreni iznos iz pool-a organizacije na ličnu kvotu člana, postavi zahtev u stanje **`Odobren`** i obavesti člana.
7. Sistem prikaže ažuriranu raspodelu kvote u organizaciji.

**Alternativni tokovi:**
A1. **Odbijanje zahteva** — u koraku 3: predstavnik izabere **„Odbij"**; sistem zatraži obavezan razlog, postavi zahtev u stanje **`Odbijen`**, obavesti člana sa razlogom; kvota se ne menja; tok se završava.
A2. **Pool organizacije nema dovoljno kvote** — u koraku 4: traženi (ili umanjeni) iznos prelazi preostali kapacitet pool-a; sistem upozori da odobravanje nije moguće bez prethodnog povećanja kvote organizacije i uputi predstavnika da zatraži povećanje od platform administratora; odobravanje se ne izvršava.
A3. **Zahtev u međuvremenu povučen ili nevažeći** — u koraku 3: član je povukao zahtev ili je deaktiviran; sistem prikaže da zahtev više nije aktivan i uklanja ga iz liste; predstavnik se vraća na korak 1.
A4. **Odustajanje** — u koraku 4 ili 5: predstavnik napusti ekran bez potvrde; zahtev ostaje u stanju **`Na čekanju`**; nijedna promena se ne upisuje.

**Rezultat:** Zahtev je u stanju **`Odobren`** (uz prenos kvote iz pool-a organizacije na ličnu kvotu člana) ili **`Odbijen`** (bez promene kvote); član je obavešten o ishodu; odluka i prenos kvote upisani su u audit log.



**Naziv:** Prisilno zaustavljanje posla koji ugrožava stabilnost klastera
**Akter:** Platform administrator
**Preduslov:** Platform administrator je prijavljen preko Keycloak-a i ima dozvolu za upravljanje poslovima na klasteru; postoji posao u stanju **`U toku`** za koji nadzor pokazuje da ugrožava stabilnost klastera (npr. prekoračenje resursa, zaglavljen posao, opterećenje koje blokira druge).

**Osnovni tok:**
1. Platform administrator otvori ekran **„Svi poslovi"** (administratorski pregled svih aktivnih poslova na PARADOX/ITE i Kubernetes pool-u).
2. Sistem prikaže listu aktivnih poslova sa vlasnikom, klasterom, potrošnjom resursa i statusom.
3. Administrator izabere problematičan posao i otvori **„Detalji posla"**.
4. Administrator izabere **„Prisilno zaustavi posao"**.
5. Sistem zatraži obavezan razlog (zaustavljanje mora biti zabeleženo u audit log).
6. Administrator unese razlog i potvrdi.
7. Sistem pošalje signal za zaustavljanje poslu na odgovarajućem klasteru (preko SLURM-a ili Kubernetes-a) i postavi posao u prelazno stanje **`Zaustavljanje u toku`**.
8. Po potvrdi klastera da je posao zaustavljen, sistem oslobodi rezervisanu kvotu i obračuna stvarnu potrošnju do prekida, postavi posao u stanje **`Prekinut`**, obavesti vlasnika posla sa razlogom i prikaže administratoru potvrdu.

**Alternativni tokovi:**
A1. **Posao se u međuvremenu završio** — u koraku 4: posao je prešao u **`Završen`** / **`Neuspeo`** pre zaustavljanja; sistem prikaže da posao više nije aktivan i ne preduzima ništa; administrator se vraća na korak 1.
A2. **Signal za zaustavljanje ne prolazi odmah** — u koraku 8: klaster ne potvrdi zaustavljanje u očekivanom roku; sistem zadrži posao u stanju **`Zaustavljanje u toku`**, ponavlja/eskalira signal i ne prebacuje posao u **`Prekinut`** dok klaster ne potvrdi; administrator vidi da prekid još nije potvrđen.
A3. **Odustajanje** — u koraku 5 ili 6: administrator napusti ekran bez potvrde; nijedna promena se ne upisuje; posao nastavlja da se izvršava.

**Rezultat:** Posao je u stanju **`Prekinut`**, rezervisana kvota oslobođena a stvarna potrošnja do prekida obračunata; vlasnik je obavešten; ko, kada i zašto je zaustavio posao zabeleženo je u audit log.


## 5. Model lifecycle — treniranje, fine-tuning, evaluacija, registracija


**Naziv:** Treniranje modela nad datasetom iz kataloga
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen i ima dozvolu pristupa odabranom datasetu; korisnik ima dovoljnu kvotu (GPU sati) za pokretanje posla.

**Osnovni tok:**
1. Prijavljen korisnik otvori ekran **„Katalog"** i u **„Filteri"** izabere dataset za trening.
2. Korisnik na ekranu **„Detalji resursa"** izabere akciju **„Pokreni trening"**.
3. Sistem prikaže formu **„Konfiguracija treninga"** sa baznim modelom/arhitekturom, hiperparametrima i izborom okruženja izvršavanja (PARADOX/ITE ili GPU pool).
4. Korisnik popuni konfiguraciju i klikne **„Pošalji posao"**.
5. Posao se kreira i prati kako je opisano u scenariju za pokretanje računski zahtevnog posla (provera kvote i dozvola, kreiranje posla, ekran **„Status posla"**, prelaz **`U redu`** → **`U toku`**, live logovi i napredak). → vidi: *Pokretanje računski zahtevnog posla*
6. Po prelasku posla u **`Završen`**, Sistem upiše artefakte treninga u artefakt skladište, poveže ih sa zapisom posla i sa automatski uhvaćenim lineage-om (dataset, konfiguracija, izvršni job), pa pošalje notifikaciju.

**Alternativni tokovi:**
A1. **Nema dozvole za dataset** — u koraku 1: dataset se ne pojavljuje u rezultatima ili je akcija **„Pokreni trening"** onemogućena; tok se prekida pre slanja.
A2. **Posao ne prođe proveru kvote ili padne / korisnik ga prekine** — obrađeno scenarijem za pokretanje računski zahtevnog posla (stanja **`Neuspeo`** / **`Zaustavljanje u toku`** → **`Prekinut`**). Posledica za ovaj tok: ako posao nije **`Završen`**, artefakti se ne upisuju i model se ne kreira; eventualni delimični checkpoint ostaje vezan za zapis posla kao artefakt posla, ne kao model. → vidi: *Pokretanje računski zahtevnog posla*

**Rezultat:** Posao treninga je u stanju **`Završen`**; artefakti modela su u artefakt skladištu, povezani sa zapisom posla i automatskim lineage-om (dataset, konfiguracija, job). Model još nije registrovan → vidi: *Registracija istreniranog artefakta kao modela (verzija 1)*.



**Naziv:** Fine-tuning modela nad sopstvenim labeled datasetom
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen; postojeći bazni model je dostupan u katalogu; korisnik je već registrovao sopstveni labeled dataset i ima dozvolu pristupa nad njim; korisnik ima dovoljnu kvotu.
> **Napomena (granica 5.1 ↔ 5.2):** treniranje od nule i fine-tuning dele isti tok izvršavanja (računski posao) i razlikuju se samo ulaznom tačkom (dataset-first vs model-first) i sadržajem konfiguracione forme. Tip posla (`trening` / `fine-tuning`) je polje u formi. Zadržani su kao dva scenarija jer je ulazna tačka u UI-ju različita; ako se u implementaciji svedu na jednu formu sa prekidačem, spojiti u jedan parametrizovan scenario.

**Osnovni tok:**
1. Prijavljen korisnik otvori **„Katalog"**, nađe bazni model i na **„Detalji resursa"** izabere **„Fine-tuning"**.
2. Sistem prikaže formu **„Konfiguracija fine-tuning-a"** sa izborom sopstvenog dataseta, strategijom treninga (pun fine-tuning ili PEFT — LoRA/QLoRA) i hiperparametrima.
3. Korisnik izabere dataset, strategiju i parametre, pa klikne **„Pošalji posao"**.
4. Posao se kreira i prati kako je opisano u scenariju za pokretanje računski zahtevnog posla. → vidi: *Pokretanje računski zahtevnog posla*
5. Po prelasku posla u **`Završen`**, Sistem upiše dobijeni checkpoint u artefakt skladište sa automatskim lineage-om (bazni model, dataset, konfiguracija, job) i notifikuje korisnika.

**Alternativni tokovi:**
A1. **Nekompatibilan dataset** — u koraku 3: Sistem prijavi grešku validacije (format/zadatak ne odgovara baznom modelu); posao se ne kreira.
A2. **Posao ne prođe proveru, padne ili korisnik prekine** — obrađeno scenarijem za pokretanje računski zahtevnog posla; checkpoint se ne upisuje ako posao nije **`Završen`**. → vidi: *Pokretanje računski zahtevnog posla*
A3. **Fine-tuning konvergira u model gori od baznog** — u koraku 5: posao jeste **`Završen`**, ali evaluacija pokazuje pad u odnosu na baseline. Ovo nije greška toka — checkpoint se upisuje normalno; odluka da li ga uopšte registrovati/predložiti za objavu donosi se kasnije, na osnovu evaluacije. Naznačeno ovde da se zna da **`Završen`** posao ne znači „dobar model". → vidi: *Evaluacija verzije modela na benchmark setu*

**Rezultat:** Fine-tune-ovani checkpoint je u artefakt skladištu sa automatskim lineage-om ka baznom modelu i datasetu. Model još nije registrovan → vidi: *Registracija istreniranog artefakta kao modela (verzija 1)*.



**Naziv:** Fine-tuning nad osetljivim podacima koji ne napuštaju kontrolisano okruženje
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen; dataset je označen kao osetljiv (atribut osetljivosti) i vezan za kontrolisano okruženje; korisnik ima ABAC atribut/sertifikaciju potreban za taj dataset; korisnik ima dovoljnu kvotu na klasteru gde podaci žive.

**Osnovni tok:**
1. Prijavljen korisnik otvori **„Katalog"** i izabere bazni model, pa **„Fine-tuning"**.
2. Sistem prikaže formu i ponudi samo ona okruženja izvršavanja koja zadovoljavaju režim osetljivog dataseta (klaster na kom podaci žive); opcija download podataka nije ponuđena.
3. Korisnik izabere osetljivi dataset i parametre, pa klikne **„Pošalji posao"**.
4. Sistem proveri ABAC atribut, dozvole i kvotu i veže izvršavanje za kontrolisano okruženje; posao se dalje prati kako je opisano u scenariju za pokretanje računski zahtevnog posla. → vidi: *Pokretanje računski zahtevnog posla*
5. Sistem pokrene posao tamo gde podaci žive; korisnik prati napredak i logove, ali nema pristup sirovim podacima.
6. Po prelasku posla u **`Završen`**, iz kontrolisane zone izlazi **samo artefakt modela (pune težine ili PEFT adapter), nakon prolaska kroz kontrolu izlaska**; Sistem ga upiše u artefakt skladište sa lineage-om i oznakom `compute-to-data`, pa notifikuje korisnika.

**Alternativni tokovi:**
A1. **Korisnik nema potreban ABAC atribut** — u koraku 4: Sistem odbije posao uz poruku da nedostaje sektorska sertifikacija/saglasnost; posao se ne kreira. *(Realno bi se akcija Fine-tuning mogla onemogućiti već u koraku 1; provera je ostavljena i u koraku 4 jer atribut može da nedostaje i pored vidljivosti modela — vidi otvorenu tačku ABAC vidljivosti.)*
A2. **Posao padne ili korisnik prekine** — obrađeno scenarijem za pokretanje računski zahtevnog posla; logovi se čuvaju bez izlaganja osetljivih podataka, artefakt ne izlazi iz zone. → vidi: *Pokretanje računski zahtevnog posla*

**Rezultat:** Artefakt modela je u artefakt skladištu sa lineage-om i oznakom `compute-to-data`; sirovi osetljivi podaci nisu napustili kontrolisano okruženje.



**Naziv:** Pregled i poređenje eksperimenata (experiment tracking)
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen; bar jedan trening/fine-tuning je pokrenut kroz platformu i povezan sa korisnikovim (ili timskim) okruženjem za experiment tracking.

**Osnovni tok:**
1. Prijavljen korisnik otvori ekran **„Eksperimenti"** i vidi `run`-ove sa parametrima i metrikama (uživo za poslove **`U toku`**, konačno za **`Završen`**).
2. Korisnik izabere dva ili više `run`-ova i klikne **„Uporedi"**.
3. Sistem prikaže uporedni prikaz parametara i metrika (tabela i grafici).
4. Korisnik na osnovu prikaza identifikuje najbolji `run`. *(Izbor „najboljeg" je ljudska procena po metrici koju korisnik bira — sistem ne promoviše automatski.)*

**Alternativni tokovi:**
A1. **Izabran samo jedan `run`** — u koraku 2: dugme **„Uporedi"** je onemogućeno dok se ne izabere bar dva.
A2. **`run`-ovi nemaju zajedničke metrike** — u koraku 3: Sistem prikaže dostupne metrike i obeleži one koje kod nekog `run`-a nema; poređenje se prikazuje sa prazninama.
A3. **Alat za experiment tracking nedostupan tokom posla na klasteru** — beleženje za vreme nedostupnosti izostane; po vraćanju veze nastavlja se, ili `run` ostaje delimičan uz oznaku. *(Ponašanje preko delimično izolovane veze ka klasteru je eliminacioni kriterijum iz `funkcionalnosti.md` — nije rešeno, vidi otvorene tačke.)*

**Rezultat:** Prikazan je uporedni prikaz eksperimenata; nijedno stanje sistema se ne menja (read-only operacija).



**Naziv:** Registracija istreniranog artefakta kao modela (verzija 1)
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen; postoji istreniran model/checkpoint kao artefakt u artefakt skladištu koji pripada korisniku ili njegovom timu.

**Osnovni tok:**
1. Prijavljen korisnik otvori ekran **„Moji modeli"** i izabere **„Registruj model"**.
2. Sistem prikaže formu **„Registracija modela"** sa poljima za metapodatke (naziv, licenca, jezik, tip zadatka, vidljivost public/private/team) i izborom izvornog artefakta.
3. Korisnik poveže artefakt, popuni metapodatke i klikne **„Sačuvaj"**.
4. Sistem kreira zapis u registru u stanju **`Draft`**, dodeli mu verziju 1 i poveže lineage.
5. Sistem prikaže **„Detalji resursa"** novog modela sa stanjem **`Draft`**, verzijom i vidljivošću.
> **Lineage — dva izvora poverenja:** ako je artefakt nastao kroz platformski posao (treniranje/fine-tuning), lineage je **automatski uhvaćen** (dataset, konfiguracija, job) i pouzdan. Ako korisnik ručno povezuje artefakt uvezen spolja, lineage se unosi rukom i obeležava se kao `lineage: nepotvrđen`, jer Sistem ne može da garantuje tačnost ručno unete veze. Ova razlika mora biti vidljiva na **„Detalji resursa"**.

**Alternativni tokovi:**
A1. **Nepotpuni metapodaci** — u koraku 3: Sistem označi obavezna polja koja nedostaju (npr. licenca); zapis se ne kreira dok se ne popune.
A2. **Model sa istim nazivom već postoji** — u koraku 4: Sistem ponudi da se postojeći model verzioniše umesto kreiranja novog zapisa; korisnik bira. → vidi: *Dodavanje nove verzije postojećeg modela*
A3. **Odustajanje** — u bilo kom koraku pre koraka 4: korisnik napusti formu; zapis se ne kreira.

**Rezultat:** Model je registrovan kao verzija 1 u stanju **`Draft`**, sa metapodacima i lineage-om (potvrđenim ili `nepotvrđen`), vidljiv prema podešenoj vidljivosti.



**Naziv:** Dodavanje nove verzije postojećeg modela
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen; model je već registrovan; korisnik ima novi checkpoint/artefakt za istu liniju modela i dozvolu izmene.

**Osnovni tok:**
1. Prijavljen korisnik otvori **„Detalji resursa"** registrovanog modela i izabere **„Dodaj verziju"**.
2. Sistem prikaže formu za povezivanje novog artefakta i lineage podataka (dataset, kod, job).
3. Korisnik poveže artefakt, popuni izvor i klikne **„Sačuvaj"**.
4. Sistem kreira novu verziju (npr. v2) u stanju **`Draft`** i poveže njen lineage; prethodne verzije ostaju dostupne.
5. Sistem prikaže ažurirane **„Detalji resursa"** sa listom verzija.

**Alternativni tokovi:**
A1. **Nepotpun lineage** — u koraku 3: Sistem upozori da nedostaje izvor (dataset ili job). Da li se verzija sme sačuvati sa nepotpunim lineage-om zavisi od lineage politike: ako politika to dozvoljava, verzija se snima sa oznakom `lineage: nepotpun`; inače se traži dopuna. *(Governance pravilo — vidi otvorene tačke.)*
A2. **Nema dozvole izmene** — u koraku 1: akcija **„Dodaj verziju"** je onemogućena.

**Rezultat:** Model ima novu verziju u stanju **`Draft`** sa povezanim lineage-om; prethodne verzije ostaju dostupne za pregled i rollback.



**Naziv:** Promocija `run`-a u verziju registrovanog modela
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen; izabrani `run` ima validan artefakt modela; korisnik ima dozvolu upisa u registar za taj model.

**Osnovni tok:**
1. Prijavljen korisnik na ekranu **„Eksperimenti"** otvori izabrani `run` i klikne **„Promoviši u registar"**.
2. Sistem prikaže formu sa izborom ciljne linije modela (postojeća ili nova).
3. Korisnik potvrdi i klikne **„Promoviši"**.
4. Sistem kreira verziju modela iz tog `run`-a u stanju **`Draft`** i poveže automatski lineage ka `run`-u (a preko njega ka datasetu, konfiguraciji i poslu).
5. Sistem prikaže **„Detalji resursa"** modela sa novom verzijom u stanju **`Draft`**.

**Alternativni tokovi:**
A1. **`run` nema validan artefakt modela** — u koraku 1: akcija **„Promoviši u registar"** je onemogućena uz objašnjenje.
A2. **Nema dozvole za upis u registar** — u koraku 3: Sistem odbije promociju uz poruku o dozvoli; verzija se ne kreira.
A3. **Odustajanje** — pre koraka 3: korisnik napusti formu; promocija se ne izvršava.

**Rezultat:** Iz izabranog `run`-a je nastala nova verzija registrovanog modela u stanju **`Draft`**, sa automatskim lineage-om ka `run`-u.



**Naziv:** Evaluacija verzije modela na benchmark setu
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen; postoji verzija modela za evaluaciju; korisnik ima dovoljnu kvotu.

**Osnovni tok:**
1. Prijavljen korisnik na **„Detalji resursa"** modela izabere **„Pokreni evaluaciju"**.
2. Sistem prikaže formu sa izborom benchmark seta i verzije benchmarka (lm-evaluation-harness + sektorski datasetovi).
3. Korisnik izabere benchmark i verziju benchmarka i klikne **„Pošalji posao"**.
4. Evaluacioni posao se kreira i prati kao platformski posao. → vidi: *Pokretanje računski zahtevnog posla*
5. Po prelasku posla u **`Završen`**, Sistem upiše rezultate evaluacije i poveže ih sa verzijom modela i verzijom benchmarka, pa notifikuje korisnika.

**Alternativni tokovi:**
A1. **Izabrani benchmark dataset nije dostupan** — u koraku 2/3: ako sektorski benchmark nije objavljen ili korisnik nema pristup, Sistem ga ne nudi u listi ili blokira slanje uz objašnjenje; posao se ne kreira.
A2. **Posao padne ili korisnik prekine** — obrađeno scenarijem za pokretanje računski zahtevnog posla; rezultati se ne upisuju ako posao nije **`Završen`**. → vidi: *Pokretanje računski zahtevnog posla*

**Rezultat:** Rezultati evaluacije su sačuvani i povezani sa verzijom modela i verzijom benchmarka, dostupni na **„Detalji resursa"**.



**Naziv:** Podnošenje verzije modela za recenziju i objavu
**Akter:** Prijavljen korisnik (vlasnik modela / član tima sa dozvolom)
**Preduslov:** Korisnik je prijavljen; postoji verzija modela u stanju **`Draft`**; verzija ima rezultate evaluacije → vidi: *Evaluacija verzije modela na benchmark setu*.

**Osnovni tok:**
1. Prijavljen korisnik na **„Detalji resursa"** verzije u stanju **`Draft`** klikne **„Pošalji na recenziju"**.
2. Sistem proveri da verzija ima rezultate evaluacije i kompletne obavezne metapodatke.
3. Korisnik (opciono) upiše propratni komentar i potvrdi.
4. Sistem prevede verziju iz **`Draft`** u **`Na recenziji`**, stavi je u **„Red za pregled"** i notifikuje recenzente.

**Alternativni tokovi:**
A1. **Verzija nema evaluaciju** — u koraku 2: Sistem blokira slanje i uputi korisnika na **„Pokreni evaluaciju"**; stanje ostaje **`Draft`**. → vidi: *Evaluacija verzije modela na benchmark setu*
A2. **Nepotpuni obavezni metapodaci** — u koraku 2: Sistem označi šta nedostaje; stanje ostaje **`Draft`**.
A3. **Odustajanje** — pre koraka 4: korisnik napusti tok; stanje ostaje **`Draft`**.

**Rezultat:** Verzija modela je u stanju **`Na recenziji`** i nalazi se u **„Redu za pregled"**, sa zabeleženim podnosiocem i vremenom.



**Naziv:** Vraćanje servisnog aliasa na prethodnu verziju
**Akter:** Prijavljen korisnik (sa dozvolom izmene nad modelom)
**Preduslov:** Korisnik je prijavljen; registrovani model ima bar dve verzije; korisnik ima dozvolu izmene nad tim modelom.

**Osnovni tok:**
1. Prijavljen korisnik otvori **„Detalji resursa"** modela i pređe na karticu **„Verzije"**.
2. Korisnik izabere raniju verziju i klikne **„Vrati na ovu verziju"**.
3. Sistem zatraži potvrdu akcije.
4. Korisnik potvrdi.
5. Sistem premesti servisni alias (npr. `production`) na izabranu raniju verziju; sve verzije ostaju sačuvane.
6. Sistem prikaže ažurirane **„Detalji resursa"** sa naznakom koja je verzija sada servirana.

**Alternativni tokovi:**
A1. **Model ima samo jednu verziju** — u koraku 1: akcija **„Vrati na ovu verziju"** je onemogućena.
A2. **Odustajanje** — u koraku 4: korisnik odbije potvrdu; servisni alias ostaje nepromenjen.
A3. **Verzija koja se servira vezana je za aktivni deployment** — u koraku 5: Sistem upozori da je verzija u upotrebi i traži dodatnu potvrdu pre prebacivanja. *(Da li rollback dira živu serviranu instancu ili samo pokazivač u registru zavisi od toga koliko model deployment ulazi u MVP — vidi otvorene tačke.)*

**Rezultat:** Servisni alias modela pokazuje na izabranu raniju verziju; nijedna verzija nije obrisana, nijedno lifecycle stanje nije promenjeno.



**Naziv:** Pregled evaluacije i odobravanje / odbijanje / vraćanje verzije
**Akter:** Model reviewer (uloga `model-reviewer`)
**Preduslov:** Model reviewer je prijavljen i ima ulogu `model-reviewer`; postoji verzija **modela** u stanju **`Na recenziji`** sa rezultatima evaluacije. Ovaj scenario pokriva isključivo recenziju modela — pregled i odluka o objavi **dataseta** obrađeni su zasebno → vidi: *Pregled i odluka o objavi dataseta*.

**Osnovni tok:**
1. Model reviewer otvori ekran **„Red za pregled"** i izabere verziju koja čeka odluku.
2. Sistem prikaže **„Detalji resursa"** sa rezultatima evaluacije, lineage-om i metapodacima.
3. Reviewer pregleda rezultate i upiše komentar.
4. Reviewer klikne **„Odobri"** ili **„Odbij"**.
5. Sistem zabeleži odluku, autora i vreme; verzija pređe iz **`Na recenziji`** u **`Odobren`** ili **`Odbijen`**.
6. Sistem notifikuje podnosioca o ishodu.

**Alternativni tokovi:**
A1. **Nepotpuni podaci za odluku** — u koraku 3: reviewer klikne **„Vrati podnosiocu"** uz komentar šta nedostaje; verzija pređe iz **`Na recenziji`** u **`Vraćeno na doradu`** (vraća se na **`Draft`** na strani podnosioca), bez odluke o objavi.
A2. **Odbijanje** — u koraku 4: reviewer izabere **„Odbij"** i upiše razlog; verzija pređe u **`Odbijen`**, ne objavljuje se.

**Rezultat:** Verzija modela je u stanju **`Odobren`**, **`Odbijen`** ili **`Vraćeno na doradu`**, sa zabeleženom odlukom, autorom i komentarom.



**Naziv:** Objava odobrene verzije lokalno i (opciono) u Pharos katalog
**Akter:** Model reviewer ili Platform administrator (uloga sa dozvolom objave)
**Preduslov:** Postoji verzija modela u stanju **`Odobren`**.

**Osnovni tok:**
1. Ovlašćeni akter na **„Detalji resursa"** odobrene verzije klikne **„Objavi lokalno"**.
2. Sistem prevede verziju u **`Objavljen lokalno`**, učini je vidljivom u lokalnom katalogu prema podešenoj vidljivosti i zabeleži ko je objavio i kada.
3. Za federaciju: akter dodatno klikne **„Objavi u Pharos katalog"**.
4. Sistem kroz **adapter ka Pharos-u** mapira metapodatke modela u Pharos šemu kataloga i pošalje zahtev za objavu; po potvrdi, verzija pređe u **`Objavljen (Pharos)`**.
5. Sistem prikaže **„Detalji resursa"** sa naznakom statusa objave (lokalno / Pharos) i zabeleženim autorom i vremenom.

**Alternativni tokovi:**
A1. **Verzija nije `Odobren`** — u koraku 1: akcije **„Objavi lokalno"** / **„Objavi u Pharos katalog"** su onemogućene.
A2. **Pharos nedostupan ili odbije šemu** — u koraku 4: Sistem ne menja stanje preko **`Objavljen lokalno`**, prijavi grešku federacije i ponudi ponovni pokušaj; lokalna objava ostaje na snazi. Nijedan kvar partnera ne sme da pokvari lokalno stanje.
A3. **Povlačenje iz upotrebe** — kasnije: ovlašćeni akter klikne **„Deprecate"**; verzija pređe u **`Deprecated`** (i dalje dostupna za referencu/lineage, ali označena kao zastarela), a potom opciono u **`Arhiviran`**.

**Rezultat:** Verzija modela je u stanju **`Objavljen lokalno`** ili **`Objavljen (Pharos)`**, vidljiva u odgovarajućem katalogu, sa zabeleženim autorom i vremenom objave.



**Naziv:** Objava leaderboard-a sa verzionisanjem benchmarka
**Akter:** Platform administrator (uloga `platform-admin`)
**Preduslov:** Platform administrator je prijavljen i ima ulogu `platform-admin`; postoje evaluacioni rezultati više modela na istoj verziji benchmarka.

**Osnovni tok:**
1. Platform administrator otvori ekran **„Leaderboard administracija"**.
2. Administrator izabere verziju benchmarka i pregleda rezultate kandidata za prikaz.
3. Administrator izabere koje rezultate uključuje i klikne **„Objavi leaderboard"**.
4. Sistem kreira javno vidljiv leaderboard vezan za tu verziju benchmarka i zabeleži ko ga je objavio i kada.
5. Sistem prikaže objavljeni **„Leaderboard"** sa naznakom verzije benchmarka.

**Alternativni tokovi:**
A1. **Rezultati sa različitih verzija benchmarka** — u koraku 3: Sistem upozori da se mešaju verzije benchmarka i ne dozvoli objavu dok se ne izabere jedna verzija. *(Da li leaderboard sme da meša verzije benchmarka — pretpostavka je da ne sme jer poređenje gubi smisao; potvrditi.)*
A2. **Odustajanje** — pre koraka 4: administrator napusti ekran; leaderboard se ne objavljuje.

**Rezultat:** Objavljen je leaderboard vezan za tačnu verziju benchmarka, javno vidljiv, sa zabeleženim autorom i vremenom objave.


## 6. Upravljanje datasetovima


**Naziv:** Pregled opisa i uzorka javnog dataseta
**Akter:** Anonimni posetilac
**Preduslov:** Dataset je objavljen u katalogu sa vidljivošću `javno`.

**Osnovni tok:**
1. Posetilac otvara SAIFA portal.
2. Posetilac klikne na **„Katalog"** u navigaciji.
3. Sistem prikazuje listu javno dostupnih resursa (modeli, datasetovi, kursevi).
4. Posetilac filtrira po tipu resursa i bira **„Datasetovi"**.
5. Posetilac klikne na naziv dataseta.
6. Sistem otvara stranicu dataseta sa: nazivom, opisom, metapodacima (licenca, jezik, domen, veličina), i tabelarnim prikazom prvih N redova.
7. Posetilac čita opis i pregleda uzorak podataka.

**Alternativni tokovi:**
A1. **Dataset nije javan** — na koraku 5: sistem ne prikazuje dataset u listi; posetilac ne može ni da dospe do stranice direktnim URL-om (vraća se poruka „resurs nije dostupan").
A2. **Posetilac pokušava preuzimanje** — na koraku 7: dugme **„Preuzmi"** je onemogućeno ili skriveno; sistem prikazuje poruku „Prijavite se da biste preuzeli ovaj dataset".
A3. **Dataset nema preview** — na koraku 6: uzorak podataka se ne prikazuje (npr. binarni format koji se ne može renderovati); sistem prikazuje samo opis i metapodatke sa napomenom „Preview nije dostupan za ovaj format".

**Rezultat:** Posetilac je pregledao opis i uzorak dataseta bez kreiranja naloga i bez preuzimanja podataka.



**Naziv:** Uređivanje metapodataka ili vidljivosti objavljenog dataseta
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen i vlasnik je dataseta koji je već objavljen u katalogu.

**Osnovni tok:**
1. Korisnik otvara stranicu svog dataseta u katalogu.
2. Korisnik klikne **„Uredi"**.
3. Sistem prikazuje formu popunjenu trenutnim vrednostima (naziv, opis, licenca, jezik, domen, vidljivost).
4. Korisnik menja željena polja.
5. Korisnik klikne **„Sačuvaj"**.
6. Sistem ažurira zapis u katalogu i prikazuje poruku „Izmene su sačuvane".

**Alternativni tokovi:**
A1. **Promena vidljivosti na javno** — na koraku 4: ako dataset prethodno nije bio javan, sistem upozorava „Ovaj dataset će biti vidljiv svim korisnicima platforme. Nastavi?"; korisnik potvrđuje → tok se nastavlja od koraka 5.
A2. **Promena vidljivosti zahteva review** — na koraku 5: ako je dataset označen kao osetljiv ili sektorski, sistem ne primenjuje promenu odmah nego šalje zahtev data administratoru na odobrenje; korisnik vidi poruku „Zahtev je poslat na pregled". → vidi: *Odobravanje objave osetljivog dataseta*
A3. **Obavezno polje obrisano** — na koraku 5: sistem blokira čuvanje i označava polje koje mora biti popunjeno.
A4. **Korisnik odustaje** — na bilo kom koraku: klikne **„Otkaži"**; sistem odbacuje izmene, dataset ostaje sa prethodnim vrednostima.
A5. **Korisnik nema pravo uređivanja** — na koraku 2: dugme **„Uredi"** nije prikazano; direktan URL ka formi vraća grešku „Nemate dozvolu za uređivanje ovog resursa".

**Rezultat:** Metapodaci dataseta su ažurirani u katalogu; nova vidljivost se odmah primenjuje (ili čeka odobrenje u slučaju A2).



**Naziv:** Upload dataseta
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen i ima dovoljno storage kvote za veličinu fajla koji upload-uje.

**Osnovni tok:**
1. Korisnik otvara **„Katalog"** i klikne **„Dodaj resurs → Dataset"**.
2. Sistem prikazuje formu za novi dataset.
3. Korisnik unosi naziv, opis, licencu, jezik i domen.
4. Korisnik bira vidljivost: **„Javno / Privatno / Tim"**.
5. Korisnik prevlači fajl ili klikne **„Odaberi fajl"** i bira lokalni fajl.
6. Sistem prikazuje progres upload-a.
7. Upload se završava; sistem upisuje dataset u MinIO i kreira zapis u katalogu.
8. Sistem prikazuje stranicu novokreiranog dataseta sa unetim metapodacima i pregledom uzorka (ako je format podržan).

**Alternativni tokovi:**
A1. **Nedovoljna kvota** — na koraku 6: sistem prekida upload i prikazuje poruku „Nedovoljna storage kvota"; korisnik se upućuje na stranicu kvota.
A2. **Dataset s istim nazivom već postoji** — na koraku 7: sistem prikazuje grešku „Dataset s ovim nazivom već postoji u vašem nalogu"; korisnik menja naziv ili odustaje.
A3. **Prekinut upload** — na koraku 6: veza se prekida; sistem zadržava delimično otpremljeni fajl i prikazuje dugme **„Nastavi upload"** kada korisnik ponovo dođe na stranicu.
A4. **Obavezno polje nije popunjeno** — na koraku 5: sistem onemogućava prelaz na sledeći korak i označava prazna obavezna polja (naziv, licenca).
A5. **Korisnik odustaje** — na bilo kom koraku: klikne **„Otkaži"**; sistem briše privremene podatke, katalog ostaje nepromenjen.

**Rezultat:** Dataset je sačuvan u MinIO, vidljiv u katalogu prema odabranoj vidljivosti, sa popunjenim metapodacima i dostupnim pregledom uzorka.



**Naziv:** Preuzimanje javnog dataseta
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen. Dataset je objavljen u katalogu sa vidljivošću `javno`.

**Osnovni tok:**
1. Korisnik otvara stranicu dataseta u katalogu.
2. Korisnik klikne **„Preuzmi"**.
3. Sistem generiše privremeni download link (presigned URL) i preuzimanje počinje.
4. Korisnik dobija fajl lokalno.

**Alternativni tokovi:**
A1. **Korisnik nije prijavljen** — na koraku 2: dugme **„Preuzmi"** nije prikazano; sistem prikazuje poruku „Prijavite se da biste preuzeli ovaj dataset".
A2. **Dataset je uklonjen ili nije dostupan** — na koraku 3: sistem prikazuje grešku „Dataset trenutno nije dostupan za preuzimanje".

**Rezultat:** Korisnik je preuzeo fajl dataseta lokalno. Sistem je zabeležio preuzimanje u audit logu.



**Naziv:** Odobravanje objave osetljivog dataseta
**Akter:** Data administrator
**Preduslov:** Prijavljen korisnik je uploadovao dataset označen kao osetljiv; dataset čeka pregled i nije vidljiv u katalogu.

**Osnovni tok:**
1. Data administrator otvara **„Pregled na čekanju"** u administratorskom panelu.
2. Sistem prikazuje listu dataseta koji čekaju odobrenje.
3. Data administrator klikne na naziv dataseta.
4. Sistem prikazuje metapodatke (naziv, opis, licenca, domen, sektor), informacije o podnosiocu i priloženu GDPR saglasnost.
5. Data administrator pregleda sadržaj i procenjuje usklađenost.
6. Data administrator klikne **„Odobri"**.
7. Sistem objavljuje dataset u katalogu prema vidljivosti koju je postavio podnosilac i obaveštava ga.

**Alternativni tokovi:**
A1. **Dataset ne zadovoljava uslove** — na koraku 6: data administrator klikne **„Odbij"**, unosi obrazloženje i šalje; sistem vraća dataset u stanje **`Odbijen`**, uklanja ga iz reda čekanja i obaveštava podnosioca sa obrazloženjem.
A2. **Nedostaje GDPR saglasnost** — na koraku 4: dokument saglasnosti nije priložen; data administrator klikne **„Zatraži dopunu"**, unosi šta nedostaje; sistem obaveštava podnosioca i dataset ostaje u stanju **`Na čekanju`**.
A3. **Data administrator treba više vremena** — na koraku 6: ostavlja dataset na čekanju bez akcije; status ostaje nepromenjen.

**Rezultat:** Dataset je objavljen u katalogu (ili odbijen sa obrazloženjem), podnosilac je obavešten, odluka je zabeležena u audit logu.


## 7. Kolaboracija i timski rad


**Naziv:** Kreiranje projekta
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen.

**Osnovni tok:**
1. Korisnik klikne **„Novi projekat"** u navigaciji ili na svojoj početnoj strani.
2. Sistem prikazuje formu za novi projekat.
3. Korisnik unosi naziv i opis projekta.
4. Korisnik bira vidljivost: **„Privatno / Tim / Javno"**.
5. Korisnik klikne **„Kreiraj"**.
6. Sistem kreira projekat i otvara njegovu stranicu.

**Alternativni tokovi:**
A1. **Naziv već postoji** — na koraku 5: sistem prikazuje grešku „Projekat s ovim nazivom već postoji"; korisnik menja naziv.
A2. **Obavezno polje nije popunjeno** — na koraku 5: sistem blokira kreiranje i označava prazno polje.
A3. **Korisnik odustaje** — na bilo kom koraku: klikne **„Otkaži"**; projekat se ne kreira.

**Rezultat:** Projekat je kreiran, korisnik je automatski postavljen kao vlasnik i jedini član; projekat je vidljiv prema odabranoj vidljivosti.



**Naziv:** Pozivanje člana u projekat
**Akter:** Prijavljen korisnik (vlasnik ili co-owner projekta)
**Preduslov:** Korisnik je prijavljen i vlasnik je ili co-owner projekta.

**Osnovni tok:**
1. Korisnik otvara stranicu projekta i klikne na tab **„Članovi"**.
2. Korisnik klikne **„Pozovi člana"**.
3. Sistem prikazuje polje za pretragu korisnika.
4. Korisnik unosi ime ili email kolege i bira ga iz rezultata.
5. Korisnik bira ulogu: **„Member / Co-owner"**.
6. Korisnik klikne **„Pošalji pozivnicu"**.
7. Sistem šalje pozivnicu pozvanom korisniku i prikazuje je u listi članova kao „na čekanju".
8. Kolega prihvata pozivnicu i dobija pristup projektu.

**Alternativni tokovi:**
A1. **Korisnik nije pronađen** — na koraku 4: pretraga ne vraća rezultate; korisnik proverava unos ili odustaje.
A2. **Kolega je već član projekta** — na koraku 6: sistem prikazuje grešku „Ovaj korisnik je već član projekta".
A3. **Kolega odbija pozivnicu** — na koraku 8: status u listi članova se menja u „odbijeno"; vlasnik projekta dobija obaveštenje.
A4. **Pozivnica ističe** — na koraku 8: kolega nije reagovao u definisanom roku; pozivnica ističe i vlasnik može da pošalje novu.
A5. **Korisnik nema pravo pozivanja** — na koraku 2: dugme **„Pozovi člana"** nije prikazano.

**Rezultat:** Kolega je dodat kao član projekta sa odabranom ulogom i ima pristup svim resursima projekta prema toj ulozi.



**Naziv:** Uklanjanje člana iz projekta
**Akter:** Prijavljen korisnik (vlasnik ili co-owner projekta)
**Preduslov:** Korisnik je prijavljen i vlasnik je ili co-owner projekta. Projekat ima najmanje jednog člana kojeg može ukloniti.

**Osnovni tok:**
1. Korisnik otvara stranicu projekta i klikne na tab **„Članovi"**.
2. Korisnik pronalazi člana kojeg želi da ukloni.
3. Korisnik klikne **„Ukloni"** pored imena člana.
4. Sistem traži potvrdu: „Da li ste sigurni da želite da uklonite ovog člana?"
5. Korisnik klikne **„Potvrdi"**.
6. Sistem uklanja člana i oduzima mu pristup svim resursima projekta.

**Alternativni tokovi:**
A1. **Korisnik uklanja samog sebe** — na koraku 3: ako je korisnik jedini vlasnik, sistem blokira akciju i prikazuje poruku „Ne možete napustiti projekat dok ste jedini vlasnik; najpre dodelite vlasništvo drugom članu."
A2. **Korisnik odustaje** — na koraku 4: klikne **„Otkaži"**; član ostaje u projektu.
A3. **Korisnik nema pravo uklanjanja** — na koraku 3: dugme **„Ukloni"** nije prikazano.

**Rezultat:** Član je uklonjen iz projekta i više nema pristup njegovim resursima; uklanjanje je zabeleženo u audit logu.


## 8. Edukacija i kursevi


**Naziv:** Pregled kurseva i learning pathova
**Akter:** Anonimni posetilac
**Preduslov:** Nema — stranica je javno dostupna bez prijave.

**Osnovni tok:**
1. Posetilac otvara SAIFA portal.
2. Posetilac klikne na **„Kursevi"** u navigaciji.
3. Sistem prikazuje listu dostupnih kurseva sa nazivom, kratkim opisom, nivoom i trajanjem.
4. Posetilac filtrira ili sortira kurseve (po nivou, sektoru, jeziku).
5. Posetilac klikne na naziv kursa.
6. Sistem prikazuje stranicu kursa: opis, syllabus, preduslov znanja, trajanje, jezik i da li kurs ima hands-on lab.

**Alternativni tokovi:**
A1. **Posetilac pokušava da se upiše** — na koraku 6: klikne **„Upiši se"**; sistem ga preusmerava na stranicu za prijavu.
A2. **Nema dostupnih kurseva** — na koraku 3: sistem prikazuje poruku „Trenutno nema dostupnih kurseva."

**Rezultat:** Posetilac je pregledao ponudu kurseva i detalje odabranog kursa bez prijavljivanja.



**Naziv:** Upis na kurs
**Akter:** Student
**Preduslov:** Student je prijavljen; kurs je u statusu **`Aktivan`** i vidljiv studentima.

**Osnovni tok:**
1. Student klikne na **„Kursevi"** u navigaciji.
2. Sistem prikazuje listu dostupnih kurseva; student bira kurs i otvara njegovu stranicu.
3. Student čita detalje kursa (opis, syllabus, preduslov znanja, trajanje).
4. Student klikne **„Upiši se"**.
5. Sistem dodaje studenta na kurs i prikazuje potvrdu „Uspešno ste upisani na kurs".

**Alternativni tokovi:**
A1. **Kurs je popunjen** — na koraku 4: kurs je dostigao maksimalan broj polaznika; sistem prikazuje poruku „Kurs je popunjen" i onemogućava upis.
A2. **Student je već upisan** — na koraku 4: sistem prikazuje da je student već upisan i umesto **„Upiši se"** nudi **„Nastavi kurs"**. → vidi: *Praćenje napretka na kursu*
A3. **Kurs zahteva preduslov koji student nema** — na koraku 4: sistem blokira upis uz poruku koji preduslovni kurs ili znanje nedostaje.

**Rezultat:** Student je upisan na kurs; kurs se pojavljuje u njegovoj listi upisanih kurseva i može pratiti napredak i otvarati lab vežbe.



**Naziv:** Praćenje napretka na kursu
**Akter:** Student
**Preduslov:** Student je prijavljen i upisan na kurs.

**Osnovni tok:**
1. Student klikne na **„Kursevi"** u navigaciji.
2. Sistem prikazuje listu kurseva na koje je student upisan.
3. Student klikne na naziv kursa.
4. Sistem prikazuje stranicu kursa sa listom modula i statusom svakog (završen / u toku / nije počet) i ukupnim procentom završenosti.
5. Student klikne na modul da nastavi rad.

**Alternativni tokovi:**
A1. **Kurs je završen** — na koraku 4: sistem prikazuje poruku „Kurs je završen" i dugme **„Preuzmi sertifikat"**.
A2. **Student nema upisanih kurseva** — na koraku 2: sistem prikazuje poruku „Nemate upisanih kurseva" i link ka listi dostupnih kurseva.

**Rezultat:** Student vidi trenutni napredak na kursu i može da nastavi od mesta gde je stao.



**Naziv:** Otvaranje interaktivne Jupyter lab sveske
**Akter:** Student
**Preduslov:** Student je prijavljen, upisan na kurs i kurs sadrži lab vežbu sa Jupyter sveskom.

**Osnovni tok:**
1. Student otvara stranicu kursa i klikne na lab vežbu.
2. Sistem prikazuje opis vežbe i dugme **„Pokreni svesku"**.
3. Student klikne **„Pokreni svesku"**.
4. Sistem pokreće JupyterHub sesiju sa unapred konfiguriranim resursima (CPU, RAM, GPU po limitu koji je postavio predstavnik institucije) i učitava svesku.
5. Sistem montira dataset vežbe kao read-only i prikazuje ga u fajl pretraživaču sveske.
6. Student dobija poruku „Okruženje je spremno" i počinje rad u svesci.

**Alternativni tokovi:**
A1. **Nema slobodnih resursa** — na koraku 4: sistem ne može da pokrene sesiju jer je pool popunjen; student dobija poruku „Resursi trenutno nisu dostupni, pokušajte ponovo kasnije."
A2. **Sesija je već aktivna** — na koraku 3: sistem detektuje postojeću aktivnu sesiju za ovog studenta i nudi **„Nastavi sesiju"** umesto pokretanja nove.
A3. **Student je neaktivan** — tokom rada: sistem detektuje neaktivnost, upozorava studenta i gasi sesiju nakon isteka vremena; radni direktorijum ostaje sačuvan.

**Rezultat:** Student ima aktivnu Jupyter sesiju sa učitanom sveskom, pristupom datasetu i konfiguriranim AI factory pristupom u okviru limita nastavnog projekta.



**Naziv:** Pokretanje unapred konfiguriranog AI factory posla u lab vežbi
**Akter:** Student
**Preduslov:** Student je prijavljen, ima aktivnu Jupyter sesiju u okviru lab vežbe, i vežba sadrži unapred konfigurisan SLURM šablon.

**Osnovni tok:**
1. Student izvršava ćeliju u svesci koja pokreće AI factory posao.
2. Sistem šalje posao na PARADOX/ITE koristeći zaključani SLURM šablon (student ne vidi niti menja SLURM skriptu).
3. Sistem prikazuje u svesci status posla: „Posao u redu čekanja".
4. Status se menja u „Posao se izvršava".
5. Posao se završava; sistem preuzima rezultate u radni direktorijum sveske.
6. Status se menja u „Završeno"; student izvršava sledeću ćeliju i vidi rezultate direktno u svesci.

**Alternativni tokovi:**
A1. **Posao ne uspe** — na koraku 5: sistem prikazuje u svesci status „Greška" i poruku iz SLURM loga; student ne može da menja šablon već se obraća mentoru.
A2. **Posao čeka predugo** — na koraku 3: student zatvori svesku; posao nastavlja da se izvršava na klasteru; rezultati su dostupni kada student ponovo otvori svesku.
A3. **Student prekoračuje dozvoljene resurse** — na koraku 2: sistem odbija slanje posla i prikazuje poruku „Zahtev premašuje dozvoljene resurse za ovu vežbu."

**Rezultat:** Posao je uspešno izvršen na klasteru, rezultati su učitani u svesku i student može da nastavi vežbu.



**Naziv:** Preuzimanje sertifikata o završetku kursa
**Akter:** Student
**Preduslov:** Student je prijavljen i završio sve module i lab vežbe kursa.

**Osnovni tok:**
1. Sistem detektuje da je student završio poslednji modul i prikazuje poruku „Čestitamo, završili ste kurs!"
2. Sistem prikazuje dugme **„Preuzmi sertifikat"**.
3. Student klikne **„Preuzmi sertifikat"**.
4. Sistem generiše PDF sertifikat sa imenom studenta, nazivom kursa, datumom i potpisom SAIFA platforme.
5. Student preuzima PDF lokalno.

**Alternativni tokovi:**
A1. **Nisu svi moduli završeni** — na koraku 2: dugme nije prikazano; student vidi koji moduli još nisu završeni.
A2. **Student traži sertifikat naknadno** — student se vraća na stranicu završenog kursa i klikne **„Preuzmi sertifikat"**; tok se nastavlja od koraka 4.

**Rezultat:** Student ima PDF sertifikat o završetku kursa.



**Naziv:** Kreiranje kursa i unos materijala
**Akter:** Mentor / instruktor
**Preduslov:** Mentor je prijavljen i ima pravo kreiranja kurseva.

**Osnovni tok:**
1. Mentor klikne **„Novi kurs"** na stranici kurseva.
2. Sistem prikazuje formu za novi kurs.
3. Mentor unosi naziv, opis, nivo, jezik, sektor i preduslov znanja.
4. Mentor klikne **„Kreiraj"**.
5. Sistem kreira kurs u statusu **`Nacrt`** i otvara stranicu za uređivanje sadržaja.
6. Mentor dodaje module jedan po jedan: za svaki unosi naziv i dodaje materijale (tekst, PDF, video link).
7. Mentor dodaje lab vežbu unutar modula: bira dataset iz kataloga i prilaže Jupyter svesku.
8. Mentor klikne **„Objavi kurs"**.
9. Sistem menja status kursa u **`Aktivan`** i kurs postaje vidljiv studentima.

**Alternativni tokovi:**
A1. **Mentor objavljuje kurs bez lab vežbe** — na koraku 8: kurs nema priloženu svesku; sistem upozorava „Kurs ne sadrži lab vežbu. Da li želite da objavite bez nje?"; mentor potvrđuje i tok se nastavlja.
A2. **Odabrani dataset je uklonjen iz kataloga** — na koraku 7: sistem prikazuje upozorenje „Dataset više nije dostupan"; mentor bira drugi dataset.
A3. **Mentor čuva nacrt za kasnije** — na koraku 6 ili 7: klikne **„Sačuvaj nacrt"**; kurs ostaje u statusu **`Nacrt`** i nije vidljiv studentima.
A4. **Obavezno polje nije popunjeno** — na koraku 4 ili 8: sistem blokira akciju i označava prazno polje.

**Rezultat:** Kurs je objavljen, vidljiv studentima na stranici kurseva, sa svim modulima, materijalima i lab vežbama.



**Naziv:** Praćenje napretka studenata
**Akter:** Mentor / instruktor
**Preduslov:** Mentor je prijavljen, kurs je aktivan i najmanje jedan student je upisan.

**Osnovni tok:**
1. Mentor otvara stranicu svog kursa i klikne na tab **„Napredak studenata"**.
2. Sistem prikazuje tabelu: red po studentu, kolona po lab vežbi, sa statusom svake (završeno / u toku / nije početo).
3. Mentor klikne na ime studenta.
4. Sistem prikazuje detaljan pregled: koje module i labove je završio, kada, i koliko pokušaja je imao.

**Alternativni tokovi:**
A1. **Nema upisanih studenata** — na koraku 2: sistem prikazuje poruku „Još uvek nema upisanih studenata."
A2. **Mentor želi da kontaktira studenta** — na koraku 4: klikne **„Pošalji poruku"**; sistem otvara formu za slanje obaveštenja studentu.

**Rezultat:** Mentor ima pregled napretka svih studenata po modulima i lab vežbama.



**Naziv:** Konfiguracija resource limita za studente
**Akter:** Predstavnik akademske institucije
**Preduslov:** Predstavnik je prijavljen i institucija ima dodeljenu kvotu od strane platform administratora.

**Osnovni tok:**
1. Predstavnik otvara **„Upravljanje institucijom"** u svom panelu.
2. Predstavnik klikne na tab **„Resource limiti"**.
3. Sistem prikazuje trenutne limite za studentski profil (CPU, RAM, GPU, maksimalno trajanje notebook sesije).
4. Predstavnik menja vrednosti u okviru ukupne kvote institucije.
5. Predstavnik klikne **„Sačuvaj"**.
6. Sistem primenjuje nove limite na sve studente institucije.

**Alternativni tokovi:**
A1. **Uneta vrednost premašuje ukupnu kvotu institucije** — na koraku 5: sistem blokira čuvanje i prikazuje poruku „Postavljeni limit premašuje raspoloživu kvotu institucije" uz prikaz dostupnih vrednosti.
A2. **Predstavnik želi različite limite po kursu** — na koraku 2: ova granularnost nije podržana; limiti se postavljaju na nivou cele institucije.

**Rezultat:** Novi resource limiti su primenjeni; svaki student institucije pri sledećem pokretanju notebook sesije dobija resurse u okviru novih vrednosti.


## 9. Administracija, monitoring i usklađenost


**Naziv:** Praćenje statusa i logova inference endpointa
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen i ima najmanje jedan deploy-ovan inference endpoint.

**Osnovni tok:**
1. Korisnik otvara **„Moji endpointi"** u svom panelu.
2. Sistem prikazuje listu endpoint-a sa statusom svakog (aktivan / zaustavljen / greška).
3. Korisnik klikne na naziv endpointa.
4. Sistem prikazuje stranicu endpointa sa: statusom, URL-om, brojem zahteva, prosečnom latencijom i logovima.
5. Korisnik pregleda logove i filtrira po vremenskom opsegu ili nivou (info / upozorenje / greška).

**Alternativni tokovi:**
A1. **Endpoint je u statusu greška** — na koraku 4: sistem prikazuje poruku o grešci i poslednji zapis u logu koji je prethodio grešci.
A2. **Korisnik nema deploy-ovanih endpointa** — na koraku 2: sistem prikazuje poruku „Nemate aktivnih endpointa" i link ka scenariju deploy-a modela. → vidi: *Deploy sopstvenog modela kao inference endpoint*

**Rezultat:** Korisnik ima uvid u trenutni status i logove svog inference endpointa.



**Naziv:** Pregled lične istorije aktivnosti
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen.

**Osnovni tok:**
1. Korisnik otvara **„Moja aktivnost"** u svom panelu.
2. Sistem prikazuje hronološku listu aktivnosti: AI factory poslovi, inference zahtevi i preuzimanja resursa.
3. Korisnik filtrira po tipu aktivnosti ili vremenskom opsegu.
4. Korisnik klikne na zapis da vidi detalje (status, trajanje, potrošeni resursi, naziv resursa).

**Alternativni tokovi:**
A1. **Nema aktivnosti za odabrani filter** — na koraku 3: sistem prikazuje poruku „Nema zabeleženih aktivnosti za odabrane kriterijume."
A2. **Korisnik želi da ponovi AI factory posao** — na koraku 4: klikne **„Ponovi"**; sistem otvara formu za novi posao sa prethodnim parametrima unapred popunjenim. → vidi: *Pokretanje računski zahtevnog posla*

**Rezultat:** Korisnik ima pregled sopstvene istorije aktivnosti na platformi.



**Naziv:** Zahtev za potvrdu usklađenosti resursa sa sektorskim propisima
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen i resurs (model ili dataset) koji želi da koristi postoji u katalogu.

**Osnovni tok:**
1. Korisnik otvara stranicu resursa u katalogu.
2. Korisnik klikne **„Zatraži potvrdu usklađenosti"**.
3. Sistem prikazuje formu: korisnik bira sektor (zdravstvo / javni sektor), unosi svrhu korišćenja i prilaže relevantnu dokumentaciju (npr. interni akt o zaštiti podataka).
4. Korisnik klikne **„Pošalji zahtev"**.
5. Sistem prosleđuje zahtev data administratoru i obaveštava korisnika da je zahtev primljen.
6. Data administrator pregleda zahtev i izdaje potvrdu.
7. Korisnik dobija obaveštenje i preuzima potvrdu iz svog panela.

**Alternativni tokovi:**
A1. **Data administrator odbija zahtev** — na koraku 6: data administrator unosi obrazloženje i odbija; korisnik dobija obaveštenje sa obrazloženjem, bez izdavanja potvrde.
A2. **Nedostaje dokumentacija** — na koraku 6: data administrator traži dopunu; korisnik dobija obaveštenje, dopunjuje dokumentaciju i ponovo podnosi.
A3. **Korisnik odustaje** — na koraku 3: klikne **„Otkaži"**; zahtev se ne kreira.

**Rezultat:** Korisnik je dobio formalnu potvrdu da sme da koristi odabrani resurs u svom sektoru; potvrda je zabeležena u audit logu.



**Naziv:** Raspodela kvote članu organizacije
**Akter:** Predstavnik organizacije
**Preduslov:** Predstavnik je prijavljen i organizacija ima raspoloživu kvotu koja još nije u potpunosti raspodeljena.

**Osnovni tok:**
1. Predstavnik otvara **„Upravljanje organizacijom"** u svom panelu.
2. Predstavnik klikne na tab **„Članovi"**.
3. Sistem prikazuje listu članova sa trenutno dodeljenom i potrošenom kvotom svakog.
4. Predstavnik klikne na ime člana.
5. Sistem prikazuje trenutnu kvotu člana (GPU sati, CPU sati, storage).
6. Predstavnik klikne **„Uredi kvotu"**.
7. Predstavnik unosi nove vrednosti.
8. Predstavnik klikne **„Sačuvaj"**.
9. Sistem primenjuje novu kvotu i obaveštava člana.

**Alternativni tokovi:**
A1. **Uneta vrednost premašuje raspoloživu kvotu organizacije** — na koraku 8: sistem blokira čuvanje i prikazuje poruku „Tražena vrednost premašuje raspoloživu kvotu organizacije" uz prikaz preostalog salda.
A2. **Predstavnik smanjuje kvotu ispod trenutne potrošnje člana** — na koraku 8: sistem upozorava „Član je već potrošio više od nove vrednosti"; predstavnik mora da unese vrednost iznad trenutne potrošnje ili sačeka da potrošnja opadne.
A3. **Predstavnik odustaje** — na bilo kom koraku: klikne **„Otkaži"**; kvota ostaje nepromenjena.

**Rezultat:** Član ima novu kvotu u okviru ukupne kvote organizacije; predstavnik vidi ažurirani saldo raspoložive kvote organizacije.



**Naziv:** Praćenje potrošnje resursa organizacije
**Akter:** Predstavnik organizacije
**Preduslov:** Predstavnik je prijavljen i organizacija ima dodeljenu kvotu.

**Osnovni tok:**
1. Predstavnik otvara **„Upravljanje organizacijom"** u svom panelu.
2. Predstavnik klikne na tab **„Potrošnja"**.
3. Sistem prikazuje dashboard sa ukupnom dodeljenom kvotom, potrošenim i preostalim saldom (GPU sati, CPU sati, storage), po organizaciji i po članu.
4. Predstavnik bira vremenski period (tekući mesec / prethodni mesec / prilagođeni opseg).
5. Sistem ažurira prikaz prema odabranom periodu.

**Alternativni tokovi:**
A1. **Nema potrošnje u odabranom periodu** — na koraku 5: sistem prikazuje poruku „Nema zabeležene potrošnje za odabrani period."
A2. **Predstavnik želi detalje po članu** — na koraku 3: klikne na ime člana; sistem prikazuje njegovu potrošnju po tipu resursa i po poslu.

**Rezultat:** Predstavnik ima pregled ukupne i po-članskoj potrošnji resursa organizacije za odabrani period.



**Naziv:** Dodeljivanje ili izmena kvote organizaciji
**Akter:** Platform administrator
**Preduslov:** Administrator je prijavljen. Organizacija je registrovana i odobrena na platformi.

**Osnovni tok:**
1. Administrator otvara **„Upravljanje organizacijama"** u administratorskom panelu.
2. Administrator pronalazi organizaciju i klikne na njeno ime.
3. Sistem prikazuje stranicu organizacije sa trenutnim kvotama (GPU sati, CPU sati, storage) i potrošnjom.
4. Administrator klikne **„Uredi kvotu"**.
5. Administrator unosi vrednosti za GPU sate, CPU sate i storage.
6. Administrator klikne **„Sačuvaj"**.
7. Sistem primenjuje novu kvotu i obaveštava predstavnika organizacije.

**Alternativni tokovi:**
A1. **Uneta vrednost premašuje raspoloživi kapacitet platforme** — na koraku 6: sistem prikazuje upozorenje „Tražena kvota premašuje trenutno raspoloživi kapacitet" i blokira čuvanje.
A2. **Administrator odustaje** — na koraku 5: klikne **„Otkaži"**; kvota ostaje nepromenjena.
A3. **Na osnovu pristiglog zahteva organizacije** — pre koraka 1: administrator otvara **„Zahtevi za kvotu"** u panelu, pronalazi zahtev organizacije, pregleda obrazloženje i dosadašnju potrošnju, pa nastavlja od koraka 2.
A4. **Administrator odbija zahtev** — na koraku A3: administrator pregleda zahtev i odluči da ga ne odobri; klikne **„Odbij"**, unosi obrazloženje i šalje; sistem obaveštava predstavnika organizacije sa obrazloženjem, zahtev se zatvara, kvota ostaje nepromenjena.

**Rezultat:** Nova kvota je dodeljena organizaciji; predstavnik organizacije može da je raspodeljuje članovima.



**Naziv:** Nadzor metrika platforme u realnom vremenu
**Akter:** Platform administrator
**Preduslov:** Administrator je prijavljen.

**Osnovni tok:**
1. Administrator otvara **„Monitoring"** u administratorskom panelu.
2. Sistem prikazuje dashboard sa metrikama u realnom vremenu: GPU iskorišćenost, redovi čekanja na klasteru, latencija inference servisa, broj aktivnih poslova i potrošnja energije.
3. Administrator pregleda metrike i identifikuje eventualne anomalije.

**Alternativni tokovi:**
A1. **Administrator želi detalje za konkretni klaster** — na koraku 2: bira PARADOX ili ITE iz padajućeg menija; sistem filtrira metrike za odabrani klaster.
A2. **Administrator želi istorijske podatke** — na koraku 2: bira vremenski opseg; sistem prikazuje grafike za odabrani period umesto realnog vremena.
A3. **Metrika premašuje prag** — tokom pregleda: sistem vizuelno označava metriku koja je prešla definisani prag (crvena boja, ikonica upozorenja).

**Rezultat:** Administrator ima uvid u trenutno stanje platforme i može da reaguje na anomalije.



**Naziv:** Postavljanje alarma na pragove iskorišćenosti
**Akter:** Platform administrator
**Preduslov:** Administrator je prijavljen.

**Osnovni tok:**
1. Administrator otvara **„Monitoring"** u administratorskom panelu i klikne na tab **„Alarmi"**.
2. Sistem prikazuje listu postojećih alarma sa statusom (aktivan/neaktivan) i pragovima.
3. Administrator klikne **„Novi alarm"**.
4. Administrator bira metriku (GPU iskorišćenost, red čekanja, latencija, storage, energija).
5. Administrator unosi prag pri kojem se alarm aktivira.
6. Administrator bira kanal obaveštavanja (email, in-app).
7. Administrator klikne **„Sačuvaj"**.
8. Sistem aktivira alarm i prikazuje ga u listi.

**Alternativni tokovi:**
A1. **Administrator uređuje postojeći alarm** — na koraku 2: klikne **„Uredi"** pored alarma; sistem otvara formu sa trenutnim vrednostima; tok se nastavlja od koraka 4.
A2. **Administrator deaktivira alarm** — na koraku 2: klikne **„Deaktiviraj"**; sistem pauzira alarm bez brisanja.
A3. **Administrator briše alarm** — na koraku 2: klikne **„Obriši"**; sistem traži potvrdu; alarm se trajno uklanja.

**Rezultat:** Alarm je aktivan; sistem obaveštava administratora kada odabrana metrika pređe definisani prag.



**Naziv:** Pregled audit loga
**Akter:** Platform administrator
**Preduslov:** Administrator je prijavljen.

**Osnovni tok:**
1. Administrator otvara **„Audit log"** u administratorskom panelu.
2. Sistem prikazuje hronološku listu operacija sa: vremenom, akterom, tipom operacije i resursom na kojem je izvršena.
3. Administrator filtrira po akteru, tipu operacije, resursu ili vremenskom opsegu.
4. Administrator klikne na zapis da vidi detalje operacije.

**Alternativni tokovi:**
A1. **Nema rezultata za zadati filter** — na koraku 3: sistem prikazuje poruku „Nema zabeleženih operacija za odabrane kriterijume."

**Rezultat:** Administrator ima pregled svih operacija na platformi u skladu sa zadatim filterima; zapisi se ne mogu menjati ni brisati.



**Naziv:** Pokretanje DPIA procedure za dataset sa ličnim podacima
**Akter:** Platform administrator
**Preduslov:** Administrator je prijavljen. Dataset je označen kao da sadrži lične podatke (od strane korisnika pri upload-u ili od strane data administratora pri pregledu).

**Osnovni tok:**
1. Administrator otvara **„Upravljanje datasetovima"** u administratorskom panelu.
2. Administrator pronalazi dataset označen zastavicom „sadrži lične podatke" i klikne na njega.
3. Sistem prikazuje detalje dataseta i status DPIA procedure (nije pokrenuta / u toku / završena).
4. Administrator klikne **„Pokreni DPIA"**.
5. Sistem generiše DPIA obrazac popunjen dostupnim metapodacima dataseta (vlasnik, sektor, tip podataka, vidljivost).
6. Administrator pregleda i dopunjuje obrazac (svrha obrade, procena rizika, mere zaštite).
7. Administrator klikne **„Sačuvaj i zaključi"**.
8. Sistem menja status DPIA procedure u „završena" i beleži datum i potpis administratora.

**Alternativni tokovi:**
A1. **DPIA ukazuje na visok rizik** — na koraku 7: administrator označava rizik kao visok; sistem automatski blokira objavu dataseta dok se ne preduzmu dodatne mere zaštite i ne dobije odobrenje.
A2. **Administrator čuva nacrt za kasnije** — na koraku 6: klikne **„Sačuvaj nacrt"**; DPIA ostaje u statusu „u toku", dataset ostaje blokiran za objavu.

**Rezultat:** DPIA procedura je završena i zabeležena u audit logu; dataset može biti objavljen (ili ostaje blokiran u slučaju visokog rizika).



**Naziv:** Konfiguracija politike gašenja neaktivnih GPU sesija
**Akter:** Platform administrator
**Preduslov:** Administrator je prijavljen.

**Osnovni tok:**
1. Administrator otvara **„Podešavanja platforme"** u administratorskom panelu.
2. Administrator klikne na tab **„Politike resursa"**.
3. Sistem prikazuje trenutne vrednosti: vreme neaktivnosti nakon kojeg se sesija gasi, vreme upozorenja pre gašenja.
4. Administrator menja vrednosti.
5. Administrator klikne **„Sačuvaj"**.
6. Sistem primenjuje novu politiku i prikazuje poruku „Politika je ažurirana."

**Alternativni tokovi:**
A1. **Administrator postavlja različite vrednosti po ulozi** — na koraku 4: bira ulogu (Student / Prijavljen korisnik) i unosi vrednosti za svaku posebno; studenti mogu imati kraće vreme neaktivnosti od ostalih korisnika.
A2. **Administrator unosi nevalidnu vrednost** — na koraku 5: sistem blokira čuvanje i prikazuje dozvoljeni opseg vrednosti.

**Rezultat:** Nova politika je aktivna; sistem gasi neaktivne GPU sesije prema novim vrednostima i upozorava korisnike pre gašenja.


## 10. Federacija sa EuroHPC (Pharos, IT4LIA)


**Naziv:** Pristup federisanim resursima sa partnerske platforme
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen putem MyAccessID (EuroHPC identitet). Federisana veza sa Pharos/IT4LIA je aktivna.

**Osnovni tok:**
1. Korisnik otvara **„Katalog"** i filtrira po izvoru → bira **„Pharos"** ili **„IT4LIA"**.
2. Sistem prikazuje resurse sa partnerske platforme sa oznakom porekla.
3. Korisnik klikne na naziv resursa.
4. Sistem prikazuje stranicu resursa sa metapodacima preuzetim sa partnerske platforme.
5. Korisnik klikne **„Koristi resurs"**.
6. Sistem autentifikuje korisnika ka partnerskoj platformi kroz Keycloak identity brokering, bez dodatne prijave.
7. Korisnik dobija pristup resursu.

**Alternativni tokovi:**
A1. **Korisnik nije prijavljen putem MyAccessID** — na koraku 6: sistem prikazuje poruku „Za pristup federisanim resursima potrebna je prijava putem EuroHPC identiteta (MyAccessID)"; korisnik se upućuje na ponovnu prijavu.
A2. **Partnerska platforma nije dostupna** — na koraku 6: sistem prikazuje poruku „Partnerska platforma trenutno nije dostupna"; resurs ostaje vidljiv u katalogu ali nije dostupan.
A3. **Resurs je uklonjen sa partnerske platforme** — na koraku 4: sistem prikazuje poruku „Ovaj resurs više nije dostupan na partnerskoj platformi".

**Rezultat:** Korisnik ima pristup resursu sa partnerske platforme kroz SAIFA portal, bez kreiranja posebnog naloga na toj platformi.



**Naziv:** Izvoz resursa na partnersku platformu
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen putem MyAccessID. Resurs (model ili dataset) je objavljen u SAIFA katalogu sa vidljivošću `javno`. Federisana veza sa Pharos/IT4LIA je aktivna.

**Osnovni tok:**
1. Korisnik otvara stranicu svog resursa u katalogu.
2. Korisnik klikne **„Izvezi na partnersku platformu"**.
3. Sistem prikazuje listu dostupnih partnerskih platformi (Pharos, IT4LIA).
4. Korisnik bira platformu i klikne **„Potvrdi"**.
5. Sistem šalje metapodatke i resurs na odabranu partnersku platformu.
6. Sistem prikazuje poruku „Resurs je uspešno izvezen" i oznaku partnerske platforme na stranici resursa.

**Alternativni tokovi:**
A1. **Resurs nije javan** — na koraku 2: dugme **„Izvezi"** nije prikazano; resurs mora biti javno objavljen pre izvoza.
A2. **Partnerska platforma nije dostupna** — na koraku 5: sistem prikazuje poruku „Partnerska platforma trenutno nije dostupna; pokušajte ponovo kasnije."
A3. **Resurs već postoji na partnerskoj platformi** — na koraku 5: sistem prikazuje poruku „Resurs je već dostupan na ovoj platformi" i nudi opciju **„Ažuriraj"**.

**Rezultat:** Resurs je dostupan na partnerskoj platformi sa oznakom porekla (SAIFA) i vidljiv korisnicima te platforme.



**Naziv:** Konfiguracija automatskog sync-a kataloga sa partnerskim platformama
**Akter:** Platform administrator
**Preduslov:** Administrator je prijavljen. Federisana veza sa Pharos/IT4LIA je aktivna.

**Osnovni tok:**
1. Administrator otvara **„Federacija"** u administratorskom panelu.
2. Administrator bira partnersku platformu (Pharos ili IT4LIA).
3. Sistem prikazuje trenutnu konfiguraciju sync-a: raspored, tip okidača i status poslednjeg sync-a.
4. Administrator klikne **„Uredi konfiguraciju"**.
5. Administrator postavlja zakazani sync (npr. svake noći u 02:00).
6. Administrator uključuje event-driven sync (novi resurs objavljen na partnerskoj platformi okida sync).
7. Administrator klikne **„Sačuvaj"**.
8. Sistem primenjuje konfiguraciju i prikazuje poruku „Konfiguracija je sačuvana."

**Alternativni tokovi:**
A1. **Administrator pokreće ručni sync** — na koraku 3: klikne **„Pokreni sync sada"**; sistem odmah povlači nove i ažurirane resurse sa partnerske platforme i prikazuje izveštaj (koliko resursa dodato, ažurirano, preskočeno).
A2. **Partnerska platforma nije dostupna tokom sync-a** — sistem beleži grešku u logu, ne upisuje ništa delimično u katalog i obaveštava administratora.
A3. **Partnerska platforma vraća neispravne metapodatke** — sistem preskače taj resurs, beleži grešku i nastavlja sa ostalima; administrator vidi listu preskočenih resursa u izveštaju.

**Rezultat:** Automatski sync je konfigurisan; katalog se redovno ažurira resursima sa partnerskih platformi prema postavljenom rasporedu i okidačima.



**Naziv:** Ručni uvoz resursa sa partnerske platforme
**Akter:** Platform administrator
**Preduslov:** Administrator je prijavljen. Federisana veza sa Pharos/IT4LIA je aktivna.

**Osnovni tok:**
1. Administrator otvara **„Federacija"** u administratorskom panelu i klikne na tab **„Uvoz"**.
2. Administrator bira partnersku platformu (Pharos ili IT4LIA).
3. Sistem prikazuje listu resursa dostupnih na partnerskoj platformi koji još nisu uvezeni u lokalni katalog.
4. Administrator pronalazi resurs i klikne **„Uvezi"**.
5. Sistem preuzima metapodatke i fajl resursa sa partnerske platforme.
6. Sistem upisuje resurs u lokalni katalog sa oznakom porekla i prikazuje poruku „Resurs je uspešno uvezen."

**Alternativni tokovi:**
A1. **Resurs već postoji u lokalnom katalogu** — na koraku 4: sistem prikazuje poruku „Ovaj resurs je već uvezen" i nudi opciju **„Ažuriraj"** ako postoji novija verzija na partnerskoj platformi.
A2. **Partnerska platforma nije dostupna** — na koraku 5: sistem prikazuje poruku „Partnerska platforma trenutno nije dostupna; pokušajte ponovo kasnije."
A3. **Metapodaci resursa su neispravni** — na koraku 5: sistem odbija uvoz, prikazuje koja polja nisu ispravna i ne upisuje ništa u lokalni katalog.

**Rezultat:** Resurs sa partnerske platforme je dostupan u lokalnom SAIFA katalogu sa oznakom porekla.
