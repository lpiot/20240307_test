This project uses a `GitPod` environment.  

Click the button below to start a new development environment based upon **test** `git` branch:

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/lpiot/gitpod-workspace/tree/test)

# Color. go

## Contexte

Il s'agit d'une application écrite en `Golang`.  
Cette application lance un serveur _Web_ en `HTTP` (port 80 par défaut, configurable dans la variable d'environnement `PORT`).

## 🎯 Objectifs

1. Comprendre ce que fait l'application
1. Compiler l'application
   - en local sur la machine `Gitpod`
  
    ```bash
    cd /workspace/20240307_test/color
    go build -o ./color ./color.go
    ```

   - dans un _container_ en mode interactif
   - dans un _container_ basé sur une image _custom_ spécialement créée pour le besoin
   - dans un _container_ générique dédié à la compilation de binaires `Golang`.
    ```bash
    docker container run --volume /workspace/20240307_test/color:/app golang go build -o /app/color /app/color.go
    ```
 
2. Récupérer le binaire exécutable
3. Lancer le binaire
   - en local sur la machine `Gitpod`
  
    ```bash
    cd /workspace/20240307_test/color
    go run ./color.go
    ```

    ou

    ```bash
    /workspace/20240307_test/color/color
    ```
   - dans un _container_ basé sur une image _custom_ spécialement créée pour le besoin
    ```bash
    echo << EOF
    FROM busybox

    WORKDIR /
    COPY ./color /

    EXPOSE 80
    CMD /color
    EOF

    docker build --tag my-go-image .
    docker run -P my-go-image
    ```
