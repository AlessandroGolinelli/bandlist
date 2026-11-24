# Bandlist — contesto per Claude Code

App web della band di Alessandro per **prepararsi alle prove** (non per i concerti).
Tutto è in un solo file, `index.html`: niente build, niente dipendenze da installare.
Interfaccia e testi in **italiano**. Il proprietario non è un programmatore: spiegagli le cose in modo semplice.

## Cosa fa (requisiti decisi con l'utente)

- **Prova** — un "pedale" con manopola (1–20 canzoni, valore condiviso in `meta/settings.count`) e un
  pulsantone *Estrai le canzoni*. L'estrazione è casuale ma pesata: più probabilità alle canzoni
  ripassate poche volte, segnate "Poco" e non toccate da tempo (`weightOf`). La lista viene poi ordinata
  con le meno ripassate in cima. La cover del mese è sempre aggiunta in fondo.
  Si può anche **fare la lista a mano** ("✎ Faccio io la lista", menu "+ Aggiungi una canzone", × per togliere).
  Su ogni canzone: "Segna ripassata" e livello **Poco / Medio / Bene**, ↻ per cambiarla a caso.
- **Repertorio** — canzoni che la band sa. Conteggio ripassi = sessioni con `done[id] === true` + `adjust`
  (correzione manuale con −/+). Modifica con ✎. Flag `own` = pezzo della band ("Nostra").
- **Proposte** — pezzi da imparare. Voto **+1 / −1, uno a testa** (ripremere toglie il voto).
  Azioni: ★ cover del mese, "La sappiamo già" (→ repertorio), Togli.
- **Cover del mese** — un documento per mese in `covers/AAAA-MM`. Si può sempre rimettere tra le proposte,
  segnare "Imparata ✓" (→ repertorio) e annullare. Se a inizio mese la precedente non è finita, si può tenere.
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
- **Accesso** — schermata "Chi sei?" con i 5 nomi di `SCALETTA_CONFIG.members` + PIN di 6 cifre.
  Sotto: Firebase Auth email/password con email `nome@bandlist.app` (vedi `emailFor`) e PIN come password.
  La registrazione libera va disattivata in Firebase, così entrano solo i 5 utenti creati a mano.

## Struttura del file

- In cima, `window.SCALETTA_CONFIG`: configurazione Firebase, `emailDomain`, `members`, `seed`
  (canzoni di partenza caricate una sola volta al primo accesso se il database è vuoto, con id fissi `seed-...`).
- **Modalità prova**: se `firebase.apiKey` inizia con `INCOLLA`, l'app usa `localStorage`
  (chiave `scaletta-demo-v1`) e si entra senza PIN. Con la configurazione vera usa Firestore.
- `firebaseStore()` e `localStore()` espongono la stessa piccola API
  (`doc(path).set/update/delete/onSnapshot`, `collection(path).doc()/onSnapshot`).
  `update` su Firestore appiattisce gli oggetti annidati in percorsi con il punto (`flatten`), così
  `update({done: {[id]: true}})` non sovrascrive tutta la mappa.
- Stato in `S`, rendering con template string e `esc()` per ogni dato inserito dagli utenti.
- I nomi delle persone si leggono da `members/{uid}` al momento del rendering: nei dati si salvano solo gli uid.

## Dati (Firestore)

| Collezione | Contenuto |
|---|---|
| `songs/{id}` | `title, artist, key, own, level (1/2/3/null), adjust, lastAdjust, spotifyId, spotifyType (track/album/youtube/apple), spotifyUrl, art, addedBy, addedAt` |
| `wishlist/{id}` | `title, artist, spotify*, art, addedBy, addedAt` |
| `votes/{uid}` | `picks: {wishId: 1 \| -1}` — scrivibile solo dal proprietario |
| `sessions/{AAAA-MM-GG}` | `songIds[], done{songId: bool}, coverDone, manual, createdBy, createdAt` |
| `covers/{AAAA-MM}` | dati del brano + `wishId, wish, setBy, learned, learnedSongId, carriedFrom` |
| `meta/settings` | `count, bandName, seeded` |
| `members/{uid}` | `name, lastSeen` — scrivibile solo dal proprietario |

Le regole di sicurezza sono in `regole-firestore.txt`: se aggiungi una collezione, aggiungila anche lì.

## Aspetto

Tema chiaro unico (sfondo bianco, voluto dall'utente), più colorato: blu = Prova, verde = Repertorio,
rosa = Proposte, giallo = cover del mese. Font: Archivo (titoli), Inter (testo), Permanent Marker (foglio).
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
   Authentication email/password con i 5 utenti e la registrazione disattivata, poi incollare
   `firebaseConfig` in cima a `index.html`. Aggiungere `alessandrogolinelli.github.io` ai domini autorizzati.
   Restare sul piano gratuito Spark (non Blaze).
3. **Link Spotify** delle canzoni (soprattutto Intercity ed Easier, pezzi della band): si possono mettere nel
   `seed` (quarto elemento) prima del primo accesso, oppure dall'app con ✎. Le copertine mancanti si
   recuperano da sole (`fillMissingArt`).
