# creation de la clé priv du srv web
se connecter au srv web
- editer le fichier : /etc/ssl/openssl.cnf
```
dir = etc/ssl
```
- créer la clé de certif
```
openssl genrsa -out /etc/ssl/private/srvwebkey.pem 4096
```
