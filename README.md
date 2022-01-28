# Időkép weather integration for HA

Az időkép oldaláról scrape-pel lehet adatokat szenzorokba kinyerni. Az általam használt konfigok ebben a mappában találhatóak.

FONTOS: Az időjárás típusa (napos, felhős, esik, stb.) az ikonokból került visszafejtésre, és a lista NEM TELJES.

FONTOS: Az Időképről nem minden adatot nyertem eddig ki (főleg a típus, hőmérséklet és a csapadék volt fókuszban), így 1-2 helyen egy másik időjárás integrációból veszek dolgokat (pl. szélerősség, légnyomás, stb.), a példámban ez a `weather.otthon` időjárás szenzorból jön.

## Felhasználás

- ha még nincs, telepítsd a [HACS-t](https://hacs.xyz/)
- telepítsd a Multiscrape integrációt HACS-ben
- másold be a 2 sort a `configuration.yaml`-ből a tiédbe (File Editor addon)
- vedd fel a két idokep file-t a `configuration.yaml` mellé
- győződj meg, hogy van egy másik időjárás integráció, amire néhány érték fallback-el (az `idokep_weather.yaml`-ben a `weather.otthon` lecserélendő a sajátodra, vagy azokat a sorokat törölni kell)
