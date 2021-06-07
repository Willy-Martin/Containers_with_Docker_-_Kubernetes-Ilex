# Containers_with_Docker-Ilex

**Document réalisé par Willy MARTIN à destination de l'entreprise ILEX-INTERNATIONAL**

Version 0.01 en date du 02/06/2021

&copy; Copyright WillyMARTIN-2021 

Référence : Remerciement à Sergeï Kudinov pour ses cours et ses exercices pratiques élaborés en 2020. Ce document fait appel à certaines parties de ce cours afin de mettre en évidence la conteneurisation.

## 1. Docker

### 1.0 Installation

Avant de commencer, il est necessaire d'installer docker sur votre système : [Docker Desktop](https://www.docker.com/get-started)

Afin de s'assurer que votre installation est correct, vous pouvez executer la commande suivante :
```
docker run hello-world
```

Si votre installation est correcte, le message suivant s'affichera :

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
b8dfde127a29: Pull complete
Digest: sha256:5122f6204b6a3596e048758cabba3c46b1c937a46b5be6225b835d091b90e46c
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
```

### 1.1 Création d'une image docker à partir d'un dockerfile

En utilisant un terminal positionnez vous dans le dossier [hello-world-docker](./hello-world-docker) téléchargeable à partir de ce répertoire.

Le fichier [DockerFile](./hello-world-docker/Dockerfile) référence les informations necessaires à docker afin de construire l'image. Dans notre cas le fichier copie les documents `server.js` et `package.json` dans notre image afin de pouvoir lancer notre site web node js.

Utilisez la commande ci-dessous afin de créer l'image docker basé sur nos fichiers : `DockerFile`, `server.js` et `package.json`

```
docker build -t hello-world-docker .
```

> Notez que l'attribut suivant `-t` correspond au nom de notre image. De plus `.` correspond à l'emplacement de notre fichier docker. Dans notre cas ce fichier est situé à la racine du dossier.

### 1.2 D'une image à un conteneur. Le lancement de notre site web

Notre Dockerfile se base sur l'image node permettant de monter un site web à partir de nos autres fichiers. Une fois que nous avons créé l'image il ne reste plus qu'a la démarrer. Pour ce faire nous allons utiliser la commande suivante :
```docker run -p 12345:8080 -d hello-world-docker```

Grâce à cette commande nous allons exectuter en backgroud (grâce à l'option `-d`) l'image hello-world-docker dans un conteneur. Notre site web initialement disponible sur le port 8080 (interne au conteneur) sera disponible sur notre machine avec le port 12345.

Votre site doit être disponible sur l'url suivante [http://localhost:12345](http://localhost:12345)

Si ce n'est pas le cas, le conteneur doit être en cours de lancement. Vous pouvez obtenir cette information grâce à la commande `docker ps`

## 2. Minikube

Afin d'utiliser le gestionnaire de conteneur Kubernetes, nous allons installer Minikube. Il interagit comme une VM supportant le cluster Kubernetes. [Lien d'installation](https://minikube.sigs.k8s.io/docs/start/)

Voici quelques commandes interressantes afin de vous assurer que votre système est fonctionnel
```
minikube start       # Permet de démarrer la VM hebergeant kubernetes
minikube status      # Permet de verifier le status de la machine
minikube ip          # Permet de connaitre l'IP de notre VM Minikube
```

## 3. Kubernetes

Kubernetes est représenté en ligne de commande par `kubectl`.

Dans un premier temps nous allons voir les commandes initiales afin d'utiliser kubernetes. Dans un second temps nous verrons l'utilisation de kubernetes à partir de fichier yaml permettant d'industrialiser un déploiement.

### 3.1 Création d'un deploiement à partir d'une image docker

Nous allons mettre en place notre service web préalablement dockeriser dans la partie 1.

**Création d'un deploiment**
```
kubectl create deployment kubernetes-hello-world --image=hello-world-docker
```

**Quelques commandes basiques utiles**
Il sera interresant de gérer les differentes objects de kubernetes. Quelques commandes sont très interressante et nous permettrons de mettre en place et verifier nos systèmes.

```


kubectl get <pods/services ...>      # Cette commande est souvent utilisé. Elle permet notament de voir l'état de nos objects.



kubectl exec -ti $POD_NAME bash     #Permet d'ouvrir un shel d'un pod afin d'y executer des lignes de commandes.
```

### 3.2 Exposer notre service

Notre service web est lancé sur son pod dédier mais il n'est pas encore accesible depuis notre machine local. Il reste local au pod.

Pour cela nous devons creer un service afin d'exposer notre pod. Nous allons utiliser la commande simplifier suivante.

```
kubectl expose deployments/$DEPLOYMENT_NAME --type="NodePort" --port $PORT_NUMBER
```
> Ici $DEPLOYMENT_NAME doit être remplacer par notre nom de deploiement : 'kubernetes-hello-world'
> $PORT_NUMBER est notre port de diffusion côté pod soit 8080

A présent notre site web dockeuriser est disponible. l'adresse IP est celle de notre [VM minikube](`#2-minikube`). le port est celui qui a été attribué par Kubernetes pour ce service, on le retrouve grâce à cette commande `kubectl get services`


### 3.3 Deployment à partir d'un fichier manifest yaml

Dans un premier temps, on s'assure que les élèments précèdents ont été supprimer pour cela :
```
kubectl delete service $SERVICE_NAME
kubectl delete deployment $DEPLOYMENT_NAME
```

Vous retrouverez 2 fichiers dans le dossier kubernetes : `deployment.yaml` et `service.yaml`

Les commandes d'execution sont les suivantes :
```
kubectl apply -f service.yaml
kubectl apply -f deployment.yaml
```
> **Petit test :** quels sont les noms de ces deploiement et services ? Modifiez leur nom afin de mieux comprendre à quoi les servent les diffentents tag.

> De même pour les étapes précèdentes, amusez vous à modifier les ports, les differents tag afin de mieux comprendre les fonctions de chacun.
> Il sera interressant également d'arrêter certains élèments afin de prendre conscience de l'utilité des commandes

