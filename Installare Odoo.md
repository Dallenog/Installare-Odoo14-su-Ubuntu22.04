# Guida di instllazione Odoo 14 su Ubuntu 20.04

### Contenuti:
1. [Introduzione](#1-introduzione)
2. [Preparazione del sistema](#1-preparazione-del-sistema)
3. [Creare un utente di sistema](#2-Creare-un-utente-di-sistema)
4. [Configurare PostgreSQL](#4-configurare-postgresql)
5. [Installare Wkhtmltopdf](#5-installare-wkhtmltopdf)
6. [Installare e configurare Odoo 14](#6-Installare-e-configurare-odoo-14)
7. [Addons necessari per la fiscalità italiana](#7-addons-necessari-per-la-fiscalità-italiana)
8. [File di log e di configurazione](#8-file-di-log-e-di-configurazione)
9. [Creare un file di unità Systemd](#9-creare-un-file-di-unità-systemd)
10. [Test dell'installazione](#10-test-dell'installazione)


## 1. Introduzione
L'installazione di Odoo in ambiente virtuale e la distribuzione come contenitore Docker, consente di avere un maggiore controllo sulla configurazione del sistema ed eseguire più versioni di Odoo sullo stesso sistema.

Questa guida, ti guidera attraverso l'installazione e la distribuzione di Odoo 14 all'interno di un ambiente virtuale Python, scaricando Odoo dal repository Github e utilizzando Nginx come proxy inverso.

Questa guida è stata testata utilizzando un server virtualizzato accedendo ad esso con ssh.

## 2. Preparazione del sistema

Accedi a Ubuntu come utente sudo non root per poter completare correttamente questo tutorial.

Aggiorna la cache Apt:

```sh
sudo apt update
```

Installa gli strumenti necessari per l'installazione e la configurazione di Odoo

```sh
sudo apt install git python3-pip build-essential wget python3-dev python3-venv python3-wheel libxslt-dev libzip-dev libldap2-dev libsasl2-dev python3-setuptools node-less net-tools
```

Installa tutte le dipendenze necessarie al funzionamento di Odoo

```sh
sudo apt install adduser fonts-inconsolata fonts-font-awesome fonts-roboto-unhinted libjs-underscore lsb-base postgresql-client python3-html2text python3-pil python3-renderpm python3-tz postgresql udo libpq-dev
```

Ps: Questi pacchetti potrebbero cambiare, se vuoi verificare che siano tutti corretti collegati a https://github.com/odoo/odoo/blob/14.0/debian/control e guarda sotto "Depends:" e "Recommends:"

## 3. Creare un utente di sistema

Crea un utente di sistema che eseguirà Odoo, chiamato odoo14 con home directory /opt/odoo14:

```sh
sudo useradd -m -d /opt/odoo14 -U -r -s /bin/bash odoo14
```

Puoi impostare il nome dell'utente che vuoi, purché successivamente crei un utente PostgreSQL con lo stesso nome.

## 4. Configurare PostgreSQL

Odoo utilizza PostgreSQL come back-end del database, è già stato installato nel punto 2, quindi procediamo con la creazione di un utente PostgreSQL con lo stesso nome dell'utente di sistema precedentemente creato, nel nostro caso odoo14:

```sh
sudo su - postgres -c "createuser -s odoo14"
```

## 5. Installare Wkhtmltopdf

Wkhtmltox fornisce una serie di strumenti da riga di comando open source in grado di eseguire il rendering HTML in PDF e vari formati di immagine. Per poter stampare report PDF, è necessario installare lo strumento wkhtmltopdf. La versione consigliata per Odoo è 0.12.6, che non è disponibile nei repository Ubuntu 20.04 predefiniti.

Scarica il pacchetto:

```sh
sudo wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6-1/wkhtmltox_0.12.6-1.bionic_amd64.deb
```

Installa il pacchetto digitando:

```sh
sudo apt install ./wkhtmltox_0.12.6-1.bionic_amd64.deb
```

## 6. Installare e configurare Odoo 14

Installeremo Odoo dal sorgente all'interno di un ambiente virtuale Python isolato.

Passa all'utente odoo14:

```sh
sudo su - odoo14
```

Clona il codice sorgente di Odoo 14 da GitHub:

```sh
git clone https://www.github.com/odoo/odoo --depth 1 --branch 14.0 /opt/odoo14/odoo
```

Una volta completato il download, crea un nuovo ambiente virtuale Python per Odoo:

Portati nella cartella di installazione

```sh
cd /opt/odoo14
```

Crea l'ambiente virtuale

```sh
python3 -m venv odoo-venv
```

Attiva l'ambiente con il seguente comando:

```sh
source odoo-venv/bin/activate
```

Installa tutti i moduli Python richiesti con pip3:

```sh
pip3 install wheel
```

```sh
pip3 install -r odoo/requirements.txt
```

```sh
pip3 install openupgradelib unidecode codicefiscale asn1crypto pyxb==1.2.6
```

Se si riscontra un errore di compilazione, assicurarsi che tutte le dipendenze richieste elencate nel punto "2. Preparazione del sistema" siano installate.

Una volta fatto, disattiva l'ambiente digitando:

```sh
deactivate
```

Torna al tuo utente sudo:

```sh
exit
```

## 7. Installare addons necessari per la fiscalità italiana

Sacrichiamo gli addons per la versione italiana.

```sh
sudo git clone https://github.com/oca/l10n-italy.git -b 14.0 /opt/odoo14/addons/l10n-italy
```

```sh
sudo git clone https://github.com/OCA/account-financial-tools.git -b 14.0 /opt/odoo14/addons/account-financial-tools
```

```sh
sudo git clone https://github.com/OCA/account-financial-reporting.git -b 14.0 /opt/odoo14/addons/account-financial-reporting
```

```sh
sudo git clone https://github.com/OCA/server-ux.git -b 14.0 /opt/odoo14/addons/server-ux
```

```sh
sudo git clone https://github.com/OCA/partner-contact.git -b 14.0 /opt/odoo14/addons/partner-contact
```
Scarichiamo altri addons dal sito di Odoo.
Creiamo, diamo i permessi di scrittura e ci spostiamo nella cartella di destinazione

```sh
sudo mkdir /opt/odoo14/addons/personal_addons
```
```sh
sudo chmod 7 /opt/odoo14/addons/personal_addons
```
```sh
cd /opt/odoo14/addons/personal_addons
```

Colleghiamoci al sito https://apps.odoo.com/apps oppure https://odoo-community.org/shop/product ,cerchiamo l'addons che ci interessa selezioniamo la versione 14 di Odoo e scarichiamo in locale il file.zip.
Scompattiamo il file e copiamo la cartella sul server, da un terminale in locale digitiamo:

Modifichiamo al seguente comando il nome_utente il nome_della_cartella e l'IP "192...." di destinazione

```sh
scp -r /home/nome_utente/Scaricati/nome_della_cartella nome_utente@192....:/opt/odoo14/addons/personal_addons
```

## 8. File di log e di configurazione

Creiamo un file di log

```sh
sudo touch /var/log/odoo14.log
```
Impostiamo i permessi per l'utente

```sh
sudo chown -R odoo14:odoo14 /opt/odoo14/ /var/log/odoo14.log
```

Creiamo un file di configurazione con il seguente contenuto:

```sh
sudo nano /etc/odoo14.conf
```

```ini
[options]
; This is the password that allows database operations:
admin_passwd = my_admin_passwd
db_host = False
db_port = False
db_user = odoo14
db_password = False
addons_path = /opt/odoo14/odoo/addons,/opt/odoo14/addons/personal_addons,/opt/odoo14/addons/l10n-italy,/opt/odoo14/addons/account-financial-tools,/opt/odoo14/addons/account-financial-reporting,/opt/odoo14/addons/server-ux,/opt/odoo14/addons/partner-contact
logfile = /var/log/odoo14.log
```

Non dimenticare di cambiare my_admin_passwd in qualcosa di più sicuro.

## 9. Creare un file di unità Systemd

Apri l'editor di testo e crea un file di unità di servizio chiamato odoo14.service con il seguente contenuto:

```sh
sudo nano /etc/systemd/system/odoo14.service
```

```ini
[Unit]
Description=Odoo14
Requires=postgresql.service
After=network.target postgresql.service

[Service]
Type=simple
SyslogIdentifier=odoo14
PermissionsStartOnly=true
User=odoo14
Group=odoo14
ExecStart=/opt/odoo14/odoo-venv/bin/python3 /opt/odoo14/odoo/odoo-bin -c /etc/odoo14.conf
StandardOutput=journal+console

[Install]
WantedBy=multi-user.target
```

Notifichiamo a systemd che esiste un nuovo file di unità:

```sh
sudo systemctl daemon-reload
```

Avviamo il servizio Odoo e abilitalo per l'avvio all'avvio eseguendo:

```sh
sudo systemctl enable --now odoo14
```

Verifichimo lo stato del servizio:

```sh
sudo systemctl status odoo14
```

L'output dovrebbe essere simile al seguente.

```sh
● odoo14.service - Odoo14
     Loaded: loaded (/etc/systemd/system/odoo14.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2020-10-17 19:25:47 CEST; 2s ago
   Main PID: 11987 (python3)
      Tasks: 4 (limit: 2286)
     Memory: 82.2M
     CGroup: /system.slice/odoo14.service
             └─11987 /opt/odoo14/odoo-venv/bin/python3 /opt/odoo14/odoo/odoo-bin -c /etc/odoo14.conf
```

Per visualizzare i messaggi registrati dal servizio Odoo, utilizzare il comando seguente:

```sh
sudo journalctl -u odoo14
```

## 10. Test dell'installazione

Se non sai l'indirizzo ip del tuo server digita:

```sh
ifconfig
```

L'output dovrebbe essere simile al seguente.

```sh
enp1s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.152.16  netmask 255.255.255.0  broadcast 192.168.162.255
        ...
```

Il tuo ip è il numero che segue inet

Apri il tuo browser e digita: http://TUO_DOMINIO_O_IP:8069 dovresti visualizzare la pagina di configurazione di Odoo 14.

Continuerà...
