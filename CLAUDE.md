# Bandlist — contesto per Claude Code

App web per **prepararsi alle prove**, più la scaletta (borderò) per i concerti. Nata per la band di Alessandro,
ora la può usare qualunque band: ognuna ha il suo spazio e si entra con un link d'invito.
Tutto è in un solo file, `index.html`: niente build, niente dipendenze da installare.
Interfaccia e testi in **italiano**. Il proprietario non è un programmatore: spiegagli le cose in modo semplice.

## Cosa fa (requisiti decisi con l'utente)

- **Prova** — un "pedale" con manopola (1–20 canzoni, valore condiviso in `meta/settings.count`) e un
  pulsantone *Estrai le canzoni*. L'estrazione è casuale ma pesata: più probabilità alle canzoni
  ripassate poche volte, segnate "Poco" e non toccate da tempo (`weightOf`). La lista viene poi ordinata
  con le meno ripassate in cima. La cover del mese è sempre aggiunta in fondo.
  Si può anche **fare la lista a mano** ("✎ Faccio io la lista", menu "+ Aggiungi una canzone", × per togliere).
  Su ogni canzone: "Segna ripassata" e livello **Poco / Medio / Bene**, ↻ per cambiarla a caso.
  Sotto la lista il pulsantone verde "✓ Ripassate tutte" (con Annulla; sparisce quando sono tutte fatte).
- **Repertorio** — canzoni che la band sa. Conteggio ripassi = sessioni con `done[id] === true` + `adjust`
  (correzione manuale con −/+). Modifica con ✎. Flag `own` = pezzo della band ("Nostra").
- **Proposte** — pezzi da imparare. Voto **+1 / −1, uno a testa** (ripremere toglie il voto).
  Azioni: ★ cover del mese, "La sappiamo già" (→ repertorio), Togli.
- **Cover del mese** — un documento per mese in `covers/AAAA-MM`. Si può sempre rimettere tra le proposte,
  segnare "Imparata" (→ repertorio) e annullare. Se a inizio mese la precedente non è finita, si può tenere.
- **Concerto** — scheda "Scaletta concerto": una sola scaletta condivisa in `meta/concert.songIds` (in ordine).
  "+ Aggiungi canzone" apre una ricerca nel repertorio; × toglie (con Annulla), "Svuota la scaletta" (con Annulla).
  Riordino tenendo premuto ☰ a destra e trascinando (pointer events, scorre da sola vicino ai bordi; `drag`,
  `dragUpdate`); con la tastiera frecce ↑ ↓ su ☰. Durante il trascinamento `renderConcert` non ridisegna.
  "⇩ Converti in PDF" (`#gig-pdf`): jsPDF 2.5.1 da cdnjs caricato al primo clic; il PDF ha solo numero, titolo
  e accordatura di chi lo scarica (niente altro, a parte nome band e data in alto).
- **Accordature personali** — si cambiano **solo in Repertorio**, con due caselle unite senza freccia:
  nota (E, D#, D, C#, C, B) e tipo (Standard, Drop + nomi propri aggiunti in Impostazioni › "Le mie accordature").
  Predefinito mostrato: D# Standard (non salvato finché non si cambia). Salvate in `tunings/{uid}`, le vede solo
  chi le ha messe. Altrove solo in lettura (`tuningText`): in Prova in viola sotto l'artista, in Concerto
  come etichetta viola accanto al titolo.
- **Ascolta** — il quadratino verde ♪ apre la canzone su Spotify (app sul telefono): il suo link Spotify se c'è,
  altrimenti una ricerca "titolo artista" (`listenUrl`). Il player in basso non viene più usato dai pulsanti.
- **Annulla** — ogni azione importante mostra un toast con "Annulla" (funzione inversa passata a `write()`).
  Mantieni questa regola per ogni nuova azione distruttiva.
- **Spotify** — incollando un link di condivisione: oEmbed (`open.spotify.com/oembed`) dà **titolo e copertina**
  (non l'artista). Per l'artista l'app cerca il titolo su Apple Music (`itunes.apple.com/search`, CORS aperto) e mostra un menu a tendina (`#s-artist-pick`, `#w-artist-pick`, `artistCandidates`); il campo resta scrivibile a mano. La pagina embed di Spotify ha l'artista ma non ha CORS; il player è l'iframe `open.spotify.com/embed/...` in una barra fissa
  in basso (`#player`) che non si ricarica quando i dati cambiano. I campi link sono `type="text"` perché
  dal telefono il link arriva spesso con del testo intorno.
- **Link YouTube / Apple Music** — oltre a Spotify si può incollare un link YouTube (oEmbed `youtube.com/oembed`:
  titolo "Artista - Brano (Official…)" diviso da `splitYouTube`) o Apple Music (`itunes.apple.com/lookup`: titolo,
  artista esatto, copertina). Si salvano negli stessi campi: `spotifyType` = `"youtube"` o `"apple"` (altrimenti è
  Spotify), `spotifyId` = id del video/brano, `spotifyUrl` = link. Vedi `parseLink`, `resolveLink`, `linkKind`, `embedUrl`.
  La tendina degli artisti (Apple Music) compare quando c'è un titolo, anche scritto a mano.
- **Accesso** — "Entra con Google" (popup, non redirect: su iPhone il redirect non va con github.io) oppure
  email e password (con "Registrati" e "Password dimenticata?"). La band storica entra ancora col **PIN**
  ("Entri col PIN? Tocca qui": i 5 nomi di `SCALETTA_CONFIG.members`, email `nome@bandlist.app` da `emailFor`,
  eccezioni in `SCALETTA_CONFIG.logins`). In Impostazioni "Collega il mio Google" (`linkWithPopup`) tiene lo stesso uid.
- **Band** — dopo l'accesso `afterLogin` legge `users/{uid}`: apre la band corrente, oppure mostra "Unisciti a …"
  (se c'è un invito) o "Crea la tua band". Tutti i dati di una band stanno in `bands/{id}/…`: l'app usa sempre
  percorsi corti (`songs/x`) e `bandStore` aggiunge il prefisso (tranne `tunings/`). Impostazioni: "Copia il link
  d'invito" (`?invito=CODICE`, salvato in localStorage `bandlist-invite` finché non si entra), "Crea un link nuovo"
  (il vecchio smette di valere), elenco membri, passaggio tra le proprie band, "+ Crea un'altra band".
- **Membri e account** — in Impostazioni l'elenco dei membri ha "Togli" (con Annulla: toglie anche i voti; i voti di chi
  non è più nella band non contano in `voteTally`). "Esci da questa band" (`leaveBand`, con conferma: l'ultimo che esce
  cancella la band e tutti i suoi dati). "Cancella il mio account" (`deleteAccountData` + `user.delete()`: Firebase vuole
  un accesso recente, quindi con Google riapre il popup, con password chiede di uscire e rientrare). Le canzoni aggiunte
  restano alla band. Chi ha creato una band può aggiungersi da solo solo nei primi 10 minuti (regole).
- **Privacy** — `privacy.html` (pagina a parte, linkata da accesso e Impostazioni). Se si salvano dati nuovi, aggiornala.
- **Trasloco della band storica** — i suoi dati erano in cima al database. Al primo accesso col PIN, `legacyJoin` /
  `migrateLegacy` li **copiano** (senza cancellarli) in `bands/fondatori` (`SCALETTA_CONFIG.legacyBand`), con
  membri e voti di tutti. La copia vecchia resta in sola lettura.

## Struttura del file

- In cima, `window.SCALETTA_CONFIG`: configurazione Firebase, `emailDomain`, `members`, `seed`
  (canzoni di partenza caricate una sola volta al primo accesso se il database è vuoto, con id fissi `seed-...`).
- **Modalità prova**: se `firebase.apiKey` inizia con `INCOLLA`, l'app usa `localStorage`
  (chiave `scaletta-demo-v1`) e si entra toccando un nome (come la band storica: fa anche il trasloco).
  Con la configurazione vera usa Firestore.
- `firebaseStore()` e `localStore()` espongono la stessa piccola API
  (`doc(path).set/update/delete/get/onSnapshot`, `collection(path).doc()/get/onSnapshot`).
  `update` su Firestore appiattisce gli oggetti annidati in percorsi con il punto (`flatten`), così
  `update({done: {[id]: true}})` non sovrascrive tutta la mappa.
- Stato in `S`, rendering con template string e `esc()` per ogni dato inserito dagli utenti.
- I nomi delle persone si leggono da `members/{uid}` al momento del rendering: nei dati si salvano solo gli uid.

## Dati (Firestore)

Tutto quello che riguarda una band sta sotto `bands/{bandId}/` (nella tabella i percorsi sono relativi a lì,
tranne le righe segnate *globale*).

| Collezione | Contenuto |
|---|---|
| `songs/{id}` | `title, artist, key, own, level (1/2/3/null), adjust, lastAdjust, spotifyId, spotifyType (track/album/youtube/apple), spotifyUrl, art, addedBy, addedAt` |
| `wishlist/{id}` | `title, artist, spotify*, art, addedBy, addedAt` |
| `votes/{uid}` | `picks: {wishId: 1 \| -1}` — modificabile solo dal proprietario |
| `sessions/{AAAA-MM-GG}` | `songIds[], done{songId: bool}, coverDone, manual, createdBy, createdAt` |
| `covers/{AAAA-MM}` | dati del brano + `wishId, wish, setBy, learned, learnedSongId, carriedFrom` |
| `meta/settings` | `count, bandName, seeded` |
| `meta/concert` | `songIds[]` (ordine della scaletta), `updatedBy, updatedAt` |
| `meta/invite` | `code` — il codice d'invito attuale |
| `members/{uid}` | `name, lastSeen, code` (il codice usato per entrare) — chi c'è dentro è della band |
| `bands/{id}` *globale* | `name, createdBy, createdAt, migrated` |
| `invites/{code}` *globale* | `band, name, createdBy, createdAt` — leggibile da chiunque abbia il link |
| `users/{uid}` *globale* | `name, band (quella aperta), bands: {id: nome}` — solo il proprietario |
| `tunings/{uid}` *globale* | `songs: {songId: {note: E\|D#\|D\|C#\|C\|B, type: Standard\|Drop\|nome proprio}}, custom: [nomi propri]` — solo il proprietario |

Le regole di sicurezza sono in `regole-firestore.txt`: se aggiungi una collezione, aggiungila anche lì.

## Aspetto

Tema chiaro unico (sfondo bianco, voluto dall'utente), più colorato: blu = Prova, verde = Repertorio,
rosa = Proposte, viola = Concerto, giallo = cover del mese. Font: Archivo (titoli), Inter (testo), Permanent Marker (foglio).
Pezzi caratteristici: il pedale da chitarra con footswitch e LED, il foglio della scaletta con lo scotch.
Deve funzionare bene sul telefono (390px) senza scorrimento orizzontale.

## Come provarlo

Apri `index.html` nel browser: parte in modalità prova. Per i test automatici con Playwright, apri il file
con `file://`, blocca la rete esterna e simula `open.spotify.com/oembed` se serve.
Per ripartire da zero cancella `localStorage` (`scaletta-demo-v1`, `scaletta-demo-user`).

## Pubblicazione

GitHub Pages da `main`, cartella root → https://alessandrogolinelli.github.io/bandlist/
(va attivato una volta in Settings › Pages).

## Da fare

1. ~~Nomi veri dei 5 membri~~ fatto: Goli, Cale, G Spot, Rollo, Aema.
2. **Firebase**: creare il progetto, Firestore (Europa) con le regole di `regole-firestore.txt`,
   Authentication email/password con i 5 utenti, poi incollare
   `firebaseConfig` in cima a `index.html`. Aggiungere `alessandrogolinelli.github.io` ai domini autorizzati.
   Restare sul piano gratuito Spark (non Blaze).
3. **Link Spotify** delle canzoni (soprattutto Intercity ed Easier, pezzi della band): si possono mettere nel
   `seed` (quarto elemento) prima del primo accesso, oppure dall'app con ✎. Le copertine mancanti si
   recuperano da sole (`fillMissingArt`).
4. **Aprire alle altre band** (in quest'ordine): pubblicare l'app nuova e le regole nuove insieme; uno della band
   storica entra col PIN (fa il trasloco in `bands/fondatori`); solo dopo attivare Google in Authentication ›
   Sign-in method e riaprire le registrazioni (Authentication › Impostazioni › Azioni utente).
