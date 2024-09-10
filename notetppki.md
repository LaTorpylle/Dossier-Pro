#TP PKI 07/11 -> 10/11/2023

![Plan réseau](/source/resPki.png "Titre de l'image")

### PFsense

---

#### Crée un certificat par notre certificat Manager
utp.nice.fr serveur nice heure.
activer https
pki:
system certf manager

crée son Certificat Manager
CA
RSA key type

crée un certificat par notre certificat Manager

changer le port
port 666


disable webconfigurator 


pare feu lan vers wan :

laisser passer le ping (icmp), http(TCP), https(TCP), dns(tcp/udp), NTP(TPC).


tous en privé
changer le port
port 666


disable webconfigurator 


pare feu lan vers wan :

laisser passer le ping (icmp), http(TCP), https(TCP), dns(tcp/udp), NTP(TPC).

---

### Installer un windows server2022:

Installer un windows server2022
sconfig option5

sconfig option 5 (désactiver les maj)
installer vm tools avec invite de commande sur esx
bureau à distance + renommer le pc
tous en privé pour le réseau avec secpol.msc -> strat gestionnaire de listes de réseaux
désactiver la sécurité internet exploreur + installer avec ninite(en lab seulement)

---

#### DHCP replica
installer le dhcp classique
clic droit configurer un basculement (pas faire d'étendu)

---

### Intranet 

IIS
HSTS pour la redirection http -> https (cocher la redirection)
Authentification par annuaire lda
ajouter les modules authenfication de base + windows (ajouter role est onctionnalité)

---

### Page web internet
avoir un domaine indiquer l'ip de notre gtw wan
sur Pfsens faire une redirection Interface Nath 
IPV 4 TCP
Destination 

---

### machine www
mettre en statique l'ip et indiquer la gtw
resolv.conf -> pour le dns
apt nmap zip curl lynx dnsutils net-tools git screen samba
/etc/nsswitch.conf hosts ajouts "wins"

---

####Realm
Voir sur le site suivant :

server-world.info/en/note?os=Debian_12&p=realmd
realm join cma.lan -U administrateur (remplace administrator par administrateur)

---

### Apache/Vhost
install LAPS

changer le fichier 000-default.conf 

```c
<VirtualHost *:80>
    ServerName www.example.com
    ServerAlias example.com
    DocumentRoot "/www/domain"
</VirtualHost>

```

a2enmod ssl
a2ensite default-ssl


```c
<VirtualHost *:80>
    Redirect Permanent / https://www.cma.lan 
</VirtualHost>

<VirtualHost *:443>
    ServerName www.example.com
    ServerAlias example.com
    DocumentRoot "/www/domain"
</VirtualHost>
```

création de deux fichier Vhost:
un pour le coté wan
un pour le lan


certifbot 
installer snapd
apt install snapd
snapd install core
snap install --classic certbot
générer la clef et le certificat ssl
changer les les lien de clef et certf


ajouter 2 regles Nat au pfsense wan redirigé vers le serveur web

---

### GPO
pour edge ajouter des amdx pour upload le gestionnaire de stratégie

---

### Sniffer

dossier crée /var/pcap
tcpdump -i -enp2s0 -w dst port 80 toto.pcap
apt install samba 
smb.conf
smbdpasswd -a toto

---

### wiki

LAMPS
phpmyadmin
mediawiki.org

---

### Amélioration des flux 
améliorer les flux dns (53)
TLS DoT

port ldap 389 TCP
ldaps 636

---

### WSUS

-Base de donné ( remonté sql server (payant), sql server express (vers gratuite))

-IIS 

8530 tcp protocole http, la ou les maj transite


GPO

---

### PKI
chaque ordinateur possède un magasin de clef
trustor
je crée le CA primaire, j'éteins tous mes sous clef usb bonne pratique après on travaille avec le certificat intermédiaire.



https://gtw.cma.lan:666

important triade Confidentialité, Authentification, Integrité
SAN subject alternate name 

Anssi rgs RSA 2048   SHA256
Spécifier le nom de l'AC
CMACA

desactiver la comptabilité IE6

Ajout du certificat powershell
```
certreq -submit -attrib "CertificatTemplate:webServer \nSAN:intranet.cma.lan&ipaddress=172.16.1.108" nomdefichier.csr
```

obtention de certifact en .cer

ouvrir le magain de cert :

certlm

help 

certuil -repairestore webHosting Serial_Number 

---

##installation  de cert root sur linux
ssh user@sniffer.cma.lan

su-

/etc/ca-certificates

/user/share/ca-certificates

Avec WinSCP 

protocole SCP

cd/home/user
mv cmaca.cer /usr/share/ca-certificates/mozzilla/

cURL (binaire) ou lynx 

curl https://intranet.cma.lan

cls

---

###Certificat rdp
https://www.it-connect.fr/securite-windows-server-certificat-rdp-adcs/