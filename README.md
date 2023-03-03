# Időkép weather integration for HA

Az időkép oldaláról scrape-pel lehet adatokat szenzorokba kinyerni. Az általam használt konfigok ebben a mappában találhatóak.

FONTOS: Az időjárás típusa (napos, felhős, esik, stb.) az ikonokból került visszafejtésre, és a lista NEM TELJES.

FONTOS: Az Időképről nem minden adatot nyertem eddig ki (főleg a típus, hőmérséklet és a csapadék volt fókuszban), így 1-2 helyen egy másik időjárás integrációból veszek dolgokat (pl. szélerősség, légnyomás, stb.), a példámban ez a `weather.otthon` időjárás szenzorból jön.

## Felhasználás

- ha még nincs, telepítsd a [HACS-t](https://hacs.xyz/)
- telepítsd a Multiscrape integrációt HACS-ben
- másold be a sorokat a `configuration.yaml`-ből a tiédbe (File Editor addon)
- vedd fel a plusz yaml file-okat a `configuration.yaml` mellé
- győződj meg, hogy van egy másik időjárás integráció, amire néhány érték fallback-el (az `idokep_weather.yaml`-ben a `weather.otthon` lecserélendő a sajátodra, vagy azokat a sorokat törölni kell)
- a good_morning script egy példa, hogyan használjuk fel az eredményt egy media player-rel, spotify-jal összekötve, és google cloud TTS-t használva (ez a fizetős TTS, de ez ad jobb magyar hangot, és normál kereteken belül még az ingyenes limiten belül lehet maradni)
