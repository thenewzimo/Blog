# 🕵️ Underpass CTF - Walkthrough

## 🔍 Primo Step

Ho eseguito una scansione con `nmap` e ho rilevato due porte aperte:

- **🛡️ Porta 22** → SSH
- **🌐 Porta 80** → HTTP

Accedendo al sito sulla porta 80, ho scoperto che tutte le sezioni erano bloccate da Apache. Ho eseguito `gobuster`, ma non ho trovato nulla di utile.

Successivamente, ho provato una scansione `nmap` su **UDP**, scoprendo alcune porte interessanti, tra cui una con un servizio **SNMP**.

## 📡 Enumerazione con SNMP

Utilizzando `snmpwalk`, ho ottenuto diverse informazioni, tra cui:

- 🧑‍💻 Un utente chiamato **Steve**.
- 🎛️ La presenza di un server **RADIUS** su un'altra porta.

Dall'analisi del servizio, ho scoperto che il server in uso era **Daloradius**. Secondo la documentazione, il percorso di default è `Sito-web/daloradius`. Tuttavia, provando ad accedere via browser, Apache lo bloccava.

A questo punto, ho avviato `dirbuster` con una scansione più approfondita e ho trovato un file interessante: `login.php`.

## 🔑 Accesso a Daloradius

Dalla documentazione di Daloradius, le credenziali di default sono:

- **🔐 Admin: admin**
- **🔐 Administrator: radius**

Ho provato la seconda opzione e sono riuscito ad accedere al pannello di amministrazione.

All'interno ho trovato un altro utente ""
La sua password era hashata, quindi l'ho craccata con `hashcat` e ho ottenuto la password in chiaro. 

## 🖥️ Accesso SSH

Utilizzando l'utente trovato e la password craccata, sono riuscito a connettermi via SSH:

```bash
ssh steve@10.10.11.XX
```

All'interno della home directory di **Steve**, ho trovato la flag **🏴 user.txt**.

## 🚀 Privilege Escalation

Per verificare i privilegi dell'utente, ho eseguito:

```bash
sudo -l
```

Ho scoperto che potevo eseguire `mosh-server` senza richiedere la password di root.

Ho avviato **mosh** con il comando:

```bash
mosh --server="sudo /usr/bin/mosh-server" localhost
```

🔥 Questo mi ha fornito una shell con privilegi di **root**!

All'interno della directory `/root/`, ho trovato la flag **🏴 root.txt**.

---

🎯 **CTF completata con successo!** 🚀
