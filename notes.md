# Notes lecture d'article algoHPC

L'article s'intéresse à différente politique d'accès au noeud de données en fonction des patternes d'accès au données par l'application.

L'article propose une politique basé sur le problème du sac à dos (Knapsack problem) à choix multiple. Ce problem cherche à maximisé la bandpassat global en donnant plus de noeud I/O au applications qui en on le plus besoin.

Le context de l'article est :

- Les applications HPC font de plus en plus d'accès au données nommé (I/O)
- Il y a pliens de type d'applications (accès hétérogénéité aux données, IA, bigData)
- on augment le nombre de moeux de calculs pour avoir plus de performance mais le PFS ne peut pas scale pour autant de noeud. (je suppose une histoire de broadcast qui faut mettre en multicast?)
- le \emph{I/O forwarding} cherche a corriger le problème en diminuent le nombre de machine qui accéde au PFS.

NInt : noeud interface
Ncal : noeud de calculs

Le \emph{I/O forwarding} consiste à une groupe de noeud qui resoive les requétes de
l'application et les transmet au system de fichier paralléle. Permet aussi de changer la form
des requétes (Ex : au lieu de plusieurs petite requétes on en fait une seul grand).

Le \emph{I/O forwarding} :

- une technique très utilisé dans le HPC
- Cette technique permet d'augmenté les performances
- évite les accès direct au machine de stockage parallèle
- NInt ce met entre les noeuds de calculs et les noeud PFS
- Ajouté des Nint permet de controlé les demand d'accès au données et d'appliqué des théchniques d'optimisation
- c'est transparent pour les utilisateurs (app, FS...)
- diminuent la contention à accès au serveurs de stockage de données.
- try to répatition uniform ?
- utilisé par des machine du top 500

les théchniques d'optimisation :

- la planification des demandes
- l'agrégation des demandes.

La technique habituelle est d'assigné statiquement un noeud de stockage à un noeud de calculs.
Ces liens ce correspond pas tous le temps au transfére de données d'on l'appication à besoin.

Le pb :

- la connection entre les NInt et les Ncal est statique
- Donc l'application peut utilisé qu'un nombre prédéfinis de noeud I/O est communique avec un seul NInt.
- pas assé flexible
- ça peut amené a de mauvaise allocation des ressources (Yu et al. [8] Bez et al. [9])
- La bande passant est déterminé par le fonctionnement des applications (plus ou moins de calcule avec plus ou moins d'accès au données...) ce qui fait qu'avec un nombre statique de noeud ou peut ce retouvé à avoir top de noeud I/O et donc une bande passant pas utilisé a sont pliens potenciel. (caractéristiques de la charge de travail / accès I/O).

Pour les perfs du début ils on :

- fait plusieurs lancement
- exec sur MareNostrum 4 (MN4) supercomputer
- avec différente configuration de forwarding
- avec différent nombre de noeud I/O
- pour voir différents patterns
- utilise un outil FORGE (I/OForwarding Explorer) pour relancer le profiles I/O des app
- il décrive les machines de MN4

Politiques avec les quelle il ce sont comparer :

- \emph{ZERO and ONE Policies}
- \emph{STATIC Policy}
- \emph{SIZE and PROCESS Policies}
- \emph{ORACLE Policy}

Pas besoin de profiling car on regarde juste les politique d'allocation.

Ils executé 189 patterns sur la machine MN4.
Ils on fait des groupe de 16 pour simulé chaque politique.

1 pattern = one app for this test.

Il on généré 10 000 groupe de 16 pattern qui prènne le même temps d'exécution.
Jusque à 128 I/O forwarding nodes. (128 / 16app = 8 by app)
Number of compute nodes : median 256, minimum 88, maximum 512.

La band passante de chaque groupe est calculé par la somme de toutes les (WriteSize + ReadSize / runningTime)
\begin{equation*}aggregate\ BW=\sum_{a=1}^{16}\left(\frac{W_{a}+R_{a}}{runtime_{a}}\right) \tag{2}\end{equation*}

- pas facile de changer la strategies sans avoir un mauvé impacte car statique

Il on en premier regardé que toutes les applications n'on pas les même perf avec le même nombre de I/O nodes. Il on vue que certain patternes on besoin de plus de I/O nodes et d'autre moins. Donc il on montré qu'une allocation statique c'est null.

Pour un ensemble de tâches à exécuter et un nombre fixe de de nœuds I/O il faut déterminé le nombre de noeud I/O forwarding qui maximise l'utilisation de la bande passante globale. Ce probléme dois étre considered as an optimization problem.

Contributions :

1. Le papier évaluons plusieurs politiques d'allocation \emph{I/O forwarding} => démontre que l'allocation dynamique peut amélioré l'utilisation de la bande passante et l'utilisation des noeud I/O.

2. Une politique d'allocation basé sur \emph{Multiple-Choice Knapsack Problem} (\textbf{MCKP}) pour la répartition des noeud I/O aux app.
compute nodes must be divisible by the number of I/O nodes.
on est limité par le nombre total de noeud I/O.

3. Il propose un service \emph{I/O forwarding} appelé \emph{GekkoFWD} qui implémente la politique \textbf{MCKP}. on-demand, au user-level, sans modif du code, simple à déployé, utilise sont propre fs.
Ce fs est enrichi pour perpètre différent déploiement??.

propose : (user-level + portable + don't edit apps => solution utile)

- user-level i/o solution
- on demand
- applique différente politique d'allocation a l'exécution (dynamique)
- changer le nombre de noeud I/O à l'exécution sans perturbé l'app

GekkoFS :

- Un system de fichier local aux noeuds de calculs
- Un \emph{burst-buffer} qui est un buffer qui ce met entre l'app et le FS pour amélioré les perfs. Ce gain de perf est obtenu car il augmente l'utilisation de la band passante (en permettant le stockage de données en même temps que le calcule) et permet d'attenué la charge sur le PFS quant il y à un peaks de demande.
- Fourni un system de nom qui est partagé entre tous les noeuds de calculs. Ce mecanisme peut être remplacé par un PFS (grâce au mode d'extension GekkoFWD).

Grâce à GekkoFWD, GekkoFS est utilisé comme un noeud intermédiaires entre le noeud de calculs et le noeud I/O.

Pour capturé les requêtes de l'app au PFS de façon transparent GekkoFS intércepte les appéles system.

Une fois les requêtes capturé GekkoFWD transfére les requêtes à un seul serveur qui lui vas appliqué la politique d'allocation voulue.

Grâce à GekkoFS ils on donc ajouté un thread qui check si le mappage à changer.

Ce système transparent permet de d'ajouté des optimisation comme la planification des demandes au niveau des fichiers donc il on ajouté la bibliothèque AGIOS dans GekkoFWD.

AGIOS :

- fourni plusieurs ordonanceur.

AGIOS permet à GekkoFWD d'utilisé différent ordonanceur.

Quand un noeud I/O resoive les requêtes il les envoi à AGIOS pour ordonancé le moment ou elle seront traité.

Une fois fini tous est envoyer au PFS.
