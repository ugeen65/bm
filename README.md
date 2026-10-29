# bm — gestore di bookmark da terminale

**bm** è un gestore di segnalibri (bookmark) per il terminale Linux, scritto interamente in **Bash**.
Ha un'interfaccia interattiva a schermo intero (TUI) navigabile con i tasti freccia e una serie di
comandi da riga di comando per l'uso negli script.

Il "database" è una semplice cartella di **file di testo `.txt`**: niente SQLite, niente formati
proprietari. Puoi leggere, modificare, copiare, sincronizzare e mettere sotto `git` i tuoi bookmark
con qualsiasi strumento.

```
 BOOKMARKS   ·   database: ~/.segnalibri/<categoria>.txt
 Tutte (58)   generale (52)   finanza (4)   linux (2)

 CATEGORIA                    TAGS                    URL
______________________________________________________________________________
 AMAZON                       #acquisti               amazon.it
 BONUS COLONNINE DOMESTICHE   #colonnine #utenze      bonuscolonnine.it
 BUDGETBAKERS                 #finanza-personale      budgetbakers.com
 CODEBERG                     #git                    codeberg.org
──────────────────────────────────────────────────────────────────────────────
 AMAZON                                                   file: generale.txt
 https://www.amazon.it/                                    2026-10-29  (1/58)
 F1 ? aiuto │ ↑↓←→ muovi  Invio apri  a nuovo  t tag  # filtra tag  e modifica …
```

---

## Indice

- [Caratteristiche](#caratteristiche)
- [Requisiti](#requisiti)
- [Installazione](#installazione)
- [Dove vengono salvati i dati](#dove-vengono-salvati-i-dati)
- [Formato dei file](#formato-dei-file)
- [L'interfaccia interattiva (TUI)](#linterfaccia-interattiva-tui)
- [Comandi da terminale](#comandi-da-terminale)
- [Tag](#tag)
- [Importare i bookmark](#importare-i-bookmark)
- [Esportare i bookmark](#esportare-i-bookmark)
- [Ordinamento alfabetico](#ordinamento-alfabetico)
- [Configurazione](#configurazione)
- [Risoluzione dei problemi](#risoluzione-dei-problemi)
- [Limiti noti](#limiti-noti)
- [Licenza](#licenza)

---

## Caratteristiche

- **Interfaccia a schermo intero** navigabile con le frecce, con colonne, riga dei titoli e aiuto in linea (`F1`).
- **Database in file di testo**: un file `.txt` per categoria, una riga per bookmark.
- **Categorie** (una per file) e **tag** multipli per ogni bookmark.
- **Ricerca** istantanea in nome, indirizzo e tag; **filtro per tag** scelto da un elenco.
- **Importazione** dai browser (file HTML esportato da Firefox, Chrome, Edge, Brave, buku…) e da
  **file di testo** con un indirizzo per riga, anche in formati misti.
- **Esportazione** in HTML, reimportabile in qualsiasi browser.
- **Ordinamento alfabetico** dei bookmark di ogni categoria.
- **Controllo dei link** non più raggiungibili.
- Apertura nel browser, copia dell'indirizzo negli appunti.
- Nessuna dipendenza oltre agli strumenti standard di Linux.

---

## Requisiti

**Obbligatori** (presenti in qualsiasi distribuzione Linux):

- `bash` 4.4 o successivo
- `awk` (va bene sia `mawk`, predefinito su Debian/Ubuntu, sia `gawk`), `sed`, `grep`, `sort`
  e gli altri strumenti di base (`coreutils`)

**Facoltativi**, per alcune funzioni:

| Programma | A cosa serve |
|---|---|
| `curl` | ricavare automaticamente il titolo della pagina quando aggiungi un bookmark; comando `check` |
| `xdg-open` (pacchetto `xdg-utils`) | aprire i bookmark nel browser predefinito |
| `wl-clipboard`, `xclip` oppure `xsel` | copiare l'indirizzo negli appunti (tasto `y`) |
| un editor (`nano`, `vim`…) | comando `bm edit`; si usa quello indicato in `$EDITOR` |

Su Debian/Ubuntu:

```bash
sudo apt install curl xdg-utils xclip
```

---

## Installazione

1. Scarica lo script `bm` dal repository.
2. Copialo in una cartella del `PATH` e rendilo eseguibile:

```bash
mkdir -p ~/.local/bin
cp bm ~/.local/bin/bm
chmod +x ~/.local/bin/bm
```

3. Se `~/.local/bin` non è nel `PATH`, aggiungilo al `~/.bashrc` e riapri il terminale:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
```

4. Avvia:

```bash
bm
```

> **Attenzione ai conflitti di nome.** Se sul sistema esiste già un alias, una funzione o un
> programma chiamato `bm`, quello avrà la precedenza. Controlla con `type -a bm`. In caso di conflitto
> basta installare lo script con un altro nome (ad esempio `~/.local/bin/segnalibri`): funziona allo
> stesso modo.

---

## Dove vengono salvati i dati

I bookmark sono salvati nella cartella:

```
~/.segnalibri/
```

cioè `/home/<utente>/.segnalibri/`. È una cartella nascosta (il nome inizia con un punto): nei file
manager si visualizza con `Ctrl+H`, nel terminale con `ls -a ~`.

Dentro c'è **un file `.txt` per ogni categoria**:

```
~/.segnalibri/
├── generale.txt        ← categoria predefinita
├── finanza.txt
└── linux.txt
```

- La cartella e i file vengono creati automaticamente al primo utilizzo.
- Il percorso è mostrato nella barra in alto della TUI, nella schermata d'aiuto e con `bm path`.
- Per usare un'altra cartella, ad esempio una sincronizzata con Nextcloud o Syncthing, imposta la
  variabile `BM_DIR` (vedi [Configurazione](#configurazione)).

---

## Formato dei file

Ogni riga di un file di categoria è un bookmark, con quattro campi separati da `|`:

```
data|indirizzo|nome|tag1,tag2
```

Esempio (`~/.segnalibri/linux.txt`):

```
2026-10-28|https://wiki.archlinux.org|Arch Wiki|linux,docs
2026-10-28|https://www.debian.org/|Debian|linux,distro
2025-03-01|https://codeberg.org|CODEBERG|git
```

| Campo | Contenuto |
|---|---|
| data | data di inserimento, formato `AAAA-MM-GG` |
| indirizzo | URL completo, con `https://` |
| nome | titolo del bookmark (può essere vuoto: in quel caso viene mostrato l'indirizzo) |
| tag | elenco di tag separati da virgola (può essere vuoto) |

Regole:

- le righe vuote e quelle che iniziano con `#` sono ignorate (puoi usarle per i commenti);
- il carattere `|` non può comparire nei campi: se lo inserisci viene sostituito da `/`;
- il nome del file (senza `.txt`) è il nome della categoria e può contenere solo lettere, numeri,
  `.`, `_` e `-`.

I file si possono modificare a mano con qualsiasi editor; bm li rilegge a ogni avvio (o premendo `r`).

---

## L'interfaccia interattiva (TUI)

Si apre lanciando `bm` senza argomenti (oppure `bm tui`).

### Struttura dello schermo

| Zona | Contenuto |
|---|---|
| riga 1 | nome del programma e percorso del database |
| riga 2 | categorie (`Tutte` e una voce per ogni file), con il numero di bookmark; quella attiva è evidenziata |
| riga 3 | filtri attivi (ricerca e/o tag), altrimenti vuota |
| riga 4 | titoli delle colonne: **CATEGORIA** (nome del bookmark), **TAGS**, **URL** |
| riga 5 | separatore `____` |
| elenco | i bookmark; la riga selezionata è evidenziata |
| in basso | dettagli del bookmark selezionato: nome, file in cui è salvato, indirizzo completo, data, posizione (es. `3/58`) |
| ultima riga | tasti principali oppure messaggi (es. "Bookmark modificato.") |

Le colonne si adattano alla larghezza del terminale; i testi troppo lunghi vengono accorciati con `…`.

### Tasti

**Movimento**

| Tasto | Azione |
|---|---|
| `↑` `↓` oppure `k` `j` | muove la selezione |
| `PagSu` `PagGiù` | pagina precedente / successiva |
| `Home` `Fine` oppure `g` `G` | primo / ultimo bookmark |
| `←` `→` oppure `Tab` | cambia categoria (`Tutte` mostra tutto) |

**Azioni sul bookmark selezionato**

| Tasto | Azione |
|---|---|
| `Invio` oppure `o` | apre nel browser |
| `y` | copia l'indirizzo negli appunti |
| `t` | aggiunge / toglie tag scegliendoli da un elenco |
| `e` | modifica indirizzo, nome e tag |
| `m` | sposta in un'altra categoria |
| `d` oppure `Canc` | elimina (chiede conferma) |

**Altro**

| Tasto | Azione |
|---|---|
| `a` | aggiunge un nuovo bookmark |
| `/` | cerca in nome, indirizzo e tag |
| `#` | mostra solo i bookmark con un certo tag |
| `Esc` | toglie ricerca e filtro per tag |
| `s` | ordina alfabeticamente la categoria mostrata (in `Tutte`: ogni categoria) |
| `i` | importa da un file `.html` del browser o da un `.txt` |
| `x` | esporta tutti i bookmark in HTML |
| `r` | ricarica i file da disco |
| `F1` oppure `?` oppure `h` | aiuto in linea |
| `q` | esce |

Nei campi di testo (in basso): `Invio` conferma, `Ctrl+D` annulla, `Tab` completa i percorsi dei file.
I campi di modifica sono già compilati con il valore attuale.

### Aiuto in linea

`F1` (oppure `?`) apre una schermata con tutti i tasti, tutti i comandi da terminale e il percorso
dei dati. Si scorre con `↑` `↓` `PagSu` `PagGiù` e si chiude con `q`, `Esc` o di nuovo `F1`.

> Alcuni emulatori di terminale usano `F1` per il proprio aiuto e non lo passano ai programmi:
> in quel caso usa `?`.

### Aggiungere un bookmark

Premi `a` e rispondi alle domande:

1. **URL** — se manca `https://` viene aggiunto. Lasciandolo vuoto si annulla.
2. **Nome** — se lo lasci vuoto, bm prova a ricavarlo dal titolo della pagina (serve `curl`).
3. **Categoria** — proposta quella che stai guardando; una categoria nuova crea un nuovo file.
4. **Tag** — separati da virgola.

Gli indirizzi già presenti non vengono aggiunti una seconda volta.

---

## Comandi da terminale

Tutte le funzioni sono disponibili anche come comandi, utili negli script.
L'**INDICE** è il numero mostrato da `bm list` o `bm search`.

| Comando | Descrizione |
|---|---|
| `bm` | apre la TUI (se l'output non è un terminale, equivale a `bm list`) |
| `bm add URL [-t nome] [-c categoria] [-g tag1,tag2]` | aggiunge un bookmark |
| `bm list [categoria]` | elenca i bookmark, numerati |
| `bm search TERMINE` | cerca in indirizzo, nome e tag (senza distinguere maiuscole/minuscole) |
| `bm tag NOME` | mostra i bookmark con quel tag |
| `bm tag-add INDICE tag1,tag2` | aggiunge uno o più tag |
| `bm tag-rm INDICE tag1,tag2` | toglie uno o più tag |
| `bm tags` | elenca tutti i tag con il numero di bookmark |
| `bm cats` | elenca le categorie con il numero di bookmark |
| `bm open INDICE` | apre nel browser |
| `bm edit INDICE` | apre il file della categoria in `$EDITOR`, sulla riga del bookmark |
| `bm mv INDICE CATEGORIA` | sposta in un'altra categoria |
| `bm del INDICE` | elimina (chiede conferma) |
| `bm sort [categoria]` | ordina alfabeticamente ogni categoria (o solo quella indicata) |
| `bm check` | segnala i link che non rispondono (serve `curl`) |
| `bm import FILE [-c categoria]` | importa da HTML o da testo |
| `bm export > file.html` | esporta in HTML |
| `bm path` | mostra la cartella dei dati |
| `bm help` | mostra l'aiuto |

Esempi:

```bash
bm add github.com -t "GitHub" -c sviluppo -g git,codice
bm search wiki
bm tag-add 12 linux,docs
bm mv 12 linux
bm export > ~/bookmarks.html
```

Gli indici cambiano dopo un'eliminazione, uno spostamento o un ordinamento: rilancia `bm list`
prima di usarli.

---

## Tag

- Ogni bookmark può avere quanti tag vuoi.
- I tag vengono normalizzati automaticamente, per evitare doppioni:
  - maiuscole → minuscole;
  - spazi → trattini (`Cose di Linux` → `cose-di-linux`);
  - un eventuale `#` iniziale viene tolto;
  - un tag già presente sul bookmark non viene ripetuto.

**Nella TUI**

- `t` apre l'elenco di tutti i tag esistenti, con il numero di bookmark per ciascuno.
  `Spazio`/`Invio` mette o toglie la spunta `[x]`, `n` crea tag nuovi, `q`/`Esc` salva ed esce.
- `#` mostra solo i bookmark con il tag scelto; si combina con la categoria e con la ricerca `/`.
  `Esc` toglie il filtro.

**Da terminale**: `bm tag-add`, `bm tag-rm`, `bm tag`, `bm tags`.

---

## Importare i bookmark

```bash
bm import FILE
bm import FILE -c categoria     # mette tutto in un'unica categoria
```

oppure, nella TUI, il tasto `i`.

bm riconosce il tipo di file **dal contenuto**, non dall'estensione. Gli indirizzi già presenti
vengono saltati, quindi puoi ripetere l'importazione senza creare doppioni. Al termine viene
indicato quanti bookmark sono stati importati e quanti saltati.

### Da un browser (HTML)

Esporta i segnalibri dal browser:

- **Firefox**: `Ctrl+Shift+O` → *Importa e salva* → *Esporta segnalibri in HTML…*
- **Chrome / Edge / Brave**: *Gestione preferiti* → menu ⋮ → *Esporta preferiti*

Poi:

```bash
bm import ~/bookmarks.html
```

- Ogni cartella del browser diventa una categoria (con sottocartelle conta la più interna); il nome
  viene convertito in minuscolo, con `_` al posto degli spazi e senza accenti (`Città e Meteo` →
  `citta_e_meteo`). I bookmark fuori da qualsiasi cartella vanno in `generale`.
- Vengono importati anche la data di aggiunta e i tag (se presenti, come negli export di Firefox o buku).
- I link interni del browser (es. `place:` di Firefox) vengono ignorati.

### Da un file di testo

Una riga per bookmark. Sono riconosciute tutte queste forme, anche mescolate:

```
https://www.debian.org                          ← solo l'indirizzo
Arch Wiki https://wiki.archlinux.org            ← nome + indirizzo
https://github.com - GitHub                     ← indirizzo + nome
www.repubblica.it                               ← senza https://
codeberg.org                                    ← solo il dominio
[MDN](https://developer.mozilla.org)            ← link in stile Markdown
2025-03-01|https://www.itas.it|Fondo Itas|tag   ← il formato di bm

[Città e Meteo]                                 ← le righe seguenti vanno
Meteotrentino https://www.meteotrentino.it      ←   nella categoria citta_e_meteo
```

- Una riga che contiene solo `[Nome]` apre una categoria per le righe seguenti.
- Le righe senza un indirizzo (commenti, titoletti, righe vuote) vengono ignorate.

---

## Esportare i bookmark

```bash
bm export > ~/bookmarks.html
```

oppure il tasto `x` nella TUI. Il file è nel formato standard dei browser (Netscape Bookmark File):
si importa in Firefox, Chrome ecc. Ogni categoria diventa una cartella; vengono conservati data e tag.
Il file esportato può anche essere reimportato in bm.

---

## Ordinamento alfabetico

```bash
bm sort              # tutte le categorie
bm sort linux        # solo una
```

oppure il tasto `s` nella TUI (chiede conferma).

- Ordina per nome; se il nome manca, per indirizzo (senza `https://` e `www.`).
- Non distingue maiuscole/minuscole; le lettere accentate valgono come quelle semplici
  (`Èlite` sta tra `banca` ed `Esempio`).
- L'ordine viene **scritto nei file**, quindi resta. Le righe di commento (`#`) restano in cima,
  le righe vuote vengono tolte.
- Le categorie sono sempre mostrate in ordine alfabetico (sono file letti in ordine di nome).

> L'ordinamento non è reversibile: l'ordine di inserimento originale si perde.
> Se ti interessa, fai prima una copia: `cp -r ~/.segnalibri ~/.segnalibri-backup`

---

## Configurazione

bm si configura con variabili d'ambiente, da mettere nel `~/.bashrc`:

| Variabile | Predefinito | Descrizione |
|---|---|---|
| `BM_DIR` | `~/.segnalibri` | cartella del database |
| `BM_COLORE_NOMI` | `95` (viola chiaro) | colore della prima colonna, come codice ANSI |
| `EDITOR` | `nano` | editor usato da `bm edit` |

Esempi:

```bash
export BM_DIR="$HOME/Nextcloud/segnalibri"
export BM_COLORE_NOMI=97
```

Codici colore utili: `97` bianco, `93` giallo, `92` verde, `96` azzurro, `95` viola, `1;97` bianco
grassetto.

Si possono anche usare al volo per un solo comando, ad esempio per provare un database separato:

```bash
BM_DIR=/tmp/prova bm import ~/bookmarks.html
BM_DIR=/tmp/prova bm
```

---

## Risoluzione dei problemi

**Lanciando `bm` parte un altro programma.**
Esiste un alias, una funzione o un altro comando con lo stesso nome. Controlla con:

```bash
type -a bm
```

Rimuovi l'alias/funzione, oppure installa lo script con un altro nome. Dopo aver modificato il
`~/.bashrc` riapri il terminale: alias e funzioni restano in memoria finché la shell è aperta.

**`'…/.segnalibri' esiste già ma è un file, non una cartella`.**
Il percorso dei dati è occupato da un file di un altro programma. bm non lo tocca: imposta un'altra
cartella con `BM_DIR`.

**`comando non trovato`.**
`~/.local/bin` non è nel `PATH`: vedi il punto 3 dell'[installazione](#installazione).

**Invio non apre il browser.**
Manca `xdg-open`: installa il pacchetto `xdg-utils`. Senza, bm mostra l'indirizzo in basso.

**`y` non copia negli appunti.**
Installa `wl-clipboard` (Wayland) oppure `xclip` / `xsel` (X11).

**I colori o le linee si vedono male.**
Usa un terminale con supporto UTF-8 e 256 colori; per la prima colonna vedi `BM_COLORE_NOMI`.

---

## Limiti noti

- Il campo nome non può contenere il carattere `|` (viene sostituito con `/`).
- I nomi di categoria ammettono solo lettere non accentate, numeri, `.`, `_`, `-`; nell'importazione
  le lettere accentate vengono convertite, gli altri caratteri tolti.
- Caratteri a doppia larghezza (emoji, ideogrammi) possono disallineare leggermente le colonne.
- Gli indici dei comandi da terminale non sono fissi: cambiano dopo eliminazioni, spostamenti e
  ordinamenti.
- La parola "categoria" indica sia i file `.txt` (barra in alto, tasti `←` `→`, comando `mv`), sia,
  come intestazione, la prima colonna dell'elenco, che contiene il nome di ogni bookmark.

---

## Licenza

Copyright (C) 2026 &lt;Eugenio Adami&gt;

bm è **software libero**, distribuito secondo i termini della
**GNU General Public License versione 3** o (a tua scelta) di qualsiasi versione successiva.
Il testo completo è nel file [`LICENSE`](LICENSE).

In breve:

- puoi usare bm per qualsiasi scopo;
- puoi studiarne e modificarne il codice;
- puoi ridistribuirlo, modificato o no;
- se ridistribuisci bm o una sua versione modificata, devi farlo con la stessa licenza GPL e rendere disponibile il codice sorgente.

bm è fornito **senza alcuna garanzia**.
