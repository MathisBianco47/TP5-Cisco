# TP5-Cisco

vous aurez besoin de :

+ Virtualbox
+ GNS3

les machines virtuelles Linux :

+ l'OS devra être CentOS 7 (en version minimale)
+ pas d'interface graphique (que de la ligne de commande)

les routeurs Cisco :

+ l'iOS devra être celui d'un Cisco 3640

# I. Préparation du lab #

**1.Préparation des VMs**

2. Création des VMs

+ On va juste cloner trois VMs depuis le patron du TP précédent :
+ server1.tp5.b1 est dans net1 et porte l'IP `10.5.1.10/24`
+ client1.tp5.b1 est dans net2 et porte l'IP `10.5.2.10/24`
+ client2.tp5.b1 est dans net2 et porte l'IP `10.5.2.11/24
+ Ajoutez aux trois VMs une interface host-only en deuxième carte dans le host-only précédemment créé
