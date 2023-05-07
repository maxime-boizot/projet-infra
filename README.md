
max@KymonoVps:~$ sudo ss -lanput | grep sshd
tcp   LISTEN 0      128          0.0.0.0:22          0.0.0.0:*     users:(("sshd",pid=78724,fd=3))                            
tcp   ESTAB  0      52      51.75.205.76:22    91.163.152.89:40722 users:(("sshd",pid=506892,fd=4),("sshd",pid=506866,fd=4))  
tcp   LISTEN 0      128             [::]:22             [::]:*     users:(("sshd",pid=78724,fd=4))     