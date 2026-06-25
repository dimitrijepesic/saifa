
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


**Naziv:** Registracija organizacije i zahtev za odobrenje
**Akter:** Predstavnik organizacije
**Preduslov:** Predstavnik ima aktivan nalog na platformi (lokalni, eduGAIN ili MyAccessID); nije još uvek pridružen nijednoj organizaciji; zna zvanični naziv, tip i sektor organizacije.

**Osnovni tok:**
1. Predstavnik se prijavi na SAIFA portal i na početnoj stranici ili u meniju naloga klikne **„Registruj organizaciju"**.
2. Prikaže se formular za registraciju; predstavnik popuni obavezna polja: naziv organizacije, tip (akademska institucija / istraživačka institucija / kompanija/SME), sektor (zdravstvo / poljoprivreda / jezik i kultura / održivost / opšte), kontakt email organizacije i kratak opis svrhe korišćenja platforme.
3. Opcionalno: predstavnik priloži dokumentaciju (rešenje o registraciji, pismo institucije i sl.) klikom na **„Priloži dokument"**.
4. Predstavnik prihvati uslove korišćenja za organizacije kvačicom i klikne **„Pošalji zahtev"**.
5. Sistem validira unos, kreira organizaciju u statusu **„Na čekanju"** i automatski šalje notifikaciju platform administratoru o novom zahtevu.
6. Portal prikaže potvrdu: „Vaš zahtev je primljen i čeka odobrenje administratora. Obaveštenje ćete dobiti emailom."
7. Predstavnik se vraća na početnu stranicu; u zaglavlju ili bočnom meniju vidi oznaku **„Organizacija: na čekanju"**.

**Alternativni tokovi:**
A1. **Organizacija sa istim nazivom već postoji** — u koraku 5: sistem prikaže upozorenje „Organizacija s ovim nazivom već je registrovana — ako ste član te organizacije, obratite se njenom predstavniku ili platform administratoru"; formular ostaje popunjen, zahtev se ne šalje.
A2. **Obavezno polje nije popunjeno** — u koraku 5: sistem prikaže inline grešku pored praznog polja; predstavnik ispravlja unos bez gubitka ostatka formulara.
A3. **Predstavnik već pripada organizaciji** — u koraku 1: sistem detektuje da nalog ima aktivnu organizacijsku vezu i prikaže poruku „Već ste pridruženi organizaciji [naziv] — ne možete registrovati novu dok god ste njen član"; dugme „Registruj organizaciju" nije dostupno.
A4. **Korisnik odustane** — u bilo kom koraku pre slanja: klikne „Otkaži" ili napusti stranicu; formular se ne čuva, zahtev se ne kreira.

**Rezultat:** U sistemu postoji organizacija u statusu „Na čekanju" sa svim unetim podacima i priloženom dokumentacijom; platform administrator ima notifikaciju o zahtevi i može pristupiti pregledu i odlučivanju; predstavnik čeka emailom potvrdu o ishodu.


**Naziv:** Odobravanje ili odbijanje registracije organizacije
**Akter:** Platform administrator
**Preduslov:** Postoji najmanje jedna organizacija u statusu „Na čekanju"; platform administrator je prijavljen.

**Osnovni tok:**
1. Platform administrator otvori administratorski panel i u bočnom meniju klikne **„Organizacije"** → **„Na čekanju"**.
2. Prikaže se lista zahteva za registraciju; administrator klikne na naziv organizacije da otvori njen detaljan pregled.
3. Pregled prikazuje: naziv, tip, sektor, kontakt email, opis svrhe korišćenja, priloženu dokumentaciju i podatke o nalogu predstavnika koji je podneo zahtev.
4. Administrator pregleda priloženu dokumentaciju klikom na **„Preuzmi dokument"** (ako postoji).
5. Administrator klikne **„Odobri"**; prikaže se dijalog za potvrdu koji traži dodelu početne ukupne kvote organizaciji (CPU sati, GPU sati, prostor); administrator unese vrednosti i klikne **„Potvrdi"**.
6. Sistem postavi status organizacije na **„Aktivna"**, dodeli kvotu, poveže nalog predstavnika sa ulogom Predstavnik organizacije i pošalje mu email o odobrenju.
7. Administrator se vrati na listu; organizacija više ne figuriše u redu „Na čekanju".

**Alternativni tokovi:**
A1. **Administrator odbija zahtev** — u koraku 5: administrator klikne **„Odbij"** umesto „Odobri"; prikaže se dijalog koji traži obavezan razlog odbijanja; administrator unese obrazloženje i klikne **„Potvrdi odbijanje"**; sistem postavi status organizacije na „Odbijeno", pošalje predstavniku email sa razlogom; organizacija ostaje u sistemu kao neaktivna i vidljiva administratoru u arhivi.
A2. **Administrator traži dopunu dokumentacije** — u koraku 4: administrator klikne **„Traži dopunu"**, unese komentar šta nedostaje i klikne **„Pošalji"**; sistem obavesti predstavnika emailom; zahtev ostaje u statusu „Na čekanju — tražena dopuna" dok predstavnik ne pošalje izmenu.
A3. **Administrator odustane** — u bilo kom koraku pre potvrde: klikne „Otkaži" u dijalogu ili napusti stranicu; status organizacije ostaje „Na čekanju", nikakva akcija se ne beleži.
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
A3. **Promena uloge utiče na aktivne resurse** — u koraku 6: sistem detektuje da korisnik ima pokrenute poslove ili aktivne endpointe koji zahtevaju višu ulogu od nove; prikaže se upozorenje „Promena uloge može uticati na aktivne resurse korisnika — da li želite da nastavite?"; administrator bira „Nastavi" ili „Otkaži".
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
6. Sistem postavi status naloga na **„Neaktivan"**, trenutno prisilno prekine sve aktivne sesije korisnika, zaustavi pokrenute poslove i suspenduje aktivne inference endpointe.
7. Korisnik dobija email obaveštenje da je njegov nalog deaktiviran, sa navedenim razlogom.
8. Stranica detalja korisnika se osvežava; pored imena prikazuje se oznaka **„Neaktivan"**.

**Alternativni tokovi:**
A1. **Korisnik je jedini predstavnik aktivne organizacije** — u koraku 5: sistem prikaže blokirajuće upozorenje „Ovaj korisnik je jedini predstavnik organizacije [naziv] — deaktivacija nije moguća dok organizacija nema drugog predstavnika"; administrator mora prvo dodeliti novog predstavnika, pa tek onda deaktivirati nalog.
A2. **Korisnik ima pokrenute HPC poslove** — u koraku 5: dijalog eksplicitno navede broj aktivnih poslova i upozori da će biti zaustavljeni; administrator odlučuje da nastavi ili odustane.
A3. **Administrator odustane** — u koraku 5: klikne „Otkaži" u dijalogu; nalog ostaje aktivan, nikakva akcija se ne beleži.
A4. **Nalog je već neaktivan** — u koraku 3: dugme „Deaktiviraj nalog" nije prikazano; umesto toga prikazuje se **„Aktiviraj nalog"** i datum prethodne deaktivacije.

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
A3. **Administrator uklanja ulogu višeg nivoa dok korisnik ima aktivne resurse** — u koraku 7: sistem prikaže upozorenje sa listom resursa koji zahtevaju višu ulogu (npr. aktivni inference endpointi rezervisani za ML praktičara); administrator bira „Nastavi" ili „Otkaži".
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
7. Sistem trenutno poništi sve Keycloak tokene korisnika, zatvori aktivne sveske i suspenduje aktivne inference endpointe; HPC poslovi koji su već u izvršavanju nastavljaju do završetka (osim ako administrator nije odabrao opciju prisilnog zaustavljanja poslova u dijalogu).
8. Korisnik pri sledećem zahtevu dobija poruku „Vaša sesija je istekla — prijavite se ponovo"; ako je nalog privremeno blokiran, prijava nije moguća.
9. Portal prikaže administratoru potvrdu „Sesija je uspešno prekinuta" i zabeleži incident u audit log.

**Alternativni tokovi:**
A1. **Administrator odlučuje da i zaustavi aktivne HPC poslove** — u koraku 6: u dijalogu postavi kvačicu **„Zaustavi aktivne HPC poslove"**; sistem nakon prekida sesije pošalje signal za zaustavljanje svim poslovima korisnika na PARADOX/ITE; korisnik dobija email obaveštenje o zaustavljenim poslovima.
A2. **Korisnik nema aktivnih sesija** — u koraku 2: korisnik se ne prikazuje na listi aktivnih sesija; administrator može pronaći korisnika u **„Svi korisnici"** i direktno deaktivirati nalog ako je potrebno.
A3. **Administrator želi da opozove sesiju s korisničke stranice, ne iz liste sesija** — u koraku 1: administrator umesto liste sesija ode na **„Korisnici"** → pronađe korisnika → otvori detalje naloga → klikne karticu **„Aktivne sesije"** → klikne **„Prisilno prekini sesiju"**; dalje isto od koraka 6.
A4. **Administrator odustane** — u koraku 6: klikne „Otkaži" u dijalogu; sesija ostaje aktivna, nikakva akcija se ne beleži.

**Rezultat:** Sesija korisnika je prekinuta i svi tokeni poništeni; korisnik ne može nastaviti rad bez ponovne prijave (ili uopšte, ako je nalog blokiran); incident je zabeležen u append-only audit logu sa identitetom administratora, vremenskom oznakom, IP adresom korisnika i navedenim razlogom.


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
A3. **Organizacija je dostigla limit kvote** — u koraku 2: dugme „Dodaj člana" prikazuje se zasivljeno sa napomenom „Dostignut limit članova — zatražite povećanje kvote od administratora"; predstavnik ne može dodati novog člana dok kvota ne bude proširena.
A4. **Predstavnik ne dodeljuje kvotu odmah** — u koraku 5: ostavi polja kvote prazna i klikne „Potvrdi"; član se dodaje bez dodeljene kvote i ne može pokretati poslove dok mu predstavnik naknadno ne dodeli resurse putem **„Uredi kvotu"** na stranici člana.
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
A1. **Član ima aktivne pokrenute HPC poslove** — u koraku 4: dijalog eksplicitno navede broj aktivnih poslova i upozori „Pokrenuti poslovi će nastaviti do završetka, ali novi neće moći da se pokreću"; predstavnik potvrđuje ili odustaje.
A2. **Član je vlasnik zajedničkih resursa tima** — u koraku 4: dijalog prikaže upozorenje „Ovaj korisnik je vlasnik [N] resursa deljenih sa organizacijom — resursi ostaju u katalogu, ali vlasništvo prelazi na vas kao predstavnika"; predstavnik potvrđuje ili odustaje.
A3. **Predstavnik pokušava ukloniti samog sebe** — u koraku 2: opcija „Ukloni iz organizacije" nije dostupna pored sopstvenog imena; predstavnik mora zatražiti od platform administratora da promeni predstavnika pre nego što napusti organizaciju.
A4. **Predstavnik odustane** — u koraku 4: klikne „Otkaži" u dijalogu; lista članova ostaje nepromenjena.

**Rezultat:** Osoba više nije član organizacije; njena dodeljena kvota je vraćena u pool organizacije; nalog joj ostaje aktivan kao Nezavisni korisnik i može se prijavljivati na platformu; uklanjanje je zabeleženo u audit logu sa identitetom predstavnika i vremenskom oznakom.