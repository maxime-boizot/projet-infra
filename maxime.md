# sécurisation interne a la machine:

## le ssh :

dans un premier temps on va verifier que ssh est fonctionnel c'est a dire que le service tourne bien et que il écoute sur un port

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

avec la command systemctl on peux en rajoutant l'agument status avoir le status d'un service si il fonctionne ou si il est innactif. donc dans notre cas il est actif tt va bien

ensuite on va voir si il écoute bien sur un port et lequel avec la command ss

```
max@KymonoVps:~$ sudo ss -lanput | grep sshd
tcp   LISTEN 0      128          0.0.0.0:22          0.0.0.0:*     users:(("sshd",pid=78724,fd=3))                            
tcp   ESTAB  0      52      51.75.205.76:22    91.163.152.89:40722 users:(("sshd",pid=506892,fd=4),("sshd",pid=506866,fd=4))  
tcp   LISTEN 0      128             [::]:22             [::]:*     users:(("sshd",pid=78724,fd=4))     
```

bon avec la command ss on vois toutles port et adresses qui écoute qui écoute sur quel port

donc cela donne cela dans notre cas on a utilisé la command grep pour n'afficher que les lignes qui nous interesse

``` 
max@KymonoVps:~$ sudo ss -lanput | grep sshd
tcp   LISTEN 0      128          0.0.0.0:22          0.0.0.0:*     users:(("sshd",pid=78724,fd=3))                            
tcp   ESTAB  0      52      51.75.205.76:22    91.163.152.89:40722 users:(("sshd",pid=506892,fd=4),("sshd",pid=506866,fd=4))  
tcp   LISTEN 0      128             [::]:22             [::]:*     users:(("sshd",pid=78724,fd=4))     
```