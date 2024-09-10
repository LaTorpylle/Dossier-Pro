# Étape 1 — Installation d’Easy-RSA
## Installation de Easy-RSA
```
sudo apt update
```
```
sudo apt install easy-rsa
```
# Étape 2 — Préparation d’un répertoire d’infrastructures à clés publiques

## Creation du dossier
```
mkdir ~/easy-rsa
```
## Liens symbolique 
```
ln -s /usr/share/easy-rsa/* ~/easy-rsa/
```
## Restriction du repertoire a moi seul 
```
chmod 700 /home/clement/easy-rsa
```
## Initiation de l'ICP ( Infra a cle publique)
```
cd ~/easy-rsa
```
```
./easyrsa init-pki
```
## Reponse que nous devrions avoir: 
```
Output
init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /home/clement/easy-rsa/pki\
```
# Étape 3 — Création d’une autorité de certification
## Creation du fichier pour les informations de l'organisation
```
cd ~/easy-rsa
```
```
nano vars
```
## Pattern Type :
```
~/easy-rsa/vars
set_var EASYRSA_REQ_COUNTRY    "FR"
set_var EASYRSA_REQ_PROVINCE   "CDVL"
set_var EASYRSA_REQ_CITY       "Orleans"
set_var EASYRSA_REQ_ORG        "Ouicrosoft"
set_var EASYRSA_REQ_EMAIL      "admin@oui.com"
set_var EASYRSA_REQ_OU         "IT"
set_var EASYRSA_ALGO           "ec"
set_var EASYRSA_DIGEST         "sha512"
```
## Creation de la cles root publique et prive pour l'AC 
```
./easyrsa build-ca
```
## Reponse que nous devrions avoir (appuyer sur entree pour laisser par defaut )
```
Output
. . .
Enter New CA Key Passphrase:
Re-Enter New CA Key Passphrase:
. . .
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:

CA creation complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/home/oui/easy-rsa/pki/ca.crt
```
## Permet de ne plus demander le mdp a chaque interaction avec l'AC
```
./easyrsa build-ca nopass
```
# Étape 4 — Distribuer le certificat public de votre autorité de certification
## Permet de voir le code pour importer le CA
```
cat ~/easy-rsa/pki/ca.crt
```
# Étape 5 — Ajout de certificat a la racine
## Creation du fichier ca.crt
```
nano /tmp/ca.crt
```
## Mise au bon endroit du certicat vers les certificats racine
```
cp /tmp/ca.crt /usr/local/share/ca-certificates/
```
## Mise a jour des certificat
```
update-ca-certificates
```
# Étape 6 —Création de demandes de signature de certificats sur le serveur apache2:
## Installation de apache2(+openssl)
```
apt update
apt upgrade
apt install apache2
```
## Creation d'un repertoire 
```
mkdir ~/oui-csr
```
```
cd ~/oui-csr
```
## Creation de la cle privee
```
openssl genrsa -out oui-server.key
```
## Creation du CSR:(remplir les informations)
```
openssl req -new -key oui-server.key -out oui-server.req 
```
## Verification du contenu du CSR
```
openssl req -in oui-server.req -noout -subject
```
## Envoie du .req vers le server AC
```
scp oui-server.req clement@192.168.1.100:/tmp/oui-server.req
```
# Étape 7 —Signature d’une CSR
## Allez dans le repertoire easy-rsa
```
cd ~/easy-rsa
```
## Importation du fichier
```
./easyrsa import-req /tmp/oui-server.req oui-server
```
## Reponse que nous devrions avoir 
```
Output
. . .
The request has been successfully imported with a short name of: oui-server
You may now use this name to perform signing operations on this request.
```
## Signature de la requete
```
./easyrsa sign-req server oui-server
```
## Reponse que nous devrions avoir
```
Output
You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.

Request subject, to be signed as a server certificate for 3650 days:

subject=
    commonName                = www.oui.lan


Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes
. . .
Certificate created at: /home/oui/easy-rsa/pki/issued/oui-server.crt
```
## Renvoie du certificat signer
```
scp pki/issued/oui-server.crt clement@192.168.1.101:/tmp
```
## Creation d'un dossier pour le certificat et la cle privee
```
cd /etc/apache2
```
```
mkdir Ouicrosoft
```
```
cd Ouicrosoft
```
```
Mdkir private
```
```
Mkdir certs
```
## Envoie de mon certificat vers /etc/apache2/Ouicrosoft/certs
```
cd /tmp
```
```
mv oui-server.crt /etc/apache2/Ouicrosoft/certs
```
## Envoie de ma cle privee vers /etc/apache2/Ouicrosoft/private
```
cd ~/oui-csr
```
```
mv oui-server.key /etc/apache2/Ouicrosoft/private/ 
```
## Creation du site qui va heberger le vhost
```
nano /etc/apache2/site-availables/oui.conf
```
## Configuration pour mon Vhost
```
<VirtualHost *:80>
    Redirect permanent / https://www.oui.lan/
</VirtualHost>

<IfModule mod_ssl.c>
    <VirtualHost _default_:443>
        ServerAdmin webmaster@localhost
        ServerName www.oui.lan
        DocumentRoot /var/www/html

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        SSLEngine on
        SSLCertificateFile /etc/apache2/Ouicrosoft/certs/oui-server.crt
        SSLCertificateKeyFile/etc/apache2/Ouicrosoft/private/oui-server.key
        SSLCACertificateFile /usr/local/share/ca-certificates/ca.crt
        
    </VirtualHost>
</IfModule>
```
![](https://i.gyazo.com/53c88131b93160748a16840283ed3ea3.png)

## Activation de ssl sur apache2
```
a2enmod ssl
```
## Desactivation des 2 default conf apache2
```
a2dissite 000-default.conf
```
```
a2dissite default-ssl.conf
```
## Activation du site oui.conf
```
a2ensite oui.conf
```
# Étape 8 —Recuperation du fichier RootCA pour Windows
## Connexion a WinSCP
![](https://i.gyazo.com/13675626c24b2cbe54285b8c7196e796.png)
## Recuperation du certificat ca.crt
![](https://i.gyazo.com/abaa5eb0f039633f088c7b06ce7b5b3b.png)
## Instalation du certificat ca.crt 
### cliquer sur installer le certificat
![](https://i.gyazo.com/a158bdc1ef918ab7fd4f668ca9d78358.png)
![](https://i.gyazo.com/139d53283e69924573c72ec0bdd68926.png)
![](https://i.gyazo.com/ae140d268af74138e01f42a4c04d223f.png)
![](https://i.gyazo.com/259c6e3cc4b8172f3b012994a481ec7f.png)
### Le certificat sera nomme Easy-RSA CA
![](https://i.gyazo.com/8897969972e66be64d78633b03115682.png)
# Verification TLS sur mon serveur apache2
![](https://i.gyazo.com/f84a19d696213e690585c6d0ccf338c6.png)