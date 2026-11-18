# Todos-esimerkki – REST → Muunnos → REST

**Kulku:** Ajastus → kommenttien haku (REST GET) → jokaisen kommentin muunnos todoksi → onko todo jo olemassa? → luonti (POST) tai päivitys (PATCH)

Sekä lähde että kohde on julkinen testirajapinta [dummyjson.com](https://dummyjson.com), joten muita palveluita ei tarvita. Arvioitu kesto: noin tunti. Jos olet nopeampi, voit vapaasti tehdä integraatioon omia muutoksia.

Pohja on kansiossa `todos-template` repossa [Adafy/weikio-training-examples](https://github.com/Adafy/weikio-training-examples). Integraatio on pohjassa lähes valmis. Täydennettävät kohdat on merkitty `example.camel.yaml`-tiedostoon tunnisteilla `TODO (1)`–`TODO (4)`.

## Ennen aloitusta

Tarvitset:

- Weikio CLI:n ja kirjautumisen Weikio-projektiin ja -ympäristöön, johon voit julkaista integraatioita
- projektiin salaisuuden nimeltä `training-api-token`; arvo voi olla mikä tahansa, koska dummyjson ei tarkista sitä, mutta älä käytä oikeaa tunnusta, koska se lähetetään julkiseen palveluun

Korvaa alla olevissa komennoissa:

| Paikkamerkki | Merkitys |
|---|---|
| `<nimi>` | oma nimesi tai muu projektissa yksilöllinen tunniste, esim. `liisa`; integraatiosi on silloin `todos-liisa` |
| `<projekti>` | Weikio-projektisi |
| `<ympäristö>` | Weikio-ympäristösi |

## Esimerkkidata

Lähde (`GET {{training-api-url}}/comments?limit=5&skip=251`) palauttaa `comments`-listassa viisi kommenttia, id:t 252–256. Yksi kommentti näyttää tältä:

```json
{
  "id": 252,
  "body": "Such a charming post.",
  "postId": 113,
  "likes": 7,
  "user": { "id": 57, "username": "nathand", "fullName": "Nathan Dixon" }
}
```

Siitä muunnetaan tällainen todo:

```json
{
  "todo": "[nathand] Such a charming post.",
  "userId": 57,
  "completed": true
}
```

Kommentin id tallennetaan `todoId`-propertyyn, ja samalla id:llä tarkistetaan, onko todo jo olemassa (`GET /todos/{todoId}`). Todot 252–254 löytyvät (200) ja päivitetään PATCHilla. Todoja 255–256 ei löydy (404), joten ne luodaan POSTilla osoitteeseen `/todos/add`.

## Vaiheet

1. **Lue ja tarkista pohja:** Kopioi `todos-template` uudeksi kansioksi `todos-<nimi>`. Vaihda `integration.yaml`:ssa nimi `todos-example` nimeksi `todos-<nimi>`. Muuten integraatiosi menee päällekkäin muiden kanssa, eikä niitä voi ajaa yhtä aikaa. Kansion nimi ei vaikuta toimintaan, mutta on hyvä tapa nimetä se integraation mukaan. Lue sitten molemmat reitit ja selvitä: mikä askel hakee kommentit, mikä muuntaa ne, mikä luo tai päivittää todon ja miten olemassaolo tarkistetaan? Tarkistus on omassa `direct:getTodo`-reitissään, jotta sen 404-käsittely pysyy erillään pääkulusta.

2. **Pilko ja muunna:** Haku palauttaa monta kommenttia, ja ne pitää käsitellä yksitellen. Kirjoita kohtaan `TODO (1)` Simple-lauseke, joka palauttaa vastauksen `comments`-kentän listan, esim. `${body["…"]}`. `split` käsittelee silloin jokaisen kommentin omana viestinään. Täydennä kohtaan `TODO (2)` Groovy-rivit, jotka asettavat `todo.completed`-arvoksi `true`, kun kommentilla on vähintään 5 tykkäystä (`body["likes"]`), ja `todo.userId`-arvoksi kommentin käyttäjän id:n (`body["user"]["id"]`). Mallia saat valmiista `todo.todo`-rivistä.

3. **Korjaa luonti ja päivitys:** Kirjoita kohtaan `TODO (3)` `choice`-askeleen ehto, joka on tosi, kun todoa ei löytynyt, eli kun `${exchangeProperty.lookupStatus}` on `404`. Täydennä sitten kohdat `TODO (4)`:
   - Lisää `when`-haaran `to`-askeleelle `parameters`-lohko, joka tekee POSTin osoitteeseen `{{training-api-url}}/todos/add`. Mallia saat `getTodo`-reitin `toD`-askeleesta.
   - Lisää `otherwise`-haaran `toD`-askeleen parametreihin `httpMethod: PATCH`.

4. **Julkaise ja käynnistä:** Validoi, julkaise ja ota integraatio ajoon alla olevilla komennoilla.

   ```powershell
   # 1. Validoi – tarkistaa, että YAML on oikeassa muodossa
   weikio validate -f .\todos-<nimi> --project <projekti>

   # (valinnainen) aja paikallisesti
   weikio dev run .\todos-<nimi> --env <ympäristö>

   # 2. Julkaise – vie integraation Weikioon ja kääntää sen, ei käynnistä
   weikio apply -f .\todos-<nimi> --project <projekti> --env <ympäristö>

   # 3. Deploy – asentaa integraation sitelle ja käynnistää sen
   weikio deploy <projekti>/todos-<nimi> --env <ympäristö>

   # 4. Seuraa
   weikio status <projekti>/todos-<nimi> --env <ympäristö>
   weikio logs <projekti>/todos-<nimi> --env <ympäristö> --since 10m
   ```

   Paikallinen ajo toimii vain, jos koneeltasi on pääsy kaikkiin tarvittaviin järjestelmiin. Se sopii hyvin kehityksen aikaiseen testaukseen. Odotettu tulos lokissa on kolme PATCH-päivitystä (252–254) ja kaksi POST-luontia (255–256). Deployn ja lokien tarkistuksen voi tehdä myös Weikion UI:ssa.

5. **Aiheuta virhe tarkoituksella:** Vaihda lähteen `httpUri` osoitteeseen `https://dummyjson.com/http/404` ja aja vaiheen 4 `apply` ja `deploy` uudelleen. Lokissa pitäisi näkyä lähteen GET-askeleelta `HttpOperationFailedException … statusCode: 404`. Pohdi: miksi tämä 404 pysäyttää ajon, kun `getTodo`-reitin 404 ei? Palauta lopuksi alkuperäinen osoite tekemällä deploy uudelleen edelliselle, toimivalle versiolle. Virheellinen versio jää Weikioon, mutta se ei ole enää ajossa.

6. **Pysäytä integraatio:** Kun toimiva versio on ajanut kerran, pysäytä integraatiosi komennolla `weikio stop <projekti>/todos-<nimi> --env <ympäristö>`. Muuten se kutsuu dummyjsonia edelleen joka minuutti. Pysäytys ei poista mitään: versiot jäävät Weikioon. Jos aikaa jää, kokeile versioiden deployta, pysäytystä ja käynnistystä myös UI:ssa.

## Valmista, kun

- Integraatio on julkaistu ja ajettu.
- Lokissa näkyy kolme PATCH-päivitystä (252–254) ja kaksi POST-luontia (255–256).
- Olet aiheuttanut virheen hakemalla 404-osoitteesta ja ottanut toimivan version uudelleen ajoon.
- Integraatio on pysäytetty.

## Hyvä tietää

- dummyjson vain simuloi muutokset eikä tallenna mitään: POST palauttaa aina id:n 255, ja jokainen ajo antaa saman tuloksen.
- Ajastin käynnistyy kerran minuutissa, joten samat rivit ilmestyvät lokiin uudelleen joka minuutti, kunnes pysäytät integraation.
- `{{training-api-url}}` on property placeholder, jonka arvo (`dummyjson.com`) on `application.properties`-tiedostossa. `{{training-api-token}}` viittaa projektiin lisättyyn salaisuuteen.
