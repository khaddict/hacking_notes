# Virtual Host Discovery

Lors d’un pentest web, il est fréquent qu’un serveur héberge plusieurs sites sur la même IP grâce aux Virtual Hosts (vhosts).

Le serveur choisit quel site afficher en fonction du header HTTP :

```
Host: academy.htb
```

Si le domaine n’est pas résolu correctement, on peut :

- obtenir une page par défaut Apache/Nginx
- être redirigé bizarrement
- ne pas accéder au vrai site

Ajouter un domaine dans /etc/hosts

```
sudo nano /etc/hosts
```

Ajouter:

```
10.129.229.183 academy.htb
```

Cela permet à la machine locale de résoudre:

```
academy.htb → 10.129.229.183
```

Puis accéder correctement au site.
