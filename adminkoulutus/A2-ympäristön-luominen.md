# Harjoitustehtävä A2 - Ympäristön luominen

**TODO: Luo harjoitteluille oma agentti ja environment**

**Tilanne:** Uudelle integraatioprojektille on tehtävä taustatyö, jotta kehittäjät pääsevät julkaisemaan integraatioita alustalla.

**Tehtävä:** Varmista, että projekti on valmis integraatioille.

1. Tee itsellesi uusi rooli Weikion Settings -sivulla, jossa käytät admin-roolia pohjana. Tee roolin nimeksi `<nimi>-admin`. Eli käytännössä katso, mitä oikeuksia "admin" -roolilla on, ja anna omalle roolillesi samat oikeudet.
2. Luo projekti Settingsissä. Lisää projektiin itsesi, ja anna itsellesi projektissa juuri luomasi rooli.
3. Käy  luomassa API-avain, joka on vain tähän projektiin liitetty. Laita avaimen rooliksi sama rooli, minkä pistit itsellesi projektiin.
4. Testataan CLI:n toimintaa. Varmistetaan ensin, että luomaasi projektiin on pääsy. Sitten asetetaan kyseinen projekti kehitysympäristöksi. 
    1. **Kirjaudutaan palvelimelle:** pistä `weikio login https://test.weikiolab.com` , jonka jälkeen kirjaudu sisään tunnuksillasi, tai jos CLI ei löydä selainta, syötä saamasi API-avain
    2. **Tarkastetaan identiteetti:** pistä `weikio whoami` , joka näyttää tietoja kuten mihin projekteihin pääset käsiksi.
    3. **Asetetaan kehitysympäristö:** pistä `weikio context set --project projektin-nimi` , joka asettaa luomasi projektin kehitysympäristöksi
    4. **Näytä kontekstit:** pistä `weikio context list` , joka näyttää kaikki tallennetut kontekstit
5. Luo testi-integraatiota varten Events-näkymästä uusi `Event channel`, joka ottaa tärkeimmät tapahtumat integraatio(i)lta talteen. Varmista, että tämä liitetään juuri luomaamme projektiin. Vakioasetuksiin tai koodiin ei tarvitse tehdä muutoksia. Muista myös käynnistää Event channel demoagentille luomisen jälkeen.  
  
Tämän voi tehdä CLI:ssä tai käyttöliittymässä. CLI:n komento on tällainen:  
`weikio init event-channelin-nimi --kind event-channel --project projektin-nimi --env koulutus`  
  
`init` -komennon ajamisen jälkeen toimi weikion vastauksien mukaisesti.
6. Luo käyttöliittymässä Integrations-näkymässä uusi Camel-integraatio projektiin. Alustusnäkymässä käy muokkaamassa application.properties -tiedostosta lokitettava viesti haluamasi näköiseksi. Tee esimerkki-integraatioon seuraava muutos:  
Lisää ensimmäisen `to` -käskyn jälkeen **samalle "korkeudelle"** toinen `to` -käsky, johon pistät luodun Event channelin nimen esimerkin mukaisesti.

```yaml
        - to:
            uri: log
            parameters:
              loggerName: integraation-nimi
        - to: "kamelet:weikio-event-sink?channel=event-channelin-nimi"
```
Esimerkkinä `johannes-camel-demo` lopullisesta tiedostosta:

```yaml
- route:
    id: johannes-camel-demo
    from:
      uri: timer
      parameters:
        timerName: tick
        period: 60000
      steps:
        - setBody:
            expression:
              constant:
                expression: "{{message}}"
        - to:
            uri: log
            parameters:
              loggerName: johannes-camel-demo
        - to: "kamelet:weikio-event-sink?channel=test-demo-johannes"

```

Näin integraatio on liitetty luotuun Event channeliin

Tallenna, deployaa ja käynnistä integraatio koulutusagentille.

1. Varmista, että sekä integraation lokeissa että Event channelilla lokittuu asettamasi viesti.
2. Pysäytä integraatio: integraation sivulla paina environmentin kohdalla kolmesta pisteestä, ja valitse "Stop".
