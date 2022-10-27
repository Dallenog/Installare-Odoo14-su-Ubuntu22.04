# Guida di instllazione Odoo 14 su Ubuntu 20.04
### Contenuti:

1. [Introduzione](#1-introduzione)
2. [Preparazione del sistema](#1-preparazione-del-sistema)
3. [Creare un utente di sistema](#2-Creare-un-utente-di-sistema)
4. [Configurare PostgreSQL](#4-configurare-postgresql)
5. [Installare Wkhtmltopdf](#5-installare-wkhtmltopdf)
6. [Installare e configurare Odoo 14](#6-Installare-e-configurare-odoo-14)
7. [Aggiornare Odoo 14](#7-aggiornare-Odoo-14)
8. [Addons necessari per la fiscalità italiana](#8-addons-necessari-per-la-fiscalità-italiana)
9. [File di log e di configurazione](#9-file-di-log-e-di-configurazione)
10. [Creare un file di unità Systemd](#10-creare-un-file-di-unità-systemd)
11. [Test dell'installazione](#11-test-dell'installazione)

## 1. Introduzione

Questa guida, ti aiuterà all'installazione di Odoo 14, scaricando Odoo dal repository Github e utilizzando Nginx come proxy inverso.

Questa guida è stata testata utilizzando un server virtualizzato accedendo ad esso con ssh.

## 2. Preparazione del sistema

Accedi a Ubuntu come utente sudo non root per poter completare correttamente questo tutorial.

Aggiorna la cache Apt:
```sh
sudo apt update
```
Installa gli strumenti necessari per l'installazione e la configurazione di Odoo
```sh
sudo apt install git python3-pip build-essential wget python3-dev libxslt-dev libzip-dev libldap2-dev libsasl2-dev python3-setuptools node-less net-tools
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

Installeremo Odoo dal sorgente.

Passiamo all'utente odoo14:
```sh
sudo su - odoo14
```
Clona il codice sorgente di Odoo 14 da GitHub:
```sh
git clone https://www.github.com/odoo/odoo --depth 1 --branch 14.0 /opt/odoo14/odoo
```
Una volta completato il download installa tutti i moduli Python richiesti con pip3:

```sh
pip3 install wheel
```
```sh
pip3 install -r odoo/requirements.txt
```
```sh
pip3 install unidecode codicefiscale asn1crypto pyxb==1.2.6 elementpath openupgradelib xmlschema python-barcode
```
Usciamo dall'utente odoo14
```sh
exit
```
Se si riscontra un errore di compilazione, assicurarsi che tutte le dipendenze richieste elencate nel punto "2. Preparazione del sistema" siano installate.

## 7. aggiornare Odoo 14

Fermiamo il servizio e ci spostiamo nella cartella di installazione
```sh
sudo service odoo14 stop
```
```sh
sudo su - odoo14
```
```sh
cd /opt/odoo14/odoo
```
Verifichiamo la versinone installata
```sh
git branch -a
```
Aggiorniamo
```sh
git pull --all
```
Riavviamo il servizio
```sh
sudo service odoo start
```

## 8. Installare addons necessari

Scarichiamo gli addons per la versione italiana.
```sh
git clone https://github.com/OCA/l10n-italy.git -b 14.0 /opt/odoo14/addons/l10n-italy
```
```sh
git clone https://github.com/OCA/account-financial-tools.git -b 14.0 /opt/odoo14/addons/account-financial-tools
```
```sh
git clone https://github.com/OCA/account-financial-reporting.git -b 14.0 /opt/odoo14/addons/account-financial-reporting
```
```sh
git clone https://github.com/OCA/server-ux.git -b 14.0 /opt/odoo14/addons/server-ux
```
```sh
git clone https://github.com/OCA/partner-contact.git -b 14.0 /opt/odoo14/addons/partner-contact
```
Scarichiamo gli addons per utilizzare il lettore dei codici a barre.
```sh
git clone https://github.com/OCA/stock-logistics-barcode.git -b 14.0 /opt/odoo14/addons/stock-logistics-barcode
git clone https://github.com/OCA/web.git -b 14.0 /opt/odoo14/addons/web
```
Aggiornare la fiscalità italiana
Fermiamo il servizio
```sh
sudo service odoo stop
```
Passiamo all'utente odoo14:
```sh
sudo su - odoo14
```
Aggiorniamo
```sh
cd /opt/odoo14/addons/l10n-italy && git pull --all && cd /opt/odoo14/addons/account-financial-tools && git pull --all && cd /opt/odoo14/addons/account-financial-reporting && git pull --all && cd /opt/odoo14/addons/server-ux && git pull --all && cd /opt/odoo14/addons/partner-contact && git pull --all && cd
```
Riavviamo il servizio
```sh
sudo service odoo stop
```
Gli addons aggiuntivi li andremo a scaricare nella cartella personal_addons che per il momento creiamo
```sh
mkdir /opt/odoo14/addons/personal_addons
```
Per caricare sul server i file scaricati usiamo il seguente comando
```sh
scp -r /home/nome_utente/nome_della_cartella odoo14@192....:/opt/odoo14/addons/personal_addons
```
Torna al tuo utente sudo:
```sh
exit
```

## 9. File di log e di configurazione

Creiamo un file di log
```sh
sudo touch /var/log/odoo14.log
```
Leggiamo il file di log
```sh
less /var/log/odoo14.log
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
addons_path = /opt/odoo14/odoo/addons
               ,/opt/odoo14/addons/personal_addons
               ,/opt/odoo14/addons/l10n-italy
               ,/opt/odoo14/addons/account-financial-tools
               ,/opt/odoo14/addons/account-financial-reporting
               ,/opt/odoo14/addons/server-ux
               ,/opt/odoo14/addons/partner-contact
               ,/opt/odoo14/addons/stock-logistics-barcode
               ,/opt/odoo14/addons/web
logfile = /var/log/odoo14.log
```
Non dimenticare di cambiare my_admin_passwd in qualcosa di più sicuro.

## 10. Creare un file di unità Systemd

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
ExecStart= /opt/odoo14/odoo/odoo-bin -c /etc/odoo14.conf
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
```ini
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
## 11. Test dell'installazione

Se non sai l'indirizzo ip del tuo server digita:
```sh
ifconfig
```
L'output dovrebbe essere simile al seguente.

enp1s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.152.16  netmask 255.255.255.0  broadcast 192.168.162.255
        ...

Il tuo ip è il numero che segue inet

Apri il tuo browser e digita: http://TUO_DOMINIO_O_IP:8069 dovresti visualizzare la pagina di configurazione di Odoo 14.

Continuerà...
