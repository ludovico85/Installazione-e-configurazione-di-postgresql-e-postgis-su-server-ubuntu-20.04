# Installazione e configurazione di postgresql e postgis su server ubuntu 20.04

## Settaggio iniziale del server Ubuntu
Istruzioni tradotte da https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04

In ambiente Linux, l'utente ***root*** è l'utente amministratore e possiede moltissimi privilegi. Per tali motivi è altamente sconsigliato utilizzarlo (rischio di danni irreversibili). Invece è consigliato creare un nuovo utente e loggarsi con quest'ultimo.

### 1) Creazione di un nuovo utente
Una volta loggato come utente ***root*** per creare un nuovo utente ***wilson*** digitare:

```
add user wilson
```
### 2) Impostazione dei privilegi
Per impostare il nuovo utente ***wilson*** come superuser (che permette di avere i privilegi di amministratore utilizzando la parola ```sudo```) prima di ogni comando, bisogna aggiungere l'utente al gruppo ***sudo***.

Da utente ***root*** digitare il comando:

```
usermod -aG sudo wilson
```

Pee switchare

### 3) Impostazione del Firewall
Ufw è l'applicazione predefinita di Ubuntu per la configurazione del firewall. Per vedere la lista della applicazioni gestite da Ufw:

```
ufw app list
```

Dobbiamo assicurarci che il firewall permetta le connessioni SSH per consertirci di loggarci al prossimo avvio.

```
ufw allow OpenSSH
```

Abilitare il firewall:

```
ufw enable
```

Per verificare lo stato e le connessioni consentite:

```
ufw status
```

***Il firewall in questo momento blocca tutte le connessioni in entrata tranne che SSH;*** Nel caso di installazione di servizi addizzionali è necessario modificare le impostazioni del firewall.

## Installazione di Apache Web Server (opzionale, necessario per pgadmin4)
Istruzioni tradotte da https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-20-04

Per installare Apache digitare:

```
sudo apt install apache2
```

Modificare le impostazioni del firewall:

```
sudo ufw app list
```

```
sudo ufw allow 'Apache'
```

Per verificare lo stato di Apache digitare:

```
sudo systemctl status apache2
```

Per mostrare gli indirizzi IP:

```
hostname -I
```

Per stoppare, avviare (quando stoppato) e riavviare (stoppare e avviare il servizio) digitare:

```
sudo systemctl stop apache2
```

```
sudo systemctl start apache2
```

```
sudo systemctl restart apache2
```

Di default il sistema è settato per avviare Apache in automatico. Per disattivare tale comportamento:

```
sudo systemctl disable apache2
```

Per riabilitarlo:

```
sudo systemctl enable apache2
```

## Installazione di PostgreSQL

Per prima cosa è necessario aggiornare i pacchetti di ubuntu

```
sudo apt update
```

Per installare PostgreSQL insieme ad altre utility, aggiungere ```-contrib``` all'installazione.

```
sudo apt install postgresql postgresql-contrib
```
Di default postgres utilizza il concetto di "ruoli" per gestire le autenticazioni e le autorizzazioni. La procedura di installazione crea un account di default chiamato postgres associato al ruolo di defualt Postgres. Per abilitare il ruolo digitare:

```
sudo -i -u postgres
```

Per accedere la prompt dei comandi e interagire con la gestione dei database:

```
psql
```

Per fare il logout dell'account postgres:

```
\q
```

Per tornare al prompt di ubuntu:

```
exit
```

Per collegarsi direttamente all'account postgres

```
sudo -u postgres psql
```

Di default l'account postgres non ha alcuna password settata. Per settare una nuova password digitare:
```
postgres=# ALTER USER postgres PASSWORD 'myPassword';
```

Per aggiungere un nuovo ruolo e aggiungere il nome del nuovo user

```
sudo -u postgres createuser --interactive --pwprompt
```

## installare pgadmin4 in server mode


Per installare PgAdmin4, installare prima curl

```
sudo apt install curl
```

PgAdmin4 non è disponibile di default nei repo di Ubuntu. Bisonga installarlo dal repo di pgAdmin4 APT.

```
curl https://www.pgadmin.org/static/packages_pgadmin_org.pub | sudo apt-key add
sudo sh -c 'echo "deb https://ftp.postgresql.org/pub/pgadmin/pgadmin4/apt/$(lsb_release -cs) pgadmin4 main" > /etc/apt/sources.list.d/pgadmin4.list && apt update'
sudo apt install pgadmin4
```
