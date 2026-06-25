**Naziv:** Pozivanje modela (REST API ili Jupyter sveska)
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen preko Keycloak-a i ima ulogu sa pravom na inference; postoji model dostupan korisniku prema ulozi i ABAC atributima, podignut na deljenom serving pool-u (model je poslužen, korisnik ga ne diže sam); korisnik ima način autentikacije za izabranu ulaznu tačku — izdat API ključ za REST, ili aktivnu sesiju za svesku.

**Osnovni tok:**
1. Korisnik izabere ulaznu tačku. Za REST: sa ekrana **„Detalji resursa"** modela očita identifikator modela i URL inference endpointa, a sa ekrana **„Moji API ključevi"** kopira postojeći ključ (ili ga, ako ga nema, izda kroz **„Novi API ključ"**). Za svesku: otvori svesku (podizanje JupyterHub okruženja je zaseban tok) i pozove SAIFA SDK.
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
8. Sistem kreira zapis posla u stanju **„U redu"** i prikaže potvrdu sa identifikatorom posla.

**Alternativni tokovi:**
A1. **Nedostaju obavezna polja** — u koraku 5: dataset ili odredište nisu izabrani; sistem označi prazna polja i ne pokreće posao; forma zadržava već uneto i korisnik se vraća na korak 3.
A2. **Nema pristup modelu ili datasetu** — u koraku 5: uloga ili ABAC atributi ne pokrivaju model ili dataset; sistem odbije pokretanje uz poruku o nedostatku dozvole; posao se ne kreira.
A3. **Nedovoljna kvota** — u koraku 5: preostala kvota (CPU/GPU sati na nekom nivou hijerarhije) ne pokriva procenu posla; sistem odbije pokretanje i prikaže koja kvota i koliko nedostaje; posao se ne kreira i ništa se ne rezerviše.
A4. **Odustajanje** — u koraku 3 ili 4: korisnik napusti formu bez pokretanja; nijedan posao se ne kreira; uneto u formi se ne čuva (za razliku od A1, gde se unos zadržava jer se korisnik vraća da dopuni).

**Rezultat:** Kreiran je batch inference posao u stanju **„U redu"**, vezan za korisnika, model i dataset, sa rezervisanom kvotom (korak 6); posao čeka izvršavanje na Kubernetes/SLURM-u; kreiranje je upisano u audit log.


**Naziv:** Praćenje napretka batch inference posla
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen preko Keycloak-a; postoji bar jedan batch inference posao koji je korisnik pokrenuo i koji je u nekom od stanja izvršavanja (**„U redu"**, **„U toku"**, **„Završen"**, **„Neuspeo"**, **„Prekinut"**).

**Osnovni tok:**
1. Prijavljen korisnik otvori ekran **„Moji poslovi"**.
2. Sistem prikaže listu korisnikovih poslova sa statusom i vremenom pokretanja.
3. Korisnik izabere batch inference posao iz liste.
4. Sistem prikaže **„Detalji posla"**: trenutni status, poziciju u redu (ako je u redu), potrošnju resursa do tada i live log izvršavanja.
5. Korisnik prati osvežavanje statusa i logova dok posao traje.
6. Po završetku, sistem prikaže status **„Završen"** i link ka rezultatima u MinIO.

**Alternativni tokovi:**
A1. **Posao još u redu** — u koraku 4: posao nije počeo da se izvršava; sistem prikaže poziciju u redu i procenu čekanja (ako je dostupna iz SLURM-a/scheduler-a); logovi izvršavanja još ne postoje.
A2. **Posao neuspeo** — u koraku 6: izvršavanje se završilo greškom; sistem prikaže status **„Neuspeo"** sa porukom o grešci i poslednjim logovima; rezultata nema; korisnik može da pokrene posao ponovo direktno iz **„Detalji posla"**, sa istim parametrima (bez ponovnog prolaska kroz formu i biranja dataseta).
A3. **Prekid posla** — u koraku 5: korisnik izabere **„Prekini posao"**; sistem zatraži potvrdu, zaustavi izvršavanje i oslobodi rezervisanu kvotu; posao prelazi u stanje **„Prekinut"**.
A4. **Kašnjenje live log streama** — u koraku 5: prikaz logova zaostaje za stvarnim izvršavanjem; sistem nastavlja da dopunjuje logove kad stignu; status posla ostaje verodostojan jer se čita iz scheduler-a, ne iz log streama.

**Rezultat:** Korisnik je video aktuelni status i logove posla; za završen posao ima link ka rezultatima u MinIO. Pregled ne menja stanje posla (osim ako je u A3 sam prekinuo posao). Rezervisana kvota je oslobođena ili obračunata prema stvarnoj potrošnji kad posao uđe u završno stanje (završen / neuspeo / prekinut).


**Naziv:** Podešavanje rate limitinga (kvote)
**Akter:** Platform administrator
**Preduslov:** Platform administrator je prijavljen preko Keycloak-a i ima dozvolu za administraciju kvota; postoji entitet za koji se limit podešava — institucija (nivo afilijacije), korisnik unutar institucije, ili tip zahteva (`inference`, `HPC submit`, `federacija`).

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


**Naziv:** Deploy sopstvenog modela kao inference endpoint
**Akter:** Prijavljen korisnik
**Preduslov:** Korisnik je prijavljen preko Keycloak-a i ima pravo da deploy-uje modele; model je registrovan i otpremljen u MinIO i nalazi se u stanju **„Odobren"** (`approved`) ili **„Objavljen (lokalno)"** (`published locally`); postoji raspoloživ GPU kapacitet u serving pool-u.
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
A3. **Model nije u dozvoljenom stanju** — u koraku 1: model je u stanju **„Draft"** (`draft`) ili **„Na pregledu"** (`under review`); sistem ne nudi deploy dok model ne bude odobren; tok se prekida.
A4. **Neuspeo deploy** — u koraku 5: serving instanca ne uspe da podigne model (npr. nekompatibilan format težina); sistem prikaže grešku sa logom i ne ostavlja pola-podignut endpoint; korisnik se vraća na korak 2.
A5. **Odustajanje** — u koraku 2 ili 3: korisnik napusti formu; endpoint se ne kreira; nijedan resurs nije zauzet.

**Rezultat:** Model je podignut kao aktivan inference endpoint sa dodeljenim URL-om, vidljiv prema izabranoj vidljivosti (ne vidljiviji od samog modela); zauzet je GPU resurs serving pool-a; deploy je upisan u audit log.


**Naziv:** Zamena verzije modela na endpointu bez prekida
**Akter:** Platform administrator
**Preduslov:** Platform administrator je prijavljen preko Keycloak-a i ima dozvolu za upravljanje serving slojem; postoji aktivan inference endpoint sa tekućom verzijom modela; nova verzija modela je u stanju **„Odobren"** (`approved`) ili **„Objavljen (lokalno)"** (`published locally`) i kompatibilna je sa istim API-jem.
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
