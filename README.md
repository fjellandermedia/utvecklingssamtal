# Boka utvecklingssamtal

En statisk bokningssida (`index.html`) för GitHub Pages som pratar med två Power Automate-flöden.

## Länk till vårdnadshavare

Lärarens mejladress skickas med base64url-kodad i parametern `k`. Varje lärare skapar sin länk på

```
https://fjellandermedia.github.io/utvecklingssamtal/lank.html
```

Länken ser ut så här: `https://fjellandermedia.github.io/utvecklingssamtal/?k=Zm9ybmFtbi5lZnRlcm5hbW5AZWR1LnZhcm5hbW8uc2U`

Kodningen gör bara adressen mindre uppenbar. Den skyddar inte på riktigt, eftersom vem som helst kan avkoda base64.

## Flöde 1 – lediga tider (`TIDER_URL`)

Anrop (POST, `application/json`):

```json
{ "mail": "fornamn.efternamn@edu.varnamo.se" }
```

Svar: en array med lediga tider:

```json
[{ "Start": "2026-11-23T05:00:00Z", "Length": "15", "ID": 26972, "Fysiskt": false, "Digitalt": true }]
```

`Start` ska vara i UTC. Sidan visar tiden i svensk tid och döljer tider som redan har passerat.

## Flöde 2 – bokning (`BOKA_URL`)

Så länge `BOKA_URL` är tom körs sidan i testläge och visar bara den JSON som skulle ha skickats.

Anrop (POST, `application/json`):

```json
{
  "ID": 26972,
  "Larare": "fornamn.efternamn@edu.varnamo.se",
  "Barn": "Elsa Andersson",
  "Skolmejl": "elsa.andersson@edu.varnamo.se",
  "Vardnadshavare": "anna.andersson@gmail.com",
  "Motesform": "Digitalt"
}
```

`Motesform` är `"Fysiskt"` eller `"Digitalt"`. Om tiden bara tillåter en mötesform väljs den automatiskt. Om båda är tillåtna får vårdnadshavaren välja.

JSON-schema för triggern "När en HTTP-begäran tas emot":

```json
{
  "type": "object",
  "properties": {
    "ID": { "type": "integer" },
    "Larare": { "type": "string" },
    "Barn": { "type": "string" },
    "Skolmejl": { "type": "string" },
    "Vardnadshavare": { "type": "string" },
    "Motesform": { "type": "string" }
  },
  "required": ["ID", "Larare", "Barn", "Skolmejl", "Vardnadshavare", "Motesform"]
}
```

Svar som sidan förstår:

| Status | Betydelse |
|---|---|
| 200 | Bokad. Sidan visar en tackruta. |
| 409 | Tiden är redan bokad. Sidan hämtar tiderna på nytt. |
| Annat | Felmeddelande. Om svaret innehåller `{"message": "..."}` visas den texten. |

## Bra att veta

- **Webbläsaren anropar flödena direkt (CORS).** Lägg till rubriken `Access-Control-Allow-Origin: *` i Response-åtgärden i båda flödena.
- **Flödesadresserna är offentliga.** URL:en med `sig=` syns i sidans källkod, så vem som helst kan anropa flödena. Låt därför bokningsflödet själv kontrollera att tiden fortfarande är ledig (`Bokad eq 0`) och tillhör rätt lärare innan det bokar.
