# Visualisation de (très) grands arbres: Lifemap

Le cours est disponible en pdf à cette adresse : ... 

## Objectifs
Ce TP vise à appréhender l'utilisation de Lifemap depuis R. Il comporte les étapes suivantes, et se familiariser par la même occasion avec les outils de cartographie depuis R.

* **1. Intro grands arbres : illustration rapide du problème**
* **2. Appréhender Lifemap et les outils de base de Leaflet**
  * Localiser les adresses des serveurs Lifemap (tuiles + données dans Solr)
  * Construire des requêtes pour récupérer les coordonnées des espèces et des clades d'intérêt (créer fonction)
  * Ajouter un marqueur avec/sans popup + transformer le marqueur en icône personnalisée
* **3. Visualiser des données génomiques sur Lifemap**
  * récupérer les infos de tous les génomes eucaryotes séquencés
  * Visualiser dans Lifemap le nombre total de génomes séquencés pour chaque espèce
  * Visualiser dans Lifemap le nombre de génomes séquencés et assemblés entièrement pour chaque espèce
  * Visualiser le taux de GC de chaque espèces séquencée selon deux modalités (couleur et taille de cercle).
* **4. Aller plus loin dans l'interactivité avec shiny**
  * Avec Shiny, créer une version de Lifemap permettant de choisir la donnée à visualiser
  * Créer une barre de recherche

## Jeux de données 
* Serveur de tuiles Lifemap
* Données 


### 1. Intro grands arbres : illustration rapide du problème
En R, avec le package ape, il est simple de générer de arbres avec la fonction `rtree` et les visualiser rapidement avec la fonction `plot.phylo` ou `plot` .

> **Exo 1** 
> Installer ape 
> Générer un arbre de N feuilles (N petit puis N grand)
> Le visualiser
> + `?` Précéder le nom de la fonction d'un point d'interrogation donne accès à l'aide.


```r
install.packages(ape) ## ou require(ape) si déjà installé
tree<-rtree(12)
plot(tree) ### OK
tree2<-rtree(1000)
plot(tree) ### PAS OK
plot(tree, show.tip.label=F, type="u") 
```
Pour aller plus loin dans la visualisatio d'arbres avec R : utiliser ggtree. Bonne présentation par l'auteur des fonctionalitées https://guangchuangyu.github.io/presentation/2016-ggtree-chinar/


### 2. Appréhender Lifemap et les outils de base de Leaflet
Lifemap est construit et accessible comme une carte géographique. Visualiser des cartes géographiques et y ajouter des layers (couches) de données peut se faire avec [leaflet](https://leafletjs.com/) (javascript). Une des possibilités est donc de coder directement en html et javascript -> lourd. voir le code source  de http://lifemap-ncbi.univ-lyon1.fr/ .

Leaflet existe pour R ([ici](https://rstudio.github.io/leaflet/)). Il permet d'utiliser les fonctionnalité de Leaflet directement depuis R (et Shiny pour aller plus loin).

> **Exo 2** 
> Installer Leaflet pour R
> Charger une carte géographique (en mentionnant explicitement la source des tuiles)
> Trouver sur *Lifemap-ncbi* l'adresse des tuiles et l'utiliser à la place des tuiles osm.
> Se promener dans Lifemap ainsi.


```r
install.packages(leaflet) ## ou require(leaflet) si déjà installé
m<-leaflet()
m<-addTiles("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png")
m #pour voir la carte
#####
#
```
