# TP1 conteneurisation et orchestration


## Installation de docker et docker-compose via un script bash

https://docs.docker.com/engine/install/ubuntu/
https://docs.docker.com/compose/install/linux/

- Créer un script installDocker.sh
- Lancer le script 

```bash
chmod 755 installDocker.sh
./installDocker.sh
```

## Quelques commandes à tester
```
docker run hello-world
```
![](https://i.imgur.com/6abfUSl.png)

```
docker run -it ubuntu bash
```
![](https://i.imgur.com/8svtVna.png)

```
docker images
```
![](https://i.imgur.com/1zZTurU.png)

```
docker ps -a
```
![](https://i.imgur.com/DklFqCT.png)

```
docker run -p 80:80 nginx
```
![](https://i.imgur.com/d5kDTV8.png)
![](https://i.imgur.com/S2kHuWu.png)

```
docker run -d -p 80:80 nginx
```
![](https://i.imgur.com/xSqxjst.png)

![](https://i.imgur.com/DOfL9aJ.png)


## 1. Exécuter un serveur web (apache, nginx, …) dans un conteneur docker

### a. Récupérer l’image sur le Docker Hub

https://hub.docker.com/_/nginx

```bash=
root@WINDELL-5N29K7U:~/Conteneurisation-Docker$ docker pull nginx
Using default tag: latest
latest: Pulling from library/nginxDigest: sha256:75263be7e5846fc69cb6c42553ff9c93d653d769b94917dbda71d42d3f3c00d3
Status: Image is up to date for nginx:latest
```



### b. Vérifier que cette image est présente en local

```bash=
# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
nginx         latest    3964ce7b8458   32 hours ago    142MB
ubuntu        latest    6b7dfa7e8fdb   6 days ago      77.8MB
hello-world   latest    feb5d9fea6a5   14 months ago   13.3kB
```

### c. Créer un fichier index.html simple
```bash
mkidr nginx
touch index.html
```

### d. Démarrer un conteneur et servir la page html créée précédemment à l’aide d’un volume (option -v de docker run)

```bash
docker run --name srv-nginx -d -p 80:80 -v /home/kube/nginx/index.html:/usr/share/nginx/html/index.html nginx
d5e1385e096f31e9f0c7b2b418c74d158efde040e164122f03890dbaca2e7f97
```
![](https://i.imgur.com/fJC9gsD.png)

### e. Supprimer le conteneur précédent et arriver au même résultat que précédemment à l’aide de la commande docker cp

```bash
docker stop srv-nginx
srv-nginx

docker rm srv-nginx
srv-nginx

docker run --name srv-nginx -d -p 80:80 nginx
77a4126fc24e6bf11374ad9f1aacc278c035ba065f31795133f43ad17ad59181

docker cp /home/kube/nginx/index.html srv-nginx:/usr/share/nginx/html/index.html

docker exec -ti srv-nginx /bin/bash

cat /usr/share/nginx/html/index.html
coucou
```

## 2. Builder une image

### a. A l’aide d’un Dockerfile, créer une image (commande docker build)

```dockerfile
vim nginx/Dockerfile

FROM nginx
COPY index.html /usr/share/nginx/html/index.html
```
```bash
docker build . -t custom-nginx
docker images
REPOSITORY     TAG       IMAGE ID       CREATED              SIZE
custom-nginx   latest    98c140f7c034   About a minute ago   142MB
nginx          latest    3964ce7b8458   33 hours ago         142MB
```

### b. Exécuter cette nouvelle image de manière à servir la page html (commande docker run)

```
docker run --name srv-nginx -p 80:80 -d custom-nginx
42325bbda33dae01504e25e966c23f38f3856feb22f32ca1f029360214cc7143
```

### c. Quelles différences observez-vous entre les procédures 1. et 2. ? Quels sont les avantages et inconvénients de l’une et de l’autre méthode ? (Mettre en relation ce qui est observé avec ce qui a été présenté pendant le cours)

> La procédure 1 utilise une image déjà existante. Nous pouvons ensuite préciser le volume directement dans la commande. Les fichiers ajoutés dans le volume local seront ajoutés directement dans le volume du conteneur.

> La procédure 2 permet de créer une image personnalisée. Nous pouvons définir les instructions que nous voulons dans le Dockerfile pour procéder à la création de l'image. Il faudra cependant modifier le Dockerfile si on veut personnaliser de nouveau l'image.

## 3. Utiliser une base de données dans un conteneur docker

### a. Récupérer les images mysql:5.7 et phpmyadmin/phpmyadmin depuis le Docker Hub

```
docker pull mysql:5.7
docker pull phpmyadmin
docker images
REPOSITORY     TAG       IMAGE ID       CREATED          SIZE
mysql          5.7       d410f4167eea   8 days ago       495MB
phpmyadmin     latest    8907e33feea6   8 days ago       511MB
```

### b. Exécuter deux conteneurs à partir des images et ajouter une table ainsi que quelques enregistrements dans la base de données à l’aide de phpmyadmin

```
docker run --name mysql-srv -p 3306:3306 -e MYSQL_ROOT_PASSWORD=P@ssw0rd
s -d mysql:5.7
a94834027e500ecae09d9b9d9ca3f0c69bb1ddfe28eecc87bcfd35f9772cd9a9

docker run --name=phpmyadmin -d --link mysql-srv:db -p 8080:80 phpmyadmin/phpmyadmin
b9f9848b9a7b402f07c0e7f5e6d324422d4ea474927206a87713be997058f475
```

> Création de la base de données *test*, la table *tabledetest*, la colonne *macolonne* avec la valeur *coucou*

![](https://i.imgur.com/2O4bL34.png)


## 4. Faire la même chose que précédemment en utilisant un fichier docker-compose

```yaml
version: '3.1'

services:

  db:
    image: mysql:5.7
    container_name: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: P@ssw0rd
      MYSQL_DATABASE: test
    ports:
      - 3306:3306

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadmin
    restart: always
    ports:
      - 8080:80
    environment:
      - PMA_ARBITRARY= 1
    depends_on:
      - db
```
```
docker compose up -d
[+] Running 3/3
 ⠿ Network compose_default  Created                                                                                                                                            0.0s
 ⠿ Container mysql          Started                                                                                                                                            0.3s
 ⠿ Container phpmyadmin     Started


### a. Qu’apporte le fichier docker-compose par rapport aux commandes docker run ? Pourquoi est-il intéressant ? (cf. ce qui a été présenté pendant le cours)
- Le fichier est plus structuré et donne donc plus de visibilité à la configuration par rapport au CLI.
- Nous n'avons pas besoin de lancer 2 commandes, les conteneurs se lancent directement à la suite
- En faisant un docker compose down, cela supprime tous les conteneurs qui ont été lancés dans le fichier (Gain en temps)

### b. Quel moyen permet de configurer (premier utilisateur, première base de données, mot de passe root, …) facilement le conteneur mysql au lancement ?
> En utilisant l'environnement
