## Felhasználókezelés, hitelesítés és jogosultságkezelés

### Felhasználói modell és szerepkörök

A rendszerben 3 fajta felhasználói szerepkört készítettem, ez úgy gondolom alapjaiban megfelelően lefedi egy KKV-nál dolgozó emberek jogosultságait.
A legkevesebb jogosultsággal rendelkező felhasználó a SALES-es, egy kiskereskedelmi vállalkozásban egy ilyen hozzáférésű személy intézi a termékek készletrevételét, szerkesztését, készlet mozgatását, termékek eladását és számlázását, így itt is ezen feladatokhoz lesz jogosultsága egy ilyen felhasználónak.
A következő jogosultság a MANAGER, egy ilyen felhasználó hozzá fog férni mindenhez amit a SALES-es tud, viszont neki már lesz jogosultsága minimum és maximum készleteket beállítani, termék akciókat felvenni, kimutatásokat nézni a készletekről, értékesítésekről.
A legmagasabb szintű jogosultság az ADMIN, egy ilyen felhasználó hozzáfér mindenhez amihez a MANAGER, emellett még tudja módosítani a beállításokban a vállalkozás adatait, illetve a /users oldalon tud felvenni további felhasználókat, illetve tudja szerkeszteni a jelenlegieket. Fel tud függeszteni felhasználókat, illetve a kétfaktoros hitelesítésüket is vissza tudja állítani, ha a felhasználó telefont cserél például.
Mivel ez egy cég rendszere lenne így az, hogy a felhasználók maguktól tudnak regisztrálni az nem működhetne, emiatt alakítottam a rendszert így, hogy csak a megfelelő emberek tudnak felvenni új felhasználókat, és emiatt van a rendszer indulásakor egy alap ADMIN felhasználó.

### Felhasználók létrehozása és az email-alapú aktiváció

Amikor egy admin létrehoz egy felhasználót, azt a személyt a megadott email címen értesíti a rendszer és elkezdheti az első bejelentkezését.
Egy felhasználó felvételéhez csak email címet, felhasználónevet és egy jogosultság típust kell megadni. Jelszót ekkor beállítani nem kell, mivel a rendszer majd az email cím ellenőrzése után kérni fogja a felhasználót, hogy állítson be sajátot.
Az email cím ellenőrzése úgy működik, hogy a felhasználó a felületen beírja az email címét, majd ha ott tart a felhasználó a folyamatban akkor a felület kérni fogja, hogy kérjen magának egy egyszer használatos jelszót emailben, ilyenkor a backend generál egy véletlenszerű karakter sort, UUID-t, és azt kiküldi emailben. Ha a felhasználó ezt a felületen beütötte akkor tudja utána az új tartós jelszavát beállítani.
Ezt az emailben kapott karaktersorozatot a backend 30 perces érvényességre állítja be, azaz ezután újat kell igényelnie a felhasználónak ha nem sikerült validálnia.

### Jelszóbeállítása és a fiók aktiválása

Regisztráció után a felhasználónak nem lesz azonnal jelszava, míg végig nem megy az email cím ellenőrzésen és be nem állítja magának a megadott szabályok alapján. Ez azért fontos, mert így meg lehet különböztetni felhasználókat akik még nem mentek végig a hitelesítésen és egy jelszó beállítása regisztrációkor ami csak addig érvényes míg a felhasználó nem hitelesíti az email címét egészen felesleges lenne.
A jelszó-t a felhasználónak beállításkor 2x kell megadnia, a két jelszónak egyeznie kell és a jelszónak ezeknek a feltételeknek kell megfelelnie: a jelszónak legalább 8 karakter hosszúnak kell lennie, kell tartalmaznia legalább egy nagybetűt, egy kisbetűt, egy számot, egy speciális karaktert, nem lehet benne 3 karakternél hosszabb ismétlődés (aaa, 111, stb.), illetve nem lehetnek benne egyszerű sorozatok sem (például 1234, abcd, stb). 
A jelszót egy HMAC-SHA256 nevű algoritmussal hasheli a backend még mielőtt az adatbázisba mentené, így a következő bejelentkezéskor csak a kapott jelszót hasheljük ugyanezzel a módszerrel és összehasonlítjuk, hogy az így hashelt és a tárolt adat egyezik-e.

### Bejelentkezési folyamat

A bejelentkezési folyamat első lépésében a felhasználó beírja az email címét, majd innen 3 lehetőség van ami alapján a program tovább viszi a felhasználót. Az első lehetőség, hogy nem ismeri a megadott email címet és ilyenkor tovább engedi a bejelentkezni kivánó embert a jelszó megadása felületre, ahol az idők végezetéig próbálkozhat a felhasználó, mivel nem létezik a fiókja. Erre azért van szükség mivel ha az első lépésnél megmondanánk, hogy nem létezik az adatbázisban az email cím akkor az illetéktelen, a rendszerbe betörni kívánó rosszakarók munkáját könnyítenénk meg.
Második lehetőség, a felhasználó létezik a rendszerben de még nem hitelesítette az email címét, ekkor elvisszük a bejelentkezőt a felületre ahol tud igényelni egy 30 percig érvényes egyszer használatos jelszót amit az email címére küldünk ki. Ennek hitelesítése után meg kell adnia az új jelszavát. Ha ez is sikeres akkor be kell állítania a fiókjához kötelezően tartozó két faktoros hitelesítést. Ha beállította és sikeresen meg is adta az első 2FA 6 karakterű kódját, akkor beengedjük a felületre.
A harmadik lehetőség, hogy a felhasználó fiókja létezik és hitelesítve is van, ilyenkor a felhasználó az emailje után megadja a jelszavát és hitelesíti ezt a két adatot a backenden. Ha ez a két adat megfelelő akkor kapni fog egy 10 percig érvényes rövid lejáratú kódot aminek az érvényessége alatt meg kell csinálnia a kétfaktoros hitelesítést is. Erre azért van szükség mivel a jelszót nem tároljuk, és két lehetőség lett volna minden adat megfelelő kikerülhetetlen hitelesítésére, az első, hogy az email-t, a jelszót és a két faktoros hitelesítést egyben hitelesíti a backend, vagy ha ezt két körben teszi, egyszer az emailt és a jelszót, majd az emailt és a 6 számjegyű kétfaktoros hitelesító kódot. Ha nincs ez a 10 percig érvényes titkos kód akkor egy illetéktelen betolakodó egy Postman-ből megpróbálhatott volna betörni a rendszerbe ha csak az email és 6 számjegy kell neki a bejutáshoz. Ha a felhasználó időben megcsinálta a kétfaktoros hitelesítést is akkor tovább navigáljuk az irányítópultra.

### Kétfaktoros hitelesítés (2FA)

ide ezeket kell írni:
>2FA célja és előnyei;
A 2FA integráció folyamata a login után;
Mit történik, ha a felhasználó nem teljesíti a 2FA-t a shortLifeToken token lejárta előtt?;
Miért biztonságosabb így a rendszer