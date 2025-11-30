## Absztrakt

A szakdolgozatom célja egy olyan készletgazdálkodási rendszer elkészítése volt, ami biztoságos, minden igényt kielégít amire egy KKV-nek szüksége lehet.
Emellett fontosnak találtam, hogy a forráskód birtokában egyetlen parancsra futtatható legyen az egész alkalmazás, továbbfejleszthető legyen és on-premise vagy SaaS megoldással akár értékesíthető is legyen.
Jelenlegi funciói közé tartozik a felhasználók kezelése, termékek, raktárak, partnerek felvétele, kezelése és tárolása. Emellett raktárközi átadásokat, be- és kivéteket lehet készíteni, termék akciókat felvenni.
A bevezezés fejezetben bővebben kifjetem a belső motivációimat egy ilyen alkalmazás lefejlesztésére, illetve az eszközök és módszerek fejezetben bővítetten kifejtem a program jelenlegi funckióit.
A program elkészítéséhez Next.js frontend keretrendszert használtam, Postgresql adatbázis tárolja az adatokat és egy Spring Boot backend alkalmazáson keresztül tud a frontend az adatbázissal kommunikálni. 
Emellett fejlesztés közben git verziókezelést használtam, docker konténerekben futattam az adatbázist és a kódomat, illetve Jira-t használtam a feladatatok követhetősége érdekében. 