# Installazione e configurazione di postgresql e postgis su server Ubuntu 20.04

## Settaggio iniziale del server Ubuntu
Istruzioni tradotte da https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04

In ambiente Linux, l'utente ***root*** è l'utente amministratore e possiede moltissimi privilegi. Per tali motivi è altamente sconsigliato utilizzarlo (rischio di danni irreversibili). Invece è consigliato creare un nuovo utente e loggarsi con quest'ultimo.

### 1) Creazione di un nuovo utente
Una volta loggato come utente ***root*** per creare un nuovo utente ***wilson*** digitare:

```
adduser wilson
```
### 2) Impostazione dei privilegi
Per impostare il nuovo utente ***wilson*** come superuser (che permette di avere i privilegi di amministratore utilizzando la parola ```sudo```) prima di ogni comando, bisogna aggiungere l'utente al gruppo ***sudo***.

Da utente ***root*** digitare il comando:

```
usermod -aG sudo wilson
```

Per switchare tra utenti:

```
sudo -i -u wilson
```

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

### 4) Abilitare l'accesso agli utenti

Una volta che è stato creato un nuovo utente bisogna assicurare l'accesso diretto al server che dipende dalla tipologia di autenticazione scelta. Se ci si logga come root utilizzando una chiave SSH, bisogna aggiungere una copia public key al nuovo utente. Siccome una copia della chiave esiste già per l'account root, basta copiarla per il nuovo account, settare i permessi giusti e modificare la proprietà del file.


```
rsync --archive --chown=wilson:wilson ~/.ssh /home/wilson
```

Adesso è possibile iniziare una nuova sessione loggandosi con il nuovo utente.

## Installazione di Apache Web Server (opzionale, necessario per pgAdmin4)
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

Tutorial tradotto da https://www.digitalocean.com/community/tutorials/how-to-install-postgresql-on-ubuntu-20-04-quickstart

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

E' possibile creare un nuovo utente/ruolo con il comando:

```
sudo -u postgres createuser --interactive
```
oppure, se si è loggati con l'account postgres:

```
createuser --interactive
```

Un altro assunto di PostgreSQL prevede che qualsiasi ruolo utilizzato per loggarsi debba avere un database associato con lo stesso nome. Per creare un nuovo db come utente postgres:

```
sudo -u postgres createdb wilson
```

oppure, se si è loggati con l'account postgres:

```
createdb wilson
```


Adesso è possibile connettersi al nuovo db utilizzando il ruolo creato in precedenza.

```
sudo -i -u wilson
psql
```

Per verificare la connessione:

```
\conninfo
```

## Installazione di pgAdmin4 in server mode (Prima soluzione)

Tutorial tradotto da https://www.digitalocean.com/community/tutorials/how-to-install-configure-pgadmin4-server-mode

Prerequisiti necessari sono 1) un server correttamente configurato 2) Apache installato sul server 3) Postgresql installato e 4) Python3 e ```venv``` installati sul server.

### Installazione di Python3

Ubuntu viene distribuito con python3 installato. Quindi è sufficiente aggiornare i pacchetti e veirificare la versione di python installata.

```
sudo apt update
sudo apt -y upgrade
```
Per verificare la versione:

```
python3 -V
```

Successivamente bisogna installare le dipenendenze ```libgmp3-dev``` ```libpq-dev``` ```libapache2-mod-wsgi-py3```

```
sudo apt install libgmp3-dev libpq-dev libapache2-mod-wsgi-py3
```

Adesso bisogna creare le directory dove pgAdmin andrà a salvare le sessioni, i dati e i log:

```
sudo mkdir -p /var/lib/pgadmin4/sessions
sudo mkdir /var/lib/pgadmin4/storage
sudo mkdir /var/log/pgadmin4
```

E' necessario cambiare la proprietà delle directory e assegnarle ad un utente non-root.

```
sudo chown -R wilson:wilson /var/lib/pgadmin4
sudo chown -R wilson:wilson /var/log/pgadmin4
```

E' necessario installare il modulo ***venv*** al fine di creare un ambiente virtuale nel quale installare pgAdmin4 e creare una directory associata:

```
sudo apt-get install -y python3-venv
mkdir environments
cd environments
```

In questo momento si è all'interno della cartella environments. Da qui si può creare l'ambiente virtuale:

```
python3 -m venv my_env
```

Per attivarlo bisogna eseguire il comando:


```
source my_env/bin/activate
```


### Installazione di pgAdmin4

Bisogna scaricare manualmente il codice sorgente di pgAdmin4 per essere eseguito in python. Cliccare sul seguente link https://www.postgresql.org/ftp/pgadmin/pgadmin4/ scegliere l'utlima versione e navigare nella cartella pip. Copiare il link del file che trmina per .whl e da terminale, tramite il comando wget, scaircare il file all'interno del server.

```
wget https://ftp.postgresql.org/pub/pgadmin/pgadmin4/v4.30/pip/pgadmin4-4.30-py3-none-any.whl
```

Successivamente installare il pacchetto wheel.

```
python -m pip install wheel
```

Installare il pacchetto pgAdmin 4 con il seguente comando:

```
python -m pip install pgadmin4-4.30-py3-none-any.whl
```

### Configurazione di pgAdmin 4

E' possibile configurare pgAdmin 4 modificando il file config.py. Per evitare problemi creeremo il file config_local.py le cui impostazioni verranno lette dopo il primo file.

```
nano my_env/lib/python3.8/site-packages/pgadmin4/config_local.py
```

All'interno dell'editor incollare il seguente contenuto:

```
LOG_FILE = '/var/log/pgadmin4/pgadmin4.log'
SQLITE_PATH = '/var/lib/pgadmin4/pgadmin4.db'
SESSION_DB_PATH = '/var/lib/pgadmin4/sessions'
STORAGE_DIR = '/var/lib/pgadmin4/storage'
SERVER_MODE = True
```

Passo successivo è quello di aggiungere le credenziali per l'accesso a pgAdmin4:

```
python my_env/lib/python3.8/site-packages/pgadmin4/setup.py
```

Disattivare l'ambiente virtuale

```
deactivate
```

I file creati all'interno del file config_local.py devono essere accessibili agli utenti e ai gruppi di utenti che in ubuntu sono www-data (utente webserver di ubuntu)

```
sudo chown -R www-data:www-data /var/lib/pgadmin4/
sudo chown -R www-data:www-data /var/log/pgadmin4/
```

### Configurarazione di Apache

Apache web server utilizza i virtual host per caratterizzare le configurazioni ed opsitare più di un dominio dal singolo server. Settiamo un virtual host specifico per pgAdmin. Spostiamoci nella directory principale:

```
cd
```

Creiamo un nuovo file nella cartella ```/sites-available/``` chiamata ```pgadmin4.conf```

```
sudo nano /etc/apache2/sites-available/pgadmin4.conf
```

Incolliamo il segunete contenuto nel file appena creato (assicurarsi che la versione di python sia quella corretta):

```
<VirtualHost *>
    ServerName your_server_ip

    WSGIDaemonProcess pgadmin processes=1 threads=25 python-home=/home/wilson/environments/my_env
    WSGIScriptAlias / /home/wilson/environments/my_env/lib/python3.8/site-packages/pgadmin4/pgAdmin4.wsgi

    <Directory "/home/wilson/environments/my_env/lib/python3.8/site-packages/pgadmin4/">
        WSGIProcessGroup pgadmin
        WSGIApplicationGroup %{GLOBAL}
        Require all granted
    </Directory>
</VirtualHost>
```

Aggiungere il script ```a2dissite``` per disabilitare il virtual host di default:

```
sudo a2dissite 000-default.conf
```

Ora abilitare il file di configurazione del virtual host pgAdmin:

```
sudo a2ensite pgadmin4.conf
```

Verificare la correttezza della sintassi:

```
apachectl configtest
```

Attivare il virtual host:

```
sudo systemctl restart apache2
```

Digitare l'indirizzo IP in qualsiasi browser e connettersi a pgAdmin 4.

Di default PostgreSQL prende l'user Ubuntu e lo utilizza come username per il database. E' consigliabile settare una password per l'accesso. Dal terminale connettersi a PostgreSQL e settare la password.

```
sudo -u wilson psql

ALTER USER sammy PASSWORD 'password';
```

## Installazione di pgAdmin4 in server mode (Seconda soluzione)
E' possibile installare pgAdmin4 in server mode utilizzando il pacchetto ```pgadmin4-apache2```

```
sudo apt update
sudo apt install pgadmin4 pgadmin4-apache2
```

Durante l'installazione verrà chiesto di inserire l'user e la password.

Riavviare il servizio apache2:

```
sudo systemctl restart apache2
```

Aprire il browser e digitare http://my_server_IP/pgadmin4


## Installazione di PostGIS

Uno dei possibili modi per installare PostGIS su server ubuntu è quello di utilizzare UbuntuGIS. Per prima cosa bisogna aggiungere il repository:

```
sudo add-apt-repository ppa:ubuntugis/ubuntugis-unstable
```

Aggiornare i pacchetti:

sudo add-apt-repository ppa:ubuntugis/ubuntugis-unstable

```
sudo apt-get update
```

Infine installare PostGIS:

```
sudo apt-get install postgis
```

Per abilitare PostGIS si può utilizzare direttamente pgAdmin4.

Per ottimizzare PostGIS bisogna editare il file di configurazione postgresql.conf:

```
sudo nano /etc/postgresql/12/main/postgresql.conf
```

Modificare i parametri:
* shared_buffers = circa il 75% della RAM disponibile;
* work_mem = 16MB
* maintenance_work_mem = 128MB
* checkpoint_segments = 6
* random_page_cost = 2.0

Riavviare il servizio:

```
sudo service postgresql restart
```


