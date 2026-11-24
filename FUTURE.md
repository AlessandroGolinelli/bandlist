# Idee per il futuro

Appunti su cosa servirebbe per allargare Bandlist. **Niente di questo è ancora fatto.**

---

## 1. Login con Google

### Il problema
Per Firebase ogni persona è un utente con un codice interno (uid). Voti, accordature, "aggiunto da"…
sono tutti legati a quel codice. Se qualcuno entra con Google "da zero", Firebase crea un utente nuovo
e la persona perde accordature e voti.

### La soluzione: collegare Google all'account esistente
- **Firebase (a mano):** Authentication › Sign-in method › attivare **Google**. È gratis sul piano Spark.
  Il dominio `alessandrogolinelli.github.io` è già autorizzato.
- **Impostazioni dell'app:** pulsante **"Collega il mio Google"** (`linkWithPopup`). Ognuno lo preme una volta,
  dopo essere entrato col PIN: l'uid resta lo stesso e non si perde nulla.
- **Schermata d'accesso:** pulsante **"Entra con Google"** (`signInWithPopup`).
- **Usare il popup, non il redirect:** su iPhone/Safari il redirect spesso non funziona per i siti su
  github.io, perché il dominio di Firebase è diverso.
- **Sicurezza:** con Google chiunque abbia un account può autenticarsi. In `regole-firestore.txt` bisogna
  quindi permettere letture e scritture solo agli uid dei membri della band. A uno sconosciuto si mostra
  "Non sei nella band". Le regole vanno poi ricopiate in Firebase.
- **Il PIN resta come riserva** (telefono nuovo, problemi con Google). Si potrà togliere più avanti.

---

## 2. Allargare a più persone

### a) Un membro in più nella stessa band — facile
- Aggiungere il nome in `SCALETTA_CONFIG.members` in cima a `index.html`.
- Creare il suo utente in Firebase, oppure, con il login Google, aggiungere il suo uid alle regole.
- Non serve cambiare il resto dell'app.

### b) Altre band che usano l'app — lavoro grosso
Oggi tutti i dati stanno in un unico spazio condiviso: un'altra band vedrebbe le vostre canzoni.
Servirebbe:
- **Uno spazio per ogni band:** `bands/{bandId}/songs`, `.../wishlist`, `.../sessions`, `.../covers`,
  `.../meta`, `.../votes`. I dati attuali vanno spostati una volta in `bands/<la-nostra-band>/…`.
  Le accordature restano personali (`tunings/{uid}`), ma con la chiave anche per band.
- **Registrazione libera**, comodissima col login Google: niente utenti creati a mano.
- **"Crea band" / "Unisciti"**: chi crea la band riceve un **codice o un link d'invito** da girare agli altri.
- **Membri** in `bands/{bandId}/members/{uid}`, al posto della lista fissa nel file.
- **Regole di sicurezza:** "leggi e scrivi solo nelle band di cui sei membro".
- **Una persona in più band:** serve un modo per passare da una band all'altra.

Cose pratiche:
- **Costi:** il piano gratuito Spark (circa 50.000 letture e 20.000 scritture al giorno) regge qualche decina
  di band. Il piano Blaze serve solo se diventa una cosa grande.
- **Privacy:** se la usano degli sconosciuti, serve una pagina che spieghi quali dati si salvano
  (nome, email) e come farli cancellare.

### c) Alternativa senza cambiare codice
Ogni band fa una copia dell'app con il proprio progetto Firebase. Non c'è lavoro di sviluppo,
ma ognuno deve rifare tutta la configurazione: va bene solo per amici un po' smanettoni.

---

## Consiglio
Se si vuole davvero aprire ad altre band, conviene fare **login Google + uno spazio per band insieme**,
in un unico passaggio, invece di rifare l'accesso due volte.

---

## 3. Mettere l'app sugli store e guadagnarci (senza pubblicità)

### Prerequisito
Prima serve la versione per più band (punto 2b): uno spazio per band, registrazione libera e codice invito.

### Pubblicare sugli store
- **Come:** l'app resta una pagina web, dentro un "guscio" che la rende un'app vera. Si usa **Capacitor**,
  che è gratuito, e non serve riscriverla.
- **Costi fissi:** Google Play 25 $ una tantum, Apple 99 $ all'anno (serve un Mac con Xcode).
- **Richieste degli store:**
  - Apple scarta le app che sembrano "un sito impacchettato": servono icona, schermata d'avvio
    e un minimo di funzionamento anche senza rete.
  - Se c'è "Entra con Google", Apple chiede anche **"Entra con Apple"**.
  - La **cancellazione dell'account** si deve poter fare dall'app.
  - Servono una **pagina sulla privacy** e le schede sui dati raccolti da compilare in entrambi gli store.

### Come guadagnarci
| Modello | Come funziona | Pro / contro |
|---|---|---|
| App a pagamento (es. 3,99 € una tantum) | Chi la scarica paga | Semplice, ma pochi pagano senza provarla e ognuno dei membri dovrebbe pagare |
| **Abbonamento per band** (es. 2–3 €/mese o 20–25 €/anno) | Paga uno solo, tutta la band la usa | Entrate ogni mese; diviso in 5 costa pochissimo |
| Gratis + Pro | Gratis con limiti (es. 1 band, 30 canzoni); la Pro sblocca concerti, PDF, canzoni illimitate | Fa crescere gli utenti; chi prova si affeziona |

**Scelta consigliata:** gratis + **abbonamento Pro per band**. Paga il leader e gli altri entrano col codice invito.

### Da sapere sui soldi
- Apple e Google trattengono il **15%** di ogni pagamento (programma per piccoli sviluppatori, sotto 1 milione l'anno).
- Dentro le app degli store gli abbonamenti si vendono **con il sistema di pagamento dello store**,
  non con link a PayPal o Stripe. **RevenueCat** (gratuito all'inizio) gestisce gli abbonamenti di
  entrambi gli store in un posto solo.
- Con tanti utenti Firebase passa al piano **Blaze**, a consumo: i costi si coprono con gli abbonamenti.
  Le Cloud Functions per verificare i pagamenti richiedono comunque Blaze.
- **Fisco (Italia):** all'inizio può bastare la prestazione occasionale; oltre una certa cifra serve la
  **partita IVA** (spesso regime forfettario). Sentire un commercialista prima di partire.

### Percorso a tappe
1. App per più band + login Google (e Apple), sul sito come oggi.
2. Prova gratuita con 3–5 band di amici, per capire se piace e cosa manca.
3. App sugli store: prima Android (più semplice ed economico), poi iPhone.
4. Abbonamento Pro.
