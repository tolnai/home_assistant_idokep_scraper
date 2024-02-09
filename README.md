# Időkép weather integration for HA

⚠️ Breaking change: A jobb konfig elkülönítés miatt áttértem arra, hogy package-be rakom az időképes
dolgokat. Emiatt frissült a felhasználás szekció.

Az időkép oldaláról scrape-pel lehet adatokat szenzorokba kinyerni. Az általam használt konfigok ebben a mappában találhatóak.

FONTOS: Az időjárás típusa (napos, felhős, esik, stb.) az ikonokból került visszafejtésre, és a lista NEM BIZTOS, HOGY TELJES.

FONTOS: Az Időképről nem minden adatot nyertem eddig ki (főleg a típus, hőmérséklet és a csapadék volt fókuszban), így 1-2 helyen egy másik időjárás integrációból veszek dolgokat (pl. szélerősség, légnyomás, stb.), a példámban ez a `weather.otthon` időjárás szenzorból jön.

Példa egyaránt felhasználva a rövid (órás) és a hosszú (napi) előrejelzéseket:

![Minta](/assets/sample.png)

## Felhasználás

- ha még nincs, telepítsd a [HACS-t](https://hacs.xyz/)
- telepítsd a Multiscrape integrációt HACS-ben
- másold fel a packages mappás (vagyis tedd az idokep_*.yaml fájlokat egy packages/idokep mappába a HA-ban)
- győződj meg, hogy van egy másik időjárás integráció, amire néhány érték fallback-el (az `idokep_weather.yaml`-ben a `weather.otthon` lecserélendő a sajátodra, vagy azokat a sorokat törölni kell)
- a good_morning script egy példa, hogyan használjuk fel az eredményt egy media player-rel, spotify-jal összekötve, és google cloud TTS-t használva (ez a fizetős TTS, de ez ad jobb magyar hangot, és normál kereteken belül még az ingyenes limiten belül lehet maradni)

A yaml fájlokat egy package-be csoportosítva:

- jobban egymás mellé rakhatók és elkülöníthetők az összetartozó fájlok, illetve
- nem kell külön include-olni megfelelő root elembe, hanem a yaml-ben lehet a gyökér elem típus is (pl. `multiscrape`)

- másold be a sorokat a `configuration.yaml`-ből a tiédbe (File Editor addon)
  - az időképes config sorok ebből hiányoznak
- vedd fel a plusz yaml file-okat a `configuration.yaml` mellé

## Chat-GPT

A repó-ban lévő minta felolvasást én azóta egy ChatGPT-sre cseréltem, amit Node-RED-ből hívok meg. A szenzorok értékeit összegyűjtöm, majd összeállítok hozzá valami ilyesmit:

```javascript
const preText = `Vicces és cuki vagy, Cilinek hívnak, [X városban] vagy.`
const text = `Mondd el a reggeli időjárásjelentést rádiós műsor formátumban legfeljebb 6 mondatban.
[X városban] ${msg.day1Min.replace(/\.[0-9]/, '')} és ${msg.day1Max.replace(/\.[0-9]/, '')}  fok között lesz a hőmérséklet és az idő ${msg.day1CondHu}.
Most ${msg.tempOut.replace(/\.[0-9]/, '')} fok van és az idő ${msg.condition}.
Ma ${msg.nevnap.attributes.message} névnap van.
A végén mondj egy érdekes kapcsolódó ismeretterjesztő tényt.
Ebből is hozzátehetsz valamit, ha fontosnak tűnik: ${msg.forecastDetailsEntity.attributes.reszletek}`;

msg.payload = {
    "model": "gpt-4",
    "temperature": 0.7,
    "max_tokens": 300,
    "messages": [{ "role": "system", "content": preText }, { "role": "user", "content": text }]
};
return msg;
```

Az üzenetet egy sima POST kéréssel küldöm el a `https://api.openai.com/v1/chat/completions` címre, `Authorization: Bearer [token]` és `OpenAI-Organization: [org-id]` header-ekkel.

A választ egyszerű kinyerni, majd felolvastatni egy TTS-sel:

```javascript
msg.payload = msg.payload.choices[0].message.content;

return msg;
```
