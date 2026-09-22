MIT FŐZZEK
Fazeks Csaba
Fürész István
Balázs Dávid

A PROJEKT CÉLJA

A projekt célja egy olyan webes főző- és háztartási készletkezelő alkalmazás létrehozása, amely egy külső szerveren működő PostgreSQL adatbázist használ.

A rendszer nyilvántartja a felhasználó otthon rendelkezésre álló alapanyagait és azok mennyiségét, ezek alapján bevásárlólistát készít, valamint a rendelkezésre álló hozzávalók alapján receptjavaslatokat ad.

A projekt egyik legfontosabb célja nem pusztán egy receptoldal létrehozása, hanem a külső adatbázis használatának szemléltetése. Az alkalmazás minden lényeges adatot adatbázisban tárol, így a kliensoldali felület és a szerveroldali adatkezelés jól elkülöníthető.


A RENDSZER FŐ FUNKCIÓI

- Felhasználói fiók: Regisztráció és bejelentkezés. A felhasználó saját készletéhez és receptadataihoz fér hozzá.
- Alapanyag-leltár: Alapanyag hozzáadása, mennyiség módosítása, törlése és aktuális készlet megjelenítése.
- Mennyiségek kezelése: Az alapanyaghoz mennyiség és mértékegység tartozik, például 500 g liszt vagy 2 db tojás.
- Bevásárlólista: A rendszer az elfogyott vagy kevés alapanyagok alapján bevásárlólistát tud készíteni.
- Leltár frissítése: Bevásárlás után a felhasználó manuálisan frissítheti az otthoni készletet.
- Receptajánló: A korábban megadott és aktuálisan rendelkezésre álló alapanyagok alapján recepteket keres.
- Recept részletei: A recepthez kép, hozzávalólista és elkészítési útmutató tartozik.
- Adatbázis-perzisztencia: A felhasználó adatai és a készlet változásai folyamatosan mentésre kerülnek a külső MySql adatbázisba.
- Adminisztráció: Opcionálisan külön felület biztosítható alapanyagok és receptek karbantartására.


TERVEZETT RENDSZERFELÉPÍTÉS

A projekt webes architektúrára épül. A kliens HTTP/HTTPS kéréseket küld a backendnek, a backend pedig az adatbázissal kommunikál.

Egyszerűsített adatfolyam:

Felhasználó
    ↓
Böngésző
    ↓
HTML/CSS/JavaScript frontend
    ↓
REST API / backend
    ↓
MySql adatbázis
    ↓
Backend → frontend → felhasználó


TECHNOLÓGIAI SPECIFIKÁCIÓ

Elem                    Tervezett technológia
Frontend                HTML5, CSS3, JavaScript
Frontend kommunikáció   Fetch API / JSON
Backend                 Node.js + Express
Adatbázis               MySql
Adatbázis-kapcsolat     MySql/MarinaDB
Szerver                 Linux alapú virtuális gép
Képek                   Helyi tárhely vagy külső képforrás; a recepthez kép-URL tárolható
Verziókezelés           Git/GitHub


FRONTEND FELÉPÍTÉSE

A frontend feladata a felhasználói felület megjelenítése és a backend API-val való kommunikáció. A frontend nem tartalmaz adatbázis-jelszót és nem csatlakozik közvetlenül PostgreSQL-hez.

- index.html – Főoldal, navigáció, bejelentkezési állapot.
- login.html – Bejelentkezési űrlap.
- register.html – Regisztrációs űrlap.
- inventory.html – Otthoni alapanyagok és mennyiségek kezelése.
- shopping.html – Automatikusan összeállított bevásárlólista.
- recipes.html – A rendelkezésre álló alapanyagok alapján ajánlott receptek.
- style.css – Egységes megjelenés, reszponzív elrendezés.
- script.js – API-hívások, DOM-kezelés és kliensoldali logika.


BACKEND FELÉPÍTÉSE

A backend központi szerepet tölt be. Ellenőrzi a beérkező adatokat, kezeli a felhasználói jogosultságokat, végrehajtja az adatbázis-lekérdezéseket, és JSON formátumban válaszol a frontendnek.

- js – A webszerver és Express alkalmazás indítása.
- routes/ – API végpontok csoportosítása.
- controllers/ – Az üzleti műveletek kezelése.
- db/ – PostgreSQL kapcsolat és lekérdezések.
- middleware/ – Hitelesítés és alapvető ellenőrzések.
- public/ – HTML, CSS és JavaScript frontend fájlok.


ADATBÁZIS-TERV

Tábla                  Feladata
users                  Regisztrált felhasználók
ingredients            A rendszerben létező alapanyagok
inventory              Egy adott felhasználó otthoni készlete
recipes                Receptek
recipe_ingredients     Receptekhez szükséges alapanyagok
shopping_lists         Felhasználók bevásárlólistái
shopping_list_items    Egy bevásárlólista tételei

A dokumentum 5. oldalán található adatbázisdiagram további kapcsolatokat is szemléltet:
- favorite_recipes – Felhasználók és kedvenc receptek kapcsolata.
- A recipe_ingredients kapcsolja össze a recepteket az alapanyagokkal.
- Az inventory a felhasználóhoz és az alapanyaghoz kapcsolódik.
- A shopping_list_items a bevásárlólistákat és az alapanyagokat kapcsolja össze.


A RECEPTAJÁNLÓ MŰKÖDÉSE

A receptajánló alapelve egyszerű: a rendszer lekéri a felhasználó aktuális készletét, majd összehasonlítja azt a receptek hozzávalóival. Elsőként azok a receptek jelennek meg, amelyekhez a felhasználónak minden szükséges alapanyaga megvan.

1. Backend lekéri az aktuális inventory adatokat.
2. Backend lekéri a receptekhez tartozó hozzávalókat.
3. A rendszer összehasonlítja a szükséges és rendelkezésre álló mennyiségeket.
4. A teljesen elkészíthető receptek magasabb prioritást kapnak.
5. Azok a receptek is megjelenhetnek, amelyekhez csak 1–2 alapanyag hiányzik.
6. A frontend kártyákon jeleníti meg a recepteket képpel és rövid leírással.


BEVÁSÁRLÓLISTA MŰKÖDÉSE

A bevásárlólista célja, hogy a felhasználónak ne kelljen fejben vezetnie, miből mennyi fogyott el. A rendszer egy egyszerű készletminimum-logika alapján automatikusan létrehozhatja a listát.

Példa:
Ha a felhasználó készletében 0 db tojás van, a rendszer hozzáadja a tojást a bevásárlólistához. Ha 1 db van, de a beállított minimum 6 db, akkor 5 db kerülhet a listára.


FŐ API VÉGPONTOK

POST /api/auth/register
    Új felhasználó létrehozása

POST /api/auth/login
    Bejelentkezés

GET /api/inventory
    Saját készlet lekérése

POST /api/inventory
    Alapanyag hozzáadása

PUT /api/inventory/:id
    Mennyiség módosítása

DELETE /api/inventory/:id
    Alapanyag törlése

GET /api/shopping-list
    Bevásárlólista lekérése

POST /api/shopping-list/generate
    Lista automatikus elkészítése

PUT /api/shopping-list/:id
    Elem kipipálása/módosítása

GET /api/recipes
    Receptek lekérése

GET /api/recipes/recommended
    Készlet alapján ajánlott receptek

GET /api/recipes/:id
    Egy recept részletei


BIZTONSÁGI ALAPELVEK

- A MySql jelszó nem kerülhet a HTML vagy JavaScript fájlokba.
- A backend használjon környezeti változókat az adatbázis-kapcsolati adatokhoz.
- A jelszavakat hash formában kell tárolni, például bcrypt segítségével.
- Az SQL lekérdezéseknél paraméterezett lekérdezéseket kell használni.
- A felhasználó csak a saját inventory és shopping list adatait érheti el.
- Éles környezetben HTTPS használata javasolt.
- Az adatbázis portját nem szükséges közvetlenül kitenni az internetre; célszerű a backendet kontrollált hálózaton keresztül elérhetővé tenni.


MEGVALÓSÍTÁSI TERV

1. Adatbázis létrehozása: MySql adatbázis, táblák, kapcsolatok és kezdeti adatok létrehozása.
2. Backend alapok: Node.js projekt, Express szerver és MySql kapcsolat beállítása.
3. API: Alapanyag-, készlet-, recept- és bevásárlólista végpontok elkészítése.
4. Frontend váz: HTML oldalak, navigáció és alapvető CSS elkészítése.
5. Készletkezelés: Hozzáadás, módosítás, törlés és adatbázisba mentés.
6. Bevásárlólista: Automatikus lista készítése és kipipálható elemek.
7. Receptajánló: Készlet és recept-hozzávalók összehasonlítása.
8. Recept részletek: Kép, hozzávalók és elkészítési útmutató megjelenítése.
9. Bejelentkezés: Regisztráció, login és felhasználói adatok elkülönítése.
10. Tesztelés: API-, adatbázis- és frontend tesztek.
11. Szerverre telepítés: Backend és MySql telepítése a Linux VM-re.
12. Bemutató: Adat útjának bemutatása a böngészőtől a külső MySql adatbázisig.


MINTA FELHASZNÁLÓI FOLYAMAT

Példa egy teljes működési folyamatra:

1. A felhasználó bejelentkezik.
2. Megadja, hogy van otthon 500 g csirkemell, 1 kg rizs és 2 db hagyma.
3. Az adatok bekerülnek a PostgreSQL adatbázisba.
4. A felhasználó megnyitja a receptajánlót.
5. A backend lekéri a készletet és összehasonlítja a receptek hozzávalóival.
6. A rendszer például csirkés-rizses recepteket ajánl.
7. A felhasználó kiválaszt egy receptet.
8. A recept oldalon megjelenik a kép, a hozzávalók és az elkészítés.
9. Ha egy szükséges hozzávaló hiányzik, a rendszer azt megjelölheti a bevásárlólistán.

Bevásárlás után a felhasználó frissíti a készletet, amely ismét elmentésre kerül a MySql adatbázisba.


TOVÁBBFEJLESZTÉSI LEHETŐSÉGEK

- Lejárati idő kezelése és lejáró alapanyagok figyelmeztetése.
- Adagméret alapján automatikus mennyiségszámítás.
- Kedvenc receptek mentése.
- Receptek értékelése.
- Heti étkezési terv készítése.
- Kalória- és tápértékadatok.
- Mobilbarát PWA változat.
- Adminisztrátori receptfeltöltő felület.
- Automatikus MySql biztonsági mentés.
- Jogosultsági szintek.


ÖSSZEFOGLALÁS

A készülő rendszer egy egyszerűen használható, de technikailag jól bemutatható főzős webalkalmazás. A felhasználó kezeli az otthoni alapanyagait, a rendszer ezekből bevásárlólistát készít, majd a meglévő hozzávalók alapján receptet ajánl.

A receptek képpel és elkészítési útmutatóval jelennek meg.

A projekt központi technikai eleme a külső MySql adatbázis. A webes frontend HTML, CSS és JavaScript segítségével kommunikál a backenddel, a backend pedig a MySql adatbázissal. Így a projekt egyetlen alkalmazáson belül szemlélteti a frontend, backend, API, adatbázis, hálózat, virtualizáció és adatmentés kapcsolatát.
