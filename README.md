# Cloud-TP
# Duval Thomas B3 C


                        
Le but de ce projet est déployé une stack aplicative dans un type de Cloud moderne.
Nous allons voir :
                        - Deploiement Applicatif
                        - Accessibilité 
                        - Automatisation
                        - Robutesse

# Prérequis 

Dans ce TP il faut :
                        - Un cable RJ45 (Pour relier les deux laptops)
                        - Deux Ordninateurs
                                  - 1 ère Machine 192.168.10.40/24
                                  - 2 ème Machine 192.168.10.50/24
                                  
Il faut aussi récuperer le lien GIT de coreos afin de commencer le TP
Lien : https://github.com/coreos/coreos-vagrant
On fait un git clone pour recuperer les fichiers.

On obtiens un Vagrantfile on l'explore "vim vagrantfile"
Avant de commencer il faut ajouter un disk, lancer 5 machines puis utilisé une interface bridgé.

Pour changer le nombre de machine a lancé on va (# Defaults for config) et on chanfe le nombre $num_instances = 5

Pour l'ajout d'un disque : config.disksize.size = "10G".

Pour le bridge : ip = "192.168.10.#{i+100}"
                 config.vm.network :public_network, :bridge => "enp0s2 , ip : ip
Ce qu'on vient de faire c'est que tous les IP dans le réseau 10 sont bridgés
Et bien entendu our que ca le fasse on référence bien son interface.

# Docker Swarm

Docker est déjà installé par défault dans les coreos. 
Nous allons mettre en place un swarm de 5 noeuds : 3 managers et 2 workers. Le cluster restera sain tant que 2 managers seront en vie.

Avant de commencer il faut configurer le Experimental ensuite on ajoutera la clause de metrics-addr.
Aussi il faut se connecter en SSH : 
                                    -Machine 1 : ssh mach1@192.168.10.50 | 51 | 52 | 53 | 54

Pour l'activer on va dans le fichier docker deamon.json et on ajoute : "experimental": true,
    "metrics-addr": "0.0.0.0:9323" dans ce fichier.
    
Ajouter un manager on fait un:  docker swarm init --advertise-addr 192.168.10.50
Ensuite joindre des managers docker swarm join-token manager sur la 51 & 52.
Pour les wokers : docker swarm join-token worker sur la 53 puis 54.

# Pourquoi deux ordinateurs ?


On travaille avec 2 machines reliées via le réseau afin d'alléger la charge supportée par chacune de vos hôtes.
Sauf qu'on monte des technos de cluster dans tous les sens. Donc si vous débranchez le câble, tout va probablement exploser. Ce qu'on va faire c'est répartir intelligemment les services sur vos deux machines.

# Weave Cloud

Dans le lien ca explique tout et aussi vous pouvez vous le montez (Assez puissant).
Lien : https://github.com/helm/charts/tree/master/stable/weave-cloud

# CEPH

Est un outil permettant de mettre en place (notamment) un système de fichiers distribué ; plutôt que d'avoir une partition locale utilisé sur un filesystem local, nous allons mettre en commun des partitions (en réalité, des blocs) à travers le réseau, et rendre le tout accessible sur tous nos hôtes comme une simple partition.

Pas compris comment le mettre en place idée avec un tar et on le dispache sur toute les coreos...

# NFS

Une machine virtuel centos qu'on crée.
On l'a met sur le meme réseau 
IP : 192.168.10.60/24
On installe le NFS sur le V-M : yum install -y nfs-utils
On l'active on le lance comme docker :
                                      - $ systemctl enable nfs-server.service
                                      - $ systemctl start nfs-server.service

On crée le dossier qu'on veut partager : mkdir /var/nfs-partage.
On lui change de proprio : $chown nfsnobody:nfsnobody /var/nfs-partage.
Pour les droits : $chmod 755 /var/nfs-partage

Pour définir les partages sur le réseau : vim /etc/exports & on ajoute : /var/nfs-partage        192.168.10.*(rw,sync,no_subtree_check)

On enregistre et on redemarre les services suite à la modification.

- systemctl restart nfs-server

Bien connu our eviter ls bugs ou souci de lancement ou d'afficage 
- firewall-cmd --permanent --zone=public --add-service=nfs
- firewall-cmd --reload

On intègre le firewall.

Ensuite pour l'appliquer sur les autres machines coreos.
mkdir -p /home/nfs-partgage ensuite on monte le dossier : mount 192.168.10.100:/var/nfs-partage /home/nfs-partgage .

# Registry
Pour gagner du temps on monte un registre. 
Pour créer un registre : docker service create --name registry --publish published=5000,target=5000 registry:2

Pour voir si ça fonctionne : curl 127.0.0.1:5000. 
On doit avoir {}. 
Sur la coreos le retour à été correctement fait.

Savoir où tourne le registre : docker service ps registry









                        
                        
