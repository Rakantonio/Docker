## TP1 conteneurisation et orchestration


### Installation de docker et docker-compose via un script bash
- Créér un script installDocker.sh et ajouter les étapes d'installation de docker et docker-compose.
- Exécuter le script
```
sudo chmod 755 installDocker.sh
sudo ./installDocker.sh
```

### Commandes à tester
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