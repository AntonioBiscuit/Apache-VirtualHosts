# Installation d'un serveur LAMP Debian

- [Installation d'un serveur LAMP Debian](#installation-dun-serveur-lamp-debian)
  - [Installer Apache 2](#installer-apache-2)
  - [Virtual Hosts](#virtual-hosts)
    - [Trouver des IP et une interface disponible](#trouver-des-ip-et-une-interface-disponible)
    - [Configurer les zones virtuelles](#configurer-les-zones-virtuelles)
    - [Créer les arborescences des sites](#créer-les-arborescences-des-sites)
    - [Activer les vHosts](#activer-les-vhosts)
  - [Prochainement](#prochainement)


## Installer Apache 2
`# apt install apache2`

Vérifier si le service Apache est en train de tourner:  
`# systemctl status apache2`
![2](2.png)

Sur un navigateur web, essayer de charger la page en utilisant l'IP du serveur.
![9](9.png)

Les fichiers de configuration se trouvent sous `/etc/apache2/`  
`apache2.conf` : config générale  
`ports.conf` : port d'écoute  

La page web se trouve sous `/var/www/html/`

## Virtual Hosts
Pour avoir plusieurs sites sur la même machine, nous aurons besoin de créer des virtualhosts. Il nous faudra utiliser des IP différentes pour chaque arborescence de site.

### Trouver des IP et une interface disponible


Pinger les IP qui nous intéressent. Si il n'y a pas de réponse, on peut assumer que l'IP est libre pour nous.

`ip a` pour trouver notre interface réseau

Pour ajouter une IP sur une interface en créant une sous interface:  
`ip addr add IP dev INTERFACE label INTERFACE:X`  
Cette IP dure jusqu'au prochain reboot

Si on souhaitait avoir des IP persistantes, il faudrait modifier directement le fichier de config `/etc/network/interfaces.d`

À ce moment on devrait avoir exactement la même page sur un navigateur depuis la première et la seconde IP

### Configurer les zones virtuelles
`cd /etc/apache2/sites-available/`
`ls -l`

`Le fichier 000-default.conf` décrit la donfiguration virtual host par défaut.

Nous allons créer un nouveau fichier qui contiendra notre configuration:
`touch ip_vhosts.conf`
`nano ip_vhosts.conf`

```
<VirtualHost 192.168.122.219:80>
  ServerAdmin web@antoniobiscuit.fr
  DocumentRoot "/var/www/vhosts/site1"
  ServerName site1.antoniobiscuit.fr
  ErrorLog "/var/log/apache2/site1_error_log"
  CustomLog "/var/log/apache2/site1_access_log" combined
</VirtualHost>

<VirtualHost 192.168.122.220:80>
  ServerAdmin web@antoniobiscuit.fr
  DocumentRoot "/var/www/vhosts/site2"
  ServerName site2.antoniobiscuit.fr
  ErrorLog "/var/log/apache2/site2_error_log"
  CustomLog "/var/log/apache2/site2_access_log" combined
</VirtualHost>
```

### Créer les arborescences des sites

`cd /var/www`

Créer des dossiers pour nos sites:
`mkdir -p /var/www/vhosts/site1 /var/www/vhosts/site2`

Nous allons créer une page HTML rudimentaire pour chacun de nos sites:
`nano index.html`

### Activer les vHosts

`a2ensite ip_vhosts`

Si besoin de désactiver en cas de pépin:  
`a2dissite ip_vhosts`

Ne pas mettre l'extension de fichier (.conf)
La commande doit être lancée en tant que root !!!

`systemctl reload apache2`

On devrait maintenant avoir des pages différentes pour chaque IP

## Prochainement

- Avec un même adresse IP et des noms différents
- Avec des noms de domaine