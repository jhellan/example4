# Python-eksempel på gruppefunksjonalitet i Feide

- Trykk på denne knappen: [![Run on Repl.it](https://repl.it/badge/github/jhellan/example4)](https://repl.it/github/jhellan/example4)

- Logg inn på repl.it med konto fra google, github eller facebook

- I repl.it: Gå inn i filen `main.py`. Finn linjen med `APP_BASE_URI` og bytt ut `USERNAME` med ditt brukernavn i repl.it. Brukernavnet står øverst til venstre, f.eks. `@pelle42`. Ikke ta med `@`-tegnet.

- Opprett en ny tjeneste i https://kunde.feide.no.. Bruk knappen [ + Opprett en ny Feide-tjeneste ]. Se bort fra advarselen om OpenID Connect. Gi et navn, og trykk på [ Start registrering ].

- Trykk på 'Attributer som skal brukes' i punktlisten. Trykk på knappen [ Rediger ] i siden du kommer til. Huk av for de
  attributtene du ønsker. Forslag:

  - Navn
  - Brukeridentifikatorer hos organisasjonen
  - Epost
  - Organisasjonstilhørighet
  - Gruppetilhørighet for undervisning

- Trykk på [ Lagre endringer ]

- Trykk på [ &lt;- Tilbake til tjeneste ] øverst på siden.

- Vi kan hoppe over de andre elementene, så vi trykker på [ Meld inn tjenesten ].

- Får beskjed om at tjenesten må godkjennes av Feide. Feide-personell kan trykke på [ Godkjenn ny tjeneste ].

- Akkurat nå må man fremdeles taste inn manuelt lenken for å komme til siden hvor man kan registrere OpenID
  Connect-konfigurasjoner. Lenken er

  `https://kunde.feide.no/org/<orgid>/services/<tjensteid>/configurations/open-id-connect/new`

  f.eks.

  `https://kunde.feide.no/org/2217310/services/2163019/configurations/open-id-connect/new`

- Sett 'Redirect URI etter innlogging' til verdien av `APP_BASE_URI` etterfulgt av `/redirect_uri`.

- Legg inn et navn på konfigurasjonen

- Huk av for at testbrukere og Feide gjestebrukere skal kunne logge inn

- Trykk [ Lagre og lukk ]

- Et nytt avsnitt kommer frem med Client ID og Client Secret. Finn linjen med `APP_CLIENT_ID` i programkoden og bytt ut
  verdien med din client ID.

- Trykk på [ Vis ] ved siden av Client Secret i kundeportalen. Finn filen `.env` i repl.it, og bytt ut verdien bak `APP_SECRET=` med verdien av Client Secret.


- Trykk på [ &lt;- Tilbake til konfigurasjoner ] øverst på siden.

- Trykk på `Run`-knappen i repl.it.

- Hvis applikasjonen ikke starter, prøv shellkommandoen `poetry install` fra shell-fanen.

- Etter en stund dukker det opp en minibrowser i repl.it. Vi kan ikke bruke denne, men vi kan gå til samme URL i en annen tab. Logg derfra inn med Feide

- Den lokale sesjonen løper ut etter ett minutt. Det går også an å logge ut ved å gå til samme URL etterfulgt av `/logout`.

Når vi har logget inn, kommer vi til en side med info som applikasjonen har fått fra Feide:

- access\_token: En streng av hex-sifre som bare Feide kan tolke.
- id\_token: Dette er jwt med info om brukeren. I Feide inneholder den svært lite.
  Siden viser token etter dekoding.
- userinfo: Kommer fra OpenID Connect userinfo-endepunktet.
- mygroups: Gruppene innlogget bruker er med i. Kommer fra Feides gruppe-API.
- Medlemmene i alle undervisningsgrupper innlogget bruker er med i. Også fra gruppe-API.

Applikasjonen overlater til biblioteket `pyoidc` å håndtere innlogging. Bak kulissene gjør den 'authorization code flow'. I neste eksempel viser vi hvordan det foregår. Det vi kan se her er:

- Vi forteller biblioteket hvor det kan finne Feide, og hva det trenger å vite om applikasjonen.

- Rammeverket `flask` binder kode til URL-er i applikasjonen på denne måten:

  ```
  @app.route('/')
  def login1():
      <kode>

  @app.route('/logout')
  def logout():
      <kode>

  ```

- Autentisering osv. legges på slik: 

  ```
  @app.route('/')
  @auth.oidc_auth(AUTH_PROVIDER_NAME)
  def login1():
      <kode>

  @app.route('/logout')
  @auth.oidc_logout
  def logout():
      <kode>

  ```

- Når vi kommer til vår egen kode på hovedsiden er vi allerede innlogget. Biblioteket har lagt `access_token`, `id_token` og `userinfo` inn i brukersesjonen. Vi bruker `access_token` til å hente ut gruppeinfo. Til slutt viser vi frem alt sammen.
