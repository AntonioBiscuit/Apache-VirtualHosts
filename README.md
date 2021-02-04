# Installation d'un serveur LAMP Debian

- [Installation d'un serveur LAMP Debian](#installation-dun-serveur-lamp-debian)
  - [Installer Apache 2](#installer-apache-2)
  - [Virtual Hosts](#virtual-hosts)
    - [Trouver des IP et une interface disponible](#trouver-des-ip-et-une-interface-disponible)
    - [Configurer les zones virtuelles](#configurer-les-zones-virtuelles)
    - [Créer les arborescences des sites](#créer-les-arborescences-des-sites)
    - [Activer les vHosts](#activer-les-vhosts)
  - [Prochainement](#prochainement)
  - [Même chose mais avec une seule adresse IP et de ports différents](#même-chose-mais-avec-une-seule-adresse-ip-et-de-ports-différents)
  - [Avec des noms de domaine](#avec-des-noms-de-domaine)


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

- Avec une même adresse IP et des noms différents
- Avec des noms de domaine

----------

## Même chose mais avec une seule adresse IP et de ports différents

Par défaut, Apache utilise le port 80 pour HTTP et 443 pour HTTPS.
La configuration des ports utilisés se trouve dans le fichier:

`/etc/apache2/ports.conf`

On va ouvrir deux numéros de port différents pour nos deux arbrorescences différentes.

On doit enlever ce qu'on a fait au tout début avec les IP différentes, désactiver l'ancien fichier  `ip_vhosts` avec:

`a2dissite ip_vhosts`

et recharger le Apache2 avec `systemctl reload apache2`

On devrait maintenant retrouver la page par défaut de Apache sur le port par défaut.

`cd /apache2/sites-available`
`ls-la`

On peut réutiliser le fichier `ip_vhosts.conf` et en faire une copie `port_hosts.conf`. On modifiera ce qu'il faut ou créera un nouveau fichier entier.

```
<VirtualHost 192.168.122.219:80>
  ServerAdmin web@antoniobiscuit.fr
  DocumentRoot "/var/www/vhosts/site1"
  ServerName site1.antoniobiscuit.fr
  ErrorLog "/var/log/apache2/site1_error_log"
  CustomLog "/var/log/apache2/site1_access_log" combined
</VirtualHost>

<VirtualHost 192.168.122.219:8080>
  ServerAdmin web@antoniobiscuit.fr
  DocumentRoot "/var/www/vhosts/site2"
  ServerName site2.antoniobiscuit.fr
  ErrorLog "/var/log/apache2/site2_error_log"
  CustomLog "/var/log/apache2/site2_access_log" combined
</VirtualHost>
```

On utilise maintenant la même adresse IP pour les deux arborescences mais on utilise deux numéros de port différents au lieu de deux IP différentes.

On peut modifier les pages web de sorte à inclure le numéro de port dans la struture

`a2ensite port_hosts`

Pour le moment, Apache2 n'écoute encore que sur les ports 80 et 443. ON doit alors modifier un fichie de config pour activer cela.

Editer et ajouter simplement le port 8080 en dessous du port 80:
`/etc/apache2/ports.conf`

```
Listen 80
Listen 8080
```

On doit redémarrer entièrement le service car le fichier de conf n'est lu qu'au démarrage

`systemctl restart apache2`

On devrait maintenant avoir nos deux sites qui s'affichent selon le numéro de port choisi.

----------

## Avec des noms de domaine

Désactiver l'autre méthode

`a2dissite port_vhosts`

`systemctl reload apache2`

On utilisera une seule adresse IP mais des noms DNS.

On a besoin d'un service DNS. On utilise le service DNS qui est donné par notre connexion.

Fichier hosts
On peut déclarer des noms de domaine

cd `/etc/apache2/sites-available`

ls

On copie le fichier ports_vhosts.conf en `name_vhosts.conf`

On fait ensuite les modifications nécessaires:

```
<VirtualHost *:80>
  ServerAdmin web@antoniobiscuit.fr   
  DocumentRoot "/var/www/vhosts/site1" 
  ServerName site1.antoniobiscuit.local
  ErrorLog "/var/log/apache2/site1_error_log"
  CustomLog "/var/log/apache2/site1_access_log" combined
</VirtualHost>

<VirtualHost *:80>
  ServerAdmin web@antoniobiscuit.fr   
  DocumentRoot "/var/www/vhosts/site2" 
  ServerName site2.antoniobiscuit.local
  ErrorLog "/var/log/apache2/site2_error_log"
  CustomLog "/var/log/apache2/site2_access_log" combined
</VirtualHost>
```

On modifie optionnellement les sites pour savoir où on atterit

On active enfin notre fichier conf: `a2ensite name_vhosts`
`systemctl reload apache2`


Sur Windows on doit éditer le fichier `hosts` et rajouter l'IP de notre serveur

Sur Windows il se trouve sous: `c:\windows\system32\drivers\etc\hosts`  
Sur Linux il se trouve sous `\etc\hosts`

exemple:

`192.122.101.2 site1.antoniobiscuit.local site2.antoniobiscuit.local`