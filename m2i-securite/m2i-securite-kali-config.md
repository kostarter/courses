##Kali Linux

##### Pourquoi Kali Linux ?
Cette distribution est maintenue et financée par Offensive Security Ltd, elle est l’un des systèmes d’exploitation de piratage éthique les plus populaires et les plus appréciés des pirates et des professionnels de la sécurité.

<p>
  <img src ="https://geekflare.com/wp-content/uploads/2015/09/kali-linux-1200x385.jpg"/>
</p>

- Connectez-vous en SSH à votre poste d'attaque Kali
  (crédentiels fournis par l'instructeur)

```
sudo apt-get update
sudo apt-get install xrdp lxde-core lxde tigervnc-standalone-server -y
sudo nano /etc/xrdp/xrdp.ini
    max_bpp=16
sudo nano /etc/X11/Xwrapper.config
        allowed_users=kali
sudo service xrdp start
sudo passwd kali
```

- Connectez-vous en RDP à votre poste d'attaque Kali

```
ifconfig
```

- Pingez votre Machine THM

```
sudo apt install openvpn

sudo openvpn config.ovpn &

ifconfig
```

- Quelle difference remarquez-vous dans ifconfig?

- Pingez à nouveau votre Machine THM
