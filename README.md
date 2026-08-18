# Ethernet Cable Tester

Utility desktop multipiattaforma per diagnosticare un collegamento Ethernet senza confondere la velocità del cavo/LAN con quella della connessione Internet.

## Cosa misura

**Quick Check** lavora su un solo computer e mostra l'interfaccia Ethernet attiva, stato del link, IP, MAC, MTU, duplex quando disponibile e velocità negoziata (100 Mb/s, 1 Gb/s, 2.5 Gb/s, ecc.).

**Cable Test** usa la stessa applicazione su due computer della stessa LAN. Un computer avvia `Server`, l'altro esegue `Client`. Il test produce:

- throughput TCP download e upload in Mbit/s, con due stream paralleli di default;
- latenza locale min/media/mediana/p95/max;
- jitter UDP, packet loss, duplicati e pacchetti fuori ordine;
- stabilità a breve termine;
- verdetto diagnostico che separa velocità negoziata e prestazioni misurate.

Il programma **non usa Internet**, non invia telemetria e non richiede `iperf3`.

## Limite diagnostico importante

Un test software non può certificare la categoria fisica del cavo, misurare elettricamente le coppie, fare TDR o stabilire con certezza se il difetto è nel cavo. Può però evidenziare sintomi compatibili con cavo/connettore/porta/NIC: ad esempio una negoziazione bloccata a 100 Mb/s, perdita di pacchetti o prestazioni molto inferiori alla velocità del link.

## Uso

1. Avvia l'app sui due computer collegati alla stessa rete cablata.
2. Sul primo apri **Cable Test** e premi **Start Server**. Il programma mostra gli indirizzi IP locali e la porta (default `52020`).
3. Sul secondo premi **Discover** oppure inserisci manualmente l'IP del primo computer.
4. Premi **Run Test**.
5. Leggi Download, Upload, Latency, Jitter, Packet Loss, Stability e il verdetto finale.

Se il client non si collega, consenti l'app/porta nel firewall locale. Il protocollo di discovery usa UDP `52019`; il benchmark usa la porta TCP configurata più porte TCP/UDP effimere comunicate durante l'handshake.

## Avvio da sorgente

Requisiti: Python 3.11+ con Tkinter.

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate
# macOS/Linux: source .venv/bin/activate
python -m pip install -e .
python -m app.main
```

## Test

Su Linux/headless:

```bash
xvfb-run -a python -m pytest -q
```

Controlli release previsti:

```bash
python -m ruff check app tests
python -m mypy app
```

## Build Windows

La build nativa produce `dist\EthernetCableTester.exe` e non richiede Python installato sul computer dell'utente.

```powershell
powershell -ExecutionPolicy Bypass -File packaging\build_windows.ps1
```

## Build macOS

La build nativa produce `Ethernet Cable Tester.app` e un archivio ZIP distribuibile.

```bash
./packaging/build_macos.sh
```

Le build macOS non firmate possono essere bloccate/avvisate da Gatekeeper. Firma e notarizzazione richiedono credenziali Apple Developer del proprietario. Analogamente, la firma Authenticode Windows richiede un certificato di code signing.

## Build automatica Windows + macOS

`.github/workflows/build.yml` esegue test, lint e type-check, quindi costruisce separatamente su `windows-latest` e `macos-latest` e pubblica gli artefatti della workflow run.

PyInstaller non è un cross-compiler: l'eseguibile Windows deve essere costruito su Windows e l'app macOS su macOS. Per questo il repository contiene build native separate invece di fingere una cross-compilazione dal medesimo host.

## Sicurezza e privacy

- il server ascolta solo dopo un'azione esplicita dell'utente;
- un solo benchmark attivo alla volta;
- frame di controllo e datagrammi hanno limiti rigidi;
- nessun dato ricevuto dal peer viene interpolato in comandi shell;
- nessun trasferimento file, esecuzione comandi o amministrazione remota;
- storico locale SQLite; export JSON/CSV manuale;
- nessun account cloud e nessuna telemetria.
