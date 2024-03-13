This project uses a `GitPod` environment.  

Click the button below to start a new development environment based upon **test** `git` branch:

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/lpiot/gitpod-workspace/tree/test)

# Notes

## Virtualisation

### Objectifs et difficultés

🎯 L'objectif **principal** de la virtualisation est de consolider (_i.e._ optimiser) l'utilisation des serveurs et donc diminuer le rapport coût/usage.  
Cette démarche a différents effets indésirables qu'il va falloir gérer, adresser :

- comment faire cohabiter sur un même serveur des _stacks_ techniques incompatibles entre elles (ex. faire cohabiter `PHP 7.3` et `PHP 8.2`) ?
   - en jouant sur le `PATH` exposé dans la session qui lance les processus applicatifs, on peut faire cohabiter plusieurs _stacks_ (c'est le principe des outils comme `virtualenv` en `Python`). Mais c'est très pénible à gérer car il faut beaucoup d'expertise et de tests pour arriver à le configurer.
- comment isoler les _workloads_ applicatives d'un client/d'un _tenant_ par rapport aux autres ?
   - au niveau _filesystem_ (les droits `UGO` sont une première réponse. `chroot` est un niveau de réponse renforcé)
     ![chroot schema](/images/chroot.jpg)
   - au niveau réseau
   - au niveau processus
   - au niveau du quota de consommation de ressources par chaque processus
- comment gérer sans déployer une énorme expertise sysadmin ?

### Fonctionnement sous-jacent

La virtualisation _per se_ se base sur une **redirection** des appels système adressés à du matériel _virtuel_ vers le matériel physique réel.  
Ainsi une VM possède :

- son propre CPU (avec tout ou partie des capacités du CPU matériel de l'hyperviseur)
- sa propre carte réseau
- son propre disque
- etc.

#### Inconvénients

Cette redirection coûte du temps

- pour router les messages vers le bon matériel,
- pour traduire les messages destinés à un type de matériel virtuel en messages destinés au matériel physique (ex. carte réseau Intel E1000 vers carte réseau physique)
- pour disposer de temps d'exécution sur le matériel en question

La virtualisation est aussi coûteuse en effort d'administration puisque chaque VM réclame l'effort d'administration d'un serveur.

#### Avantages

Malgré tout, la virtualisation apporte une certaine souplesse de manipulation.  
On peut

- cloner une _VM_,
- la déplacer d'un _hyperviseur_ à un autre
- réduire la place disque, voire la place en mémoire RAM utilisées par des _VM_s clonées (fonctionnalité de _copy-on-write_)
- piloter ces manipulations sous forme de scripts et d'automatisations

## Containerisation

La _containerisation_ est un cousin éloigné de la virtualisation.  
En se basant schématiquement sur les mêmes principes de redirection d'appels système,

- mais plus au niveau de matériel virtuel pointant vers du matériel physique
- mais au sein même de l'_OS_, avec des mécanismes d'isolation que sont les `namespaces` et les `cgroups`.

En utilisant ces isolations à une échelle plus fine, sur des éléments plus petits que des composants physique, on dispose d'une technologie de gestion des processus isolés beaucoup plus légère, véloce et peu consommatrice en ressources matérielles et en effort d'administration.

En plus de cela, la technologie des _container images_ permet de **standardiser** la manière dont on manipule une _workload_ applicative :

- dans la manière de la _package_ et de la distribuer
- dans la manière de la _deploy_
- dans la manière de gérer son cycle de vie (démarrage / arrêt / pause / suppression)
- dans la manière dont on _monitor_ son activité

![software stack](/images/software-stack.jpg)

# Glossaire

## blue/green deployment - canary testing/deployment - rolling update

Lorsque la _workload_ est distribuée sur une architecture technique à scalabilité horizontale, sa répartition permet d'envisager des **stratégies de déploiement sphistiquées**, qui permettent de limiter les effets négatifs en cas d'incident au déploiement / à l'_upgrade_. Voire de masquer complètement l'opération aux utilisateurs.

Le _blue/green déployment_ consiste à déployer la nouvelle version (_green_) sur un sous-ensemble des nœuds qui servent la _workload_ applicative (initialement en version _blue_). Ainsi, si 10% des nœuds sont en version _green_, seuls 10% des utilisateurs sont touchés par cette mise à jour. On peut alors contrôler que tout va bien pour cette sous-population de "beta-testeurs" qui s'ignorent et refaire la même opération sur 10% de nœuds complémentaires… jusqu'à avoir remplacé 100% des nœuds servant _blue_ par des nœuds servant _green_.  
On parle aussi de _rolling-update_, dans le sens que cette mise à jour se fait de manière itérative sur l'ensemble des nœuds, en les traitant par échantillons, par sous-ensembles.

Le _canary testing_ ou _canary deployment_ est une variante de cette pratique. La méthodologie est la même. Mais l'objectif est différent.  
Ici, on souhaite exposer une version à une population restreinte d'utilisateurs, à des fins de tests, d'analyse de réaction du marché, etc. Dans ce cas, on n'adresse pas l'ensemble des nœuds, mais on s'en tient à un petit sous-ensemble qui servira d'échantillon pour tests. Des beta-testeurs à pas cher !

![canary testing schema](/images/canary-testing.jpg)

## copy-on-write

Très utilisée dans le monde de la virtualisation, du _Cloud_ et des _containers_.  
Il s'agit d'une fonctionnalité avancée des _filesystems_, qui permet de conserver différentes versions d'un même fichier, non pas en copiant intégralement le fichier v1 dans sa version v2, mais en ne recopiant que la portion qui connaît une évolution / une altération.  
C'est aussi le principe qui est utilisé dans les fonctions de déduplication des baies de stockage.

Pour schématiser, on aura donc :

- le fichier ludo-rox-v1.txt en v1
- un pointeur de fichier ludo-rox-v2.txt qui pointera vers la v1, sauf pour la ligne 42, où le texte a changé.

Ce fonctionnement est valable pour tout type de fichier, avec un impact quasi inexistant sur les performances.  
Gros avantage : si on a 30 versions du même fichier avec, à chaque fois, 1 seule ligne qui varie,

- l'espace de stockage consommé ne sera pas 30 fois la taille du fichier,
- mais 1 fois la taille du fichier + 30 lignes altérées.

Gros gain de place à la clé.  
Gros gain en latence lorsqu'on doit repartir d'un fichier initial qu'on aurait dû dupliquer (dans le cas de clonage de disque de VMs, c'est un gain substentiel qui peut se compter en minutes gagnées !)

## hyperviseur

C'est le logiciel qui permet de gérer la virtualisation sur un serveur.  
Par extension, on parle aussi d'hyperviseur pour désigner un serveur sur lequel on déploie de la virtualisation et des VMs.

## scale-out / scalabilité horizontale

Quand on veut augmenter la puissance disponible pour servir une _workload_, on a plusieurs stratégies.  
On peut augmenter le nombre de serveurs qui peuvent servir la _workload_. Dans ce cas, on parle de scalabilité horizontale ou _scale out_.  
Les requêtes soumises par les utilisateurs sont réparties sur l'ensemble des nœuds (i.e. des serveurs), qui servent la _workload_ applicative. Ainsi, chaque nœud ne sert qu'une partie des utilisateurs.

![scale out schema](/images/scale-out.jpg)

## scale-up / scalabilité verticale

Quand on veut augmenter la puissance disponible pour servir une _workload_, on a plusieurs stratégies.  
On peut augmenter la puissance du serveur, qui sert la _workload_, en ajoutant de la RAM, du disque, des processeurs, en remplaçant les disques par d'autres plus véloces (SSD / flash media). Dans ce cas, on parle de scalabilité verticale ou _scale up_.  
Ce n'est pas toujours faisable :

- le serveur doit rester _up_ et l'ajout de ressources nécessite un redémarrage
- le serveur est déjà doté au maximum, plus aucun slot mémoire ou socket CPU libre

Et surtout, si ça répond (au moins temporairement) aux besoins accrus de performance, ça ne répond pas aux besoins de résilience : si le serveur connaît un incident, la _workload_ ne peut pas être répartie sur un autre serveur.

## tenant

C'est une instance logique (dont la nature fonctionnelle est variable) que l'on souhaite isoler des autres instances.  
On parle de tenants pour distinguer

- les applications dédiées à un client ou un autre,
- les différents environnements techniques d'une même application
- les différents "espaces de travail" au sein d'une même application

## workload (applicative)

Littéralement, la charge de travail. En _IT_, on parle de _workload_ pour évoquer les programmes applicatifs qui sont **en cours d'exécution** sur le serveur.


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
