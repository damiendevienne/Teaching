# Visualisation de (très) grands arbres: Lifemap

Le cours est disponible en pdf à cette adresse : ... 

## Objectifs
Ce TP vise à appréhender l'utilisation de Lifemap depuis R, et se familiariser par la même occasion avec l'outil de cartographie leaflet.

* **1. Intro grands arbres : illustration rapide du problème**
* **2. Appréhender Lifemap et les outils de base de Leaflet**
  * Prise en main de leaflet depuis R (charger la carte, mettre un marqueur et un popup associé)
  * Passage à Lifemap (tuiles + données dans Solr)
* **3. Visualiser des données génomiques sur Lifemap**
  * Récupérer les infos de tous les génomes eucaryotes séquencés
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
> - Installer ape 
> - Générer un arbre de N feuilles (N petit puis N grand)
> - Le visualiser
>
> Précéder le nom de la fonction d'un `?` donne accès à l'aide.
```r
install.packages(ape) ## ou require(ape) si déjà installé
tree<-rtree(12)
plot(tree) ### OK
tree2<-rtree(1000)
plot(tree) ### PAS OK
plot(tree, show.tip.label=F, type="u") 
```
Pour aller plus loin dans la visualisation d'arbres avec R : utiliser le package ggtree. Bonne présentation par l'auteur des fonctionnalitées ici : https://guangchuangyu.github.io/presentation/2016-ggtree-chinar/


### 2. Appréhender Lifemap et les outils de base de Leaflet
#### Prise en main de leaflet depuis R (charger la carte, mettre un marqueur et un popup associé)

Les tuiles formant le fond de carte de Lifemap sont accessibles comme celles formant les cartes osm ou google maps.

Afficher ces tuiles, zoomer, paner, dans un navigateur, se fait via des librairies javascript. Une des plus utilisées est [leaflet](https://leafletjs.com/). Suivre le lien pour voir l'étendue des possibilités.

Leaflet est utilisable depuis R  grâce au package `leaflet` (https://rstudio.github.io/leaflet/). Intégration possible avec shiny pour créer des applis plus complètes (voir la fin du TP.)

> **Exo 2** 
> - Installer Leaflet pour R
> - Charger une carte géographique (en mentionnant explicitement la source des tuiles)
> - Mettre un marqueur sur la ville de Lyon (longitude = 45.75, latitude = 4.85) avec un tooltip lorsque la souris passe dessus.
> - Mettre ensuite un marqueur sur la ville de Paris et visualiser à nouveau la carte. Qu'observe-t-on ?
> - Écrire une fonction permettant de 'recharger' une carte vierge (utile pour la suite du TP).


```r
install.packages(leaflet) ## ou require(leaflet) si déjà installé
m<-leaflet()
m<-addTiles(m, url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png")
m<-addMarkers(m, lng=4.85, lat=45.75, label="Ici Lyon")
m  # Print the map
## mettre un marqueur sur Paris
m<-addMarkers(m, lng=2.35, lat=48.85, label="Ici Paris")
## on remarque que les marqueurs s'ajoutent ! 

## Fonction pour charger une carte vierge:
newmap<-function() {
    map<-leaflet()
    map<-addTiles(map, url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png")
    return(map)
}
## Pour créer une nouvelle carte il suffit ensuite de taper : 
m<-newmap()
```
La fonction `leaflet()`peut aussi prendre en entrée des données ( `leaflet(data=...)` ), sous forme de data.frame ou autre. Il est ensuite possible de faire référence à ces données depuis les fonctions ayant l'objet 'carte' renvoyé par `leaflet()` comme premier argument (`addMarkers()`, `addCircles()`, etc.).

> **Exo 3**
> - Créer une data.frame avec les noms "Lyon" et "Paris" dans la première colonne, leurs longitude dans la seconde et leur latitude dans la troisième.
> - Visualiser les marqueurs pour ces deux villes en donnant ce data.frame en entrée à la fonction `leaflet()`
> - Modifier la fonction permettant de recharger une carte nouvelle pour prendre cela en entrée.
```r
dd<-data.frame(cities=c("Paris","Lyon"), longi=c(2.35, 4.85),lati=c(48.85,45.75))
m<-leaflet(dd)
m<-addTiles(m, url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png")
m<-addMarkers(m, lng=~longi,lat=~lati)
m
```


#### Passage à Lifemap (tuiles + données dans Solr)
Les tuiles de Lifemap ont pour url http://lifemap-ncbi.univ-lyon1.fr/osm_tiles/{z}/{x}/{y}.png. 

Les données additionnelles de Lifemap sont stockées dans deux 'coeurs' [Solr](https://lucene.apache.org/solr/). On y trouve les coordonnées de toutes les espèces et les clades (tous les noeuds et les feuilles de l'arbre), ainsi que les noms latins, les synonymes, le nombre de descendants, leurs ascendants, etc. Aller voir directement sur http://lifemap-ncbi.univ-lyon1.fr:8983/solr/#/ pour mieux comprendre. 

Au NCBI, et donc dans Lifemap, les espèces et les clades sont identifiés par un identifiant unique appelé **taxid**. 

Une requête des taxid 2, 9443 et 2087 sur le "coeur" `taxo` se fait par l'url : http://lifemap-ncbi.univ-lyon1.fr:8983/solr/taxo/select?q=taxid:(2%209443%202087)&wt=json&rows=1000. Notons que le `%20` remplace l'espace.

> **Exo 4**
> - Visualiser Lifemap en lieu et place de la carte osm et modifier la fonction pour recharger la carte en conséquence.
> - Écrire une fonction pour récupérer les coordonnées des espèces (à partir d'un vecteur de **taxid**) en utilisant la fonction `fromJSON()` du package `jsonlite` (à installer) <sup>[Aller plus loin](#aller-plus-loin)</sup>
> - Mettre des repères sous formes de ronds avec la fonction `addCircleMarkers()` pour les trois taxids de l'exemple.
> - Jouer avec l'opacité, la présence de bordure, la couleur, la taille, etc. 
> - [rigolo] : n'importe quelle image peut servir d'icône de marqueur. On peut par exemple mettre une [silhouette de cheval](https://svgsilh.com/png-512/156496-cddc39.png) à *Equus caballus* (taxid : 9796). Essayez si vous avez le temps
```r
##nouvelle fonction pour visualiser Lifemap
newmap<-function(df=NULL) {
    map<-leaflet(df)
    map<-addTiles(map, url="http://lifemap-ncbi.univ-lyon1.fr/osm_tiles/{z}/{x}/{y}.png", options = tileOptions(maxZoom = 42))
    return(map)
}
m<-newmap()

## fonction pour requêtes solr : 
install.packages("jsonlite")
require(jsonlite)
GetCooFromTaxID<-function(taxids) {
  ##taxids is an array that contains taxids.
  ## url cannot be too long, so that we need to cut the taxids (100 max in one chunk)
  ## and make as many requests as necessary.
  taxids<-as.character(taxids) #change to characters.
  DATA<-NULL
  i<-1
  while(i<=length(taxids)) {
    cat(".")
    taxids_sub<-taxids[i:(i+99)]
    taxids_sub<-taxids_sub[!is.na(taxids_sub)]
    taxids_sub<-paste(taxids_sub, collapse="%20") #accepted space separator in url
    url<-paste("http://lifemap-ncbi.univ-lyon1.fr:8983/solr/taxo/select?q=taxid:(",taxids_sub,")&wt=json&rows=1000",sep="", collapse="")
    #do the request :
    data_sub<-fromJSON(url)
    DATA<-rbind(DATA,data_sub$response$docs[,c("taxid","lon","lat", "sci_name","zoom","nbdesc")])
    i<-i+99
  } 
  for (j in 1:ncol(DATA)) DATA[,j]<-unlist(DATA[,j])
  class(DATA$taxid)<-"character"
  return(DATA)
}

##test de la fonction
data<-GetCooFromTaxID(c(2,9443,2087))

##Mettre les repères sur la carte sous forme de ronds rouuges
m<-newmap(data)
m<-addCircleMarkers(m, lng=~lon, lat=~lat, stroke = FALSE, fillOpacity = 0.5,color="red", label=~sci_name)
m

### Utiliser une image comme icône à la place du marqueur classique 
equus<-GetCooFromTaxID(9796) #récupérer coordonnées cheval
equusIcon <- makeIcon(
  iconUrl = "https://svgsilh.com/png-512/156496-cddc39.png",
  iconWidth = 51.2, iconHeight = 42,
  iconAnchorX = 25.6, iconAnchorY = 21,
)
m<-newmap(equus)
m<-addMarkers(m, lng=~lon, lat=~lat, icon = equusIcon)
m #voir !
```

### 3. Visualiser des données génomiques sur Lifemap
Le but est de récupérer des données génomiques que nous pouvons associer aux espèces de l'arbre de la vie pour visualiser sur Lifemap. 
Nous nous intéressons ici :
- aux données de séquençage de génome: quantité, représentativité, qualité
- aux données génomiques : nombre de chromosomes, nombre de gènes annotés, taux de GC dans les génomes
- autre (à vous de proposer et trouver des jeux de données)

Les données sont récupérées directement sur le ftp du NCBI avec la fonction `read.table()`. Nous nous intéresserons aux données, eucaryotes, moins volumineuses (moins de génomes séquencés que chez les bactéries). Le fichier à récupérer est disponible à l'url suivante : ftp://ftp.ncbi.nlm.nih.gov/genomes/GENOME_REPORTS/eukaryotes.txt

> **Exo 5**
> - Récupérer les données sur le ftp du NCBI. 
  Attention avec la fonction `read.table()` : les séparateurs de champs sont des tabulations (utiliser `sep="\t"`), il y a un header (utiliser `header=TRUE`), les apostrophes ne doivent pas être considéré comme des guillemets contrairement aux vrais guillemets (`"`) (utiliser `quote="\""`) et la ligne commençant par un `#` ne devrait pas être traitée comme une ligne de commentaires puisque c'est celle qui contient le nom des colonnes (utiliser `comment.char=""`).  
> - compter le nombre de génomes séquencés par espèce (fonction `table()`) et visualiser cette info sous forme de cercles de tailles proportionnelles à cette valeur.
> - 
```r
##récupérer les infos
EukGenomeInfo<-read.table("ftp://ftp.ncbi.nlm.nih.gov/genomes/GENOME_REPORTS/eukaryotes.txt", sep="\t", header=T, quote="\"", comment.char="")
###table du nombre de génomes séquencés
nbGenomeSequenced<-table(EukGenomeInfo$TaxID)
### liste des taxids uniques
taxidsofinterest<-names(nbGenomeSequenced)
### coordonnées 
coo<-GetCooFromTaxID(taxidsofinterest)
coo$nbGsequenced<-as.numeric(nbGenomeSequenced[coo$taxid])
### carte
m<-newmap(coo)
m<-addCircleMarkers(m, lng=~lon, lat=~lat, radius = ~nbGsequenced/10, color = "red", stroke = FALSE, fillOpacity = 0.5, label=~sci_name, popup = as.character(coo$nbGsequenced))
m
```
___
##### Aller plus loin
- Exo 4 : Si vous avez le temps, créez aussi une fonction permettant de récupérer les coordonnées à partir du nom latin et pas du taxid. En tolérant éventuellement les fautes de frappe, etc. (solr permet cela !!)
- 