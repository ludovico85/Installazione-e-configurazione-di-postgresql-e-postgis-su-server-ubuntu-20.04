## Installazione e configurazione di postgresql e postgis su server ubuntu 20.04

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

Per aggiungere un nuovo ruolo e aggiungere il nome del nuovo user

```
sudo -u postgres createuser --interactive
```

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

