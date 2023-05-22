> **IMPORTANT**
>
> Ce tuto n'est pas fait par Jeedom SAS mais par un utilisateur de la communauté nous ne pouvons donc garantir qu'il marche ou qu'il est encore d'actualitée.



# Installation de Letsencrypt

Voici les commandes à lancer pour installer letsencrypt avant la génération :

````
apt-get install -y git
cd /opt
git clone https://github.com/letsencrypt/letsencrypt
cd letsencrypt
./letsencrypt-auto --help
````

Pour faire une demande de certificat vous devez posséder un nom de domaine pour lequel il sera généré.

# Configuration d’Apache

Pour que le processus de letsEncrypt se termine correctement, il est nécessaire d’effectuer les trois étapes ci-dessous au préalable :

Attention il est nécessaire d'ouvrir le port 80 sur le routeur (FAI) !

-   Activer le module SSL d’apache de la box Jeedom.
-   Activer le VirtualHost HTTPS d’apache de la box Jeedom .
-   Configurer un portForwarding des requêtes HTTPS sur votre Box internet pour les rediriger vers votre Box Jeedom.

## Activation du virtualHost et du module SSL

> **Note**
>
> Se connecter en SSH sur la box Jeedom.

````
a2enmod ssl
a2ensite default-ssl.conf
service apache2 restart
````

> **Note**
>
> Aucun certificat ne sera délivré par LetsEncrypt tant que votre site en HTTPS ne sera pas joignable de l’extérieur.

``/opt/letsencrypt/letsencrypt-auto --apache --email email@domaine.com -d domaine.com``

Vous devez remplacer les paramètres <email@domaine.com> et domaine.com par vos valeurs. Normalement les paramètres d’ajout du protocole HTTPS sont ajoutés par le script dans Apache.

> **Note**
>
> Si vous utilisez la méthode de renouvellement automatique ci-dessous, vous pouvez désactiver le virtualHost ``default-ssl.conf`` avec la commande ``a2dissite default-ssl.conf`` Pensez à reporter le code par défaut ci-dessous dans le virtualHost créé par le script de renouvellement ``/etc/apache2/sites-available/000-default-le-ssl.conf``

````
<FilesMatch "\.(cgi|shtml|phtml|php)$">
   SSLOptions +StdEnvVars
</FilesMatch>
<Directory /usr/lib/cgi-bin>
   SSLOptions +StdEnvVars
</Directory>
</VirtualHost>
````

# Configuration de Nginx

Cette commande n’est à utiliser que si vous disposez d’un serveur web Nginx.

``./letsencrypt-auto certonly --email email@domaine.com -d domaine.com -a webroot --webroot-path /usr/share/nginx/www/``

Vous devez remplacer les paramètres email et domaine par vos valeurs, ainsi que le chemin vers la racine du serveur. Vous devez ajouter les deux lignes de configuration du HTTPS dans la configuration nginx :

``vi /etc/nginx/sites-enabled/default``

Ajouter les lignes suivantes, entre les lignes ``server {`` et ``root /usr/share/nginx/www ;`` :

````
listen 80;
listen 443 ssl;
ssl_certificate /etc/nginx/ssl/ jeedom.chezmoi.fr.crt;
ssl_certificate_key /etc/nginx/ssl/ jeedom.chezmoi.fr.key;
ssl_session_timeout 5m;
````

Et enfin redémarrez le serveur Nginx.

``service nginx restart``

# Renouvellement

Le renouvellement se fait avec la commande :

``/opt/letsencrypt/letsencrypt-auto --apache --renew-by-default -d mondomaine.fr``

Vous recevez un mail automatiquement à l’arrivée de l’échéance du certificat qui vous rappellera de lancer cette commande.

## Méthode automatique

C’est quand même mieux quand c’est automatique. Pour ce faire, voici les étapes à suivre :

-   Installez **bc**, utilisé dans le script le-renew : ``apt-get install -y bc``
-   Créez un fichier pour y écrire le script (son emplacement est libre) : ``nano /bin/certletsencryptrenew.sh``
-   Saisissez les lignes ci-dessous dans le fichier créé précédemment. Le copier/coller fonctionne via putty. Ce script vérifie l’expiration du certificat et le renouvelle automatiquement si la date d’expiration est à moins de 30 jours. Vous devez remplacer le paramètre domaine.com par votre valeur :
````
    curl -L -o /usr/local/sbin/le-renew https://raw.githubusercontent.com/frixo3190/le-renew/main/le-renew
    chmod +x /usr/local/sbin/le-renew
    le-renew domaine.com
````
-   Sauvegardez le fichier puis quittez l’éditeur de texte, par exemple, avec nano :
````
    ctrl+o -> Entrée     (permet de sauvegarder)
    ctrl+x -> Entrée     (permet de quitter)
````
-   Editez le crontab. Il faut être connecté en root ``crontab -e``
-   On ajoute la ligne suivante : ``0 5 * * 1 /bin/certletsencryptrenew.sh``
> **Important**
>
> Attention à bien adapter le chemin vers le script.

> **Tip**
>
> Pour comprendre la planification ``0 5 * * 1``, allez voir ici et ajustez la à votre besoin si nécessaire :
-   Sauvegardez le fichier puis quittez l’éditeur de texte en sauvegardant :
````
    ctrl+o -> Entrée
    ctrl+x -> Entrée
````
