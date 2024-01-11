# Docker + Symfony 7

## A - PRESENTATION
#### 0. Prerequis
- Docker
- Docker-compose

#### 1. Infos
- Symfony 7 utilise le asset_mapper. Il n'y a donc plus besoin de webpack et par conséquent plus besoin de NodeJS

#### 2. Apps installées
- Apache (Accessible en HTTP seulement pour un projet local)
- PHP 8.2
- MySQL 8.0
- PhpMyAdmin
- Mailcatcher
- Composer 2
- Git
- Symfony CLI

#### 3. Architecture du projet
Le projet possède l'architecture suivante :
```
/apache/default.conf
/app/ (ajouté lors de la création du projet Symfony au point 7)
/mysql/
/php/Dockerfile
.gitignore
docker-compose.yaml
README.md
```
*Le répertoire app sera ajouté lors de la création du nouveau projet Symfony.*

Le fichier `php/Dockerfile` va se charger d'installer "tout ce qu'il faut"...
Le fichier `docker-compose.yaml` va créer les containers et attribuer les identifiants

Ajoutez votre e-mail et votre nom pour la configuration Git en fin de fichier.


## B - INSTALLATION - UTILISATION

#### 1. Installer un nouveau projet Symfony
A la racine du projet :
`symfony new --webapp app`

#### 2. Lancer Docker :
A la racine du projet, dans un terminal :
`docker-compose up`

#### 3. Utiliser la Symfony CLI
1. Se connecter au container : 
`docker-compose exec php /bin/bash`
2. Utiliser la CLI Symfony comme d'habitude (exemple pour créer un controller) : 
`symfony console make:controller`

#### 4. Installer un package Javascript
Exemple pour SplideJS :
`php bin/console importmap:require @splidejs/splide`
ou encore :
`php bin/console importmap:require bootstrap`

`php bin/console importmap:require monpackageJS` va remplir le fichier importmap.php à la racine de Symfony (app/).
Ce fichier est lu et compilé.

Les fichiers du package JS sont placés dans app/assets/vendor

Pour réinstaller les fichiers JS sur un autre serveur : `php bin/console importmap:install`

Vérifier les mises à jour éventuelles de tous les packages JS installés : `php bin/console importmap:outdated`
Vérifier les mises à jour pour un package JS en particulier : `php bin/console importmap:outdated @splidejs/splide`

Mettre à jour les packages JS : `php bin/console importmap:update`
... pour un seul package JS : `php bin/console importmap:update @splidejs/splide`

#### 5. Déployer en PROD
1. `php bin/console importmap:install`
2. `php bin/console sass:build`
3. `php bin/console asset-map:compile`

#### 6. A vérifier...
- est-ce que le serveur utilise HTTP/2 (ou même HTTP/3) ? (A configurer dans Apache)
- est-ce que les assets sont compressés par le serveur web ? Apache devrait utiliser `gzip` (A aconfigurer dans Apache). Un service comme CloudFlare permet également de minifier les fichiers pour réduire la taille 
- est-ce que les Expire headers sont configurés et est-ce qu'ils sont suffisamment longs ? (jusqu'à 1 an) (A configurer dans Apache)


## C - SE CONNECTER AUX APPLICATIONS

#### 1. Les URL
* Page d'accueil Symfony 7 : http://127.0.0.1:8080
* PhpMyAdmin : http://127.0.0.1:8081
* Mailcatcher : http://127.0.0.1:8082

#### 2. MySQL
Host: `127.0.0.1:4306`
Login: `symfony`
Password: `symfony`
Database: `main`

MySQL root password: `secret`

Upload limit: 20 Mb
On peut changer cette limite dans le docker-compose.yaml


## D - ASSET MAPPER et CSS/SCSS
Le CSS est placé dans app/assets/styles
Vous pouvez utiliser directement le fichier app/assets/styles/app.css

Vous pouvez également utiliser du SCSS comme suit dans app/assets/styles :
* renommez le fichier app.css en app.scss
* vous pouvez créer d'autres fichiers CSS tels que variables.css, fonts.css, etc. dans app/assets/styles/ puis vous les importez dans app/assets/app.js, par exemple :
```javascript
import './styles/fonts.css'
import './styles/variables.css'
import './styles/app.css'
```

##### 1. utiliser Live Sass Compiler dans VSCode pour compiler les assets
Tous les fichiers seront dans app/assets/styles/
  
##### 2. installer symfonycasts/sass-bundle
a. `composer require symfonycasts/sass-bundle`

b. Faites pointer votre style dans votre template base.html.twig :
```twig
{% block stylesheets %}
    <link rel="stylesheet" href="{{ asset('styles/app.scss') }}">
{% endblock %}
```
Vérifiez que vous avez bien `app.scss` ici.

c. Buildez les assets en continu :
```bash
php bin/console sass:build --watch
```
(Omettez le --watch pour builder "à la main")

d. Puis, lors du déploiement en PROD, buildez et compilez :
```bash
php bin/console sass:build
php bin/console asset-map:compile
```
Le "compile" copiera physiquement tous les fichiers de vos répertoires mappés vers public/assets/ afin qu'ils soient servis directement par votre serveur Web.
