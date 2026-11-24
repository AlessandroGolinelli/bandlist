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
