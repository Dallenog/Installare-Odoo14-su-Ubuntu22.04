# Guida di instllazione Odoo 14 su Ubuntu 22.04
### Contenuti:

1. [Introduzione](##1-introduzione)
2. [Preparazione del sistema](##2-preparazione-del-sistema)
3. [Creare un utente di sistema](#3-Creare-un-utente-di-sistema)
4. [Configurare PostgreSQL](##4-configurare-postgresql)
5. [Installare Wkhtmltopdf](##5-installare-wkhtmltopdf)
6. [Installare e configurare Odoo 14](##6-Installare-e-configurare-odoo-14)
7. [Aggiornare Odoo 14](##7-File-di-log-e-di-configurazione)
8. [Creare un file di unità Systemd](##8-Creare-un-file-di-unità-Systemd)
9. [Aggiornare Odoo 14](##9-Aggiornare-Odoo-14)
10. [Test dell'installazione](##10-test-dell'installazione)

## 1. Introduzione

Questa guida, ti aiuterà all'installazione di Odoo 14.

Questa guida è stata testata utilizzando un server virtualizzato accedendo ad esso con ssh.

## 2. Preparazione del sistema

Aggiungiamp ppa per python 3.8  (utente no root):
```sh
sudo gpg --no-default-keyring --keyring /usr/share/keyrings/deadsnakes.gpg --keyserver keyserver.ubuntu.com --recv-keys F23C5A6CF475977595C89F51BA6932366A755776
```
```sh
sudo echo 'deb [signed-by=/usr/share/keyrings/deadsnakes.gpg] https://ppa.launchpadcontent.net/deadsnakes/ppa/ubuntu jammy main' | sudo tee -a /etc/apt/sources.list.d/python.list
```
Aggiorniamo e instlliamo dipendenze:
```sh
sudo apt update && sudo apt upgrade -y
```
```sh
sudo apt install python3.8-dev nodejs  git build-essential node-less npm python3-pip python3.8-venv python3-wheel python3-setuptools libjpeg-dev libpq-dev liblcms2-dev libwebp-dev libtiff5-dev libjpeg8-dev libopenjp2-7-dev libharfbuzz-dev libfribidi-dev libxcb1-dev libpq-dev libldap2-dev libsasl2-dev libxslt1-dev zlib1g-dev
```
Installiamo e verifichiamo il database:
```sh
sudo apt install postgresql -y
```
```sh
sudo psql --version
```
```sh
sudo systemctl status postgresql
```
## 3. Creare un utente di sistema

Creiamo un utente e impostiamo come sua cartella home /bin/bash
```sh
sudo useradd -m -d /opt/odoo14 -U -r -s /bin/bash odoo14
```
## 4. Configurare PostgreSQL

Creiamo un utente PostgreSQL e lo chiamamo come l'utente di sistema.:
```sh
sudo su - postgres -c "createuser -s odoo14"
```
## 5. Installare Wkhtmltopdf

Scarica il pacchetto:
```sh
sudo wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.jammy_amd64.deb
```
Installa il pacchetto digitando:
```sh
sudo apt install ./wkhtmltox_0.12.6.1-2.jammy_amd64.deb
```
## 6. Installare e configurare Odoo 14

Passiamo all'utente odoo14:
```sh
sudo su - odoo14
```
```sh
git clone https://www.github.com/odoo/odoo --depth 1 --branch 14.0 /opt/odoo14/odoo
```
Una volta completato il download passiamo all'utente odoo14:
```sh
cd /opt/odoo14
```
Creiamo virtual environment per Odoo 14:
```sh
python3.8 -m venv myodoo-venv
```
Attiviamo virtual environment.
```sh
source myodoo-venv/bin/activate
```
Installiamo le dipendenze di Odoo. I moduli Python necessari per l'esecuzione di Odoo 14 sono indicati nel file requirements.txt. Per installarli, eseguire i seguenti comandi:
```sh
(myodoo-venv) $ pip3 install wheel
```
```sh
(myodoo-venv) $ pip3 install -r odoo/requirements.txt
```
Installiamo le dipendenze per la fiscalità italiana:
```sh
pip3 install unidecode codicefiscale asn1crypto pyxb==1.2.6 elementpath openupgradelib xmlschema python-barcode cryptography
```
Dopo aver installato le dipendenze, disattiviamo l'ambiente virtuale:
```sh
(myodoo-venv) $ deactivate
```
Addons aggiuntivi:

Creiamo una cartella dove andremo a copiare gli addons:
```sh
mkdir /opt/odoo14/addons/personal_addons
```
Scarichiamo i moduli per la fiscalità italiana:
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
Scarichiamo i moduli per il lettore dei codici a barre.
```sh
git clone https://github.com/OCA/stock-logistics-barcode.git -b 14.0 /opt/odoo14/addons/stock-logistics-barcode
```
```sh
git clone https://github.com/OCA/web.git -b 14.0 /opt/odoo14/addons/web
```
Usciamo dall'utente odoo14
```sh
exit
```
## 7. File di log e di configurazione

Creiamo un file di log (utente root)
```sh
mkdir /var/log/odoo14
```
```sh
chown odoo14:root /var/log/odoo14
```
Leggiamo il file di log (per uscire alt-q):
```sh
less /var/log/odoo14/odoo.log
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
xmlrpc_port = 8069
addons_path = /opt/odoo14/odoo/addons
               ,/opt/odoo14/addons/personal_addons
               ,/opt/odoo14/addons/l10n-italy
               ,/opt/odoo14/addons/account-financial-tools
               ,/opt/odoo14/addons/account-financial-reporting
               ,/opt/odoo14/addons/server-ux
               ,/opt/odoo14/addons/partner-contact
               ,/opt/odoo14/addons/stock-logistics-barcode
               ,/opt/odoo14/addons/web
logfile = /var/log/odoo14/odoo.log
```
Non dimenticare di cambiare my_admin_passwd in qualcosa di più sicuro.

## 8. Creare un file di unità Systemd

Per gestire il nostro Odoo 14 dobbiamo creare un file systemd:
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
ExecStart=/opt/odoo14/myodoo-venv/bin/python3 /opt/odoo14/odoo/odoo-bin -c /etc/odoo14.conf
StandardOutput=journal+console

[Install]
WantedBy=multi-user.target
```
Notifichiamo a systemd che esiste un nuovo file di unità, avviamo il servizio e lo abilitiamo per l'avvio:
```sh
sudo systemctl daemon-reload
```
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
Se tutto è andato bene puoi cominciare a utilizzare Odoo 14  a questo indirizzo http://TUO_DOMINIO_O_IP:8069.

## 9. Aggiornare Odoo 14

Prima di incominciare facciamo una copia di sicurezza del server.
Fermiamo il servizio e ci spostiamo nella cartella di installazione
```sh
sudo service odoo14 stop
```
```sh
sudo su - odoo14
```
```sh
cd ./odoo
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
Usciamo dall'utente odoo14
```sh
exit
```
Aggiornare moduli fiscalità italiana:
```sh
sudo service odoo14 stop
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
sudo service odoo start
```
Usciamo dall'utente odoo14
```sh
exit
```
Aggiornare moduli codici a barre:
```sh
sudo service odoo14 stop
```
Passiamo all'utente odoo14:
```sh
sudo su - odoo14
```
Aggiorniamo
```sh
cd /opt/odoo14/addons/web && git pull --all && cd /opt/odoo14/addons/stock-logistics-barcode && git pull --all
```
Riavviamo il servizio
```sh
sudo service odoo start
```
Usciamo dall'utente odoo14
```sh
exit
```
## 10. Test dell'installazione

Verifichiamo lo stato del servizio di odoo 14 e del database:
```sh
sudo systemctl status odoo14
```
```sh
sudo systemctl status postgresql
```
Leggere il file di log (per uscire alt-q):
```sh
less /var/log/odoo14/odoo.log
```
Per visualizzare i messaggi registrati dal servizio Odoo, utilizzare il comando seguente:
```sh
sudo journalctl -u odoo14
```
