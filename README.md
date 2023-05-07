# sécurisation interne a la machine:

## le ssh :

dans un premier temps, on va vérifier que ssh est fonctionnel, c'est - à - dire que le service tourne bien et que il écoute sur un port.

```
max@KymonoVps:~$ systemctl status sshd
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2023-05-04 18:26:30 CEST; 2 days ago
       Docs: man:sshd(8)
             man:sshd_config(5)
    Process: 78723 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
   Main PID: 78724 (sshd)
      Tasks: 1 (limit: 2296)
     Memory: 8.9M
        CPU: 7min 45.600s
     CGroup: /system.slice/ssh.service
             └─78724 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups

```

avec la command systemctl, on peut avoir le status d'un service s'il fonctionne ou s'il est innactif, en rajoutant l'argument status. donc, dans notre cas, il est actif, tout va bien.

ensuite, on va voir s'il écoute bien sur un port et lequel avec la command ss

```
max@KymonoVps:~$ sudo ss -lanput | grep sshd
tcp   LISTEN 0      128          0.0.0.0:22          0.0.0.0:*     users:(("sshd",pid=78724,fd=3))                            
tcp   ESTAB  0      52      51.75.205.76:22    91.163.152.89:40722 users:(("sshd",pid=506892,fd=4),("sshd",pid=506866,fd=4))  
tcp   LISTEN 0      128             [::]:22             [::]:*     users:(("sshd",pid=78724,fd=4))     
```

avec la command ss, on voit tous les ports et adresses qui écoutent et sur quel port.

 dans notre cas, on a utilisé la command grep pour n'afficher que les lignes qui nous intéressent en l'occurence celle de sshd, notre service de ssh. donc, cela donne ça :

``` 
max@KymonoVps:~$ sudo ss -lanput | grep sshd
tcp   LISTEN 0      128          0.0.0.0:22          0.0.0.0:*     users:(("sshd",pid=78724,fd=3))                            
tcp   ESTAB  0      52      51.75.205.76:22    91.163.152.89:40722 users:(("sshd",pid=506892,fd=4),("sshd",pid=506866,fd=4))  
tcp   LISTEN 0      128             [::]:22             [::]:*     users:(("sshd",pid=78724,fd=4))     
```

maintenant, passons à la conf du service. on se rend donc a /etc/ssh et le fichier de conf de sshd ce trouvera la bas sous le nom sshd_config

nous allons donc rajouter les option suivantes


```
PermitRootLogin no
MaxSessions 4
PasswordAuthentication no
PermitEmptyPasswords no
```

ce son des règles simple et basique qui sécurise un minimum le service:
- on ne permet pas le login en root
- on autorise 4 session max le nombre de membres de notre groupe pour ne pas permettre plus de session en même temps
- on desactive l'authentification par mots de passe on va utiliser des clef ssh pour la connection
- et on ne permet pas les mots de passe vide


ensuite on genere des clef ssh pour une connexion sans mots de passe : 

La première chose a faire est de générer la clé ssh sur votre machine locale vous pouvez le faire avec cette command:

```
ssh-keygen -t rsa
```

il vous sera poser quelque question tel que securiser la clef avec un mots de phrase vous pouvez remplir ou simplement faire entrer jusqua ce que ceci soit afficher

```
maximeboizot@MacBook-pro-de-Maxime projet-infra % ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/maximeboizot/.ssh/id_rsa): 
/Users/maximeboizot/.ssh/id_rsa already exists.
Overwrite (y/n)? y
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /Users/maximeboizot/.ssh/id_rsa
Your public key has been saved in /Users/maximeboizot/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:+zIreItbv+6pkNspceARYsUhCsRTCtM8aR36Nf5vxpI maximeboizot@MacBook-pro-de-Maxime.local
The key's randomart image is:
+---[RSA 3072]----+
|*+o*+o           |
|o=Xo+            |
|.+oo .o          |
|   .oo .         |
|   ..o. S        |
|    o... .       |
|    o+. oo       |
|    o*ooE++      |
|    ++=*BOo      |
+----[SHA256]-----+
```
et ensuite on tape la commande suivante

```
ssh-copy-id utilisateur@ipduserveur
```

pour mettre la clef publique de ssh sur le server et ensuite on restart le service


```
systemctl restart sshd
```


## fail2ban : 

ensuite on va installer fail2ban pour proteger le ssh du brut force il est aussi possible de le mettre sur d'autre service =ais ici ce sera pour le ssh 

```
sudo apt install fail2ban
```

on install avec cette command 

on demarre le service avec 

```
systemctl start fail2ban
systemctl status fail2ban
```
pour verifier si le service tourne bien

ensuite on va ce rendre dans la conf de fail2ban pour configurer la jail avec sshd dans notre cas

```
sudo nano /etc/fail2ban/jail.d$ cat defaults-debian.conf
```

et nous remplisson le fichier comme ceci 

```
[sshd]
enabled = true
maxretry = 3
findtime = 120
bantime = 1200
```

une fois ça fait on redemmarre le service 

```
systemctl restart fail2ban
```

et plus qu'as test en ce fesant ban (astuce: pour verifier que tt marche baisser le temps de ban)

## firewalld

on install firewalld qui ne l'est pas de base sur debian 

```
sudo dnf install firewalld
```

```
systemctl start firewalld
```

pour demarrer le firewall 

et ensuite on configure comme on le souhaite dans notre cas on va ouvrir les port pour nos service le 80 et le 443 en tcp permanent 

```
sudo firewall-cmd --add-port=80/tcp --permanent
sudo firewall-cmd --add-port=443/tcp --permanent
```

on reload le firewall 

```
sudo firawall-cmd --reload
```

```
sudo firewall-cmd --list-all
```

pour list les règle inscrite

```
max@KymonoVps:~$ sudo firewall-cmd --list-all
[sudo] password for max: 
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens3
  sources: 
  services: dhcpv6-client ssh
  ports: 80/tcp 443/tcp
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules:
```
