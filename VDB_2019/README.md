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
> - Le visualiser. Observer le problème.
```r
install.packages(ape) ## ou require(ape) si déjà installé
tree<-rtree(12)
plot(tree) ### OK
tree2<-rtree(1000)
plot(tree) ### PAS OK
plot(tree, show.tip.label=F, type="u") 
```
Pour aller plus loin dans la visualisation d'arbres avec R : utiliser le package ggtree. Bonne présentation par l'auteur des fonctionnalitées ici : https://guangchuangyu.github.io/presentation/2016-ggtree-chinar/. 


### 2. Appréhender Lifemap et les outils de base de Leaflet
#### Prise en main de leaflet depuis R (charger la carte, mettre un marqueur et un popup associé)

Les tuiles formant le fond de carte de Lifemap sont accessibles comme celles formant les cartes osm ou google maps.

Afficher ces tuiles, zoomer, paner, dans un navigateur, se fait via des librairies javascript. Une des plus utilisées est [leaflet](https://leafletjs.com/). Suivre le lien pour voir l'étendue des possibilités.

Leaflet est utilisable depuis R  grâce au package `leaflet` (https://rstudio.github.io/leaflet/). Intégration possible avec shiny pour créer des applis plus complètes (voir la fin du TP).
Le code ci-dessous permet de charger un fond de carte osm avec leaflet, et d'ajouter un marqueur au niveau de la ville de Lyon. 
```r
require("leaflet")
m<-leaflet()
m<-addTiles(m)
m<-addMarkers(m, lng=4.85, lat=45.75, label="Ici Lyon")
```

> **Exo 2** 
> - Installer Leaflet pour R
> - Exécuter le code précédent, mais en mentionnant explicitement la source des tuiles : https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png
> - Ajouter (en plus de lyon) un marqueur sur la ville de Paris (longitude = 48.85, latitude = 2.35) et visualiser à nouveau la carte. Qu'observe-t-on ?
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
## Pour voir la carte : 
m 
```
La fonction `leaflet()`peut aussi prendre en entrée des données avec `leaflet(data=...)`, sous forme de data.frame ou autre. Il est ensuite possible de faire référence aux éléments présents dans le data frame depuis les fonctions `addMarkers()`, `addCircles()`, `addCircleMarkers()`, etc. en ne mentionnant que le nom de la colonne d'intérêt précédé du tilde (`~lat` pour latitude par exemple). 

> **Exo 3**
> - Créer une data.frame avec les noms "Lyon" et "Paris" dans la première colonne, leurs longitude dans la seconde et leur latitude dans la troisième.
> - Visualiser les marqueurs pour ces deux villes en donnant ce data.frame en entrée à la fonction `leaflet()`
> - Modifier la fonction permettant de recharger une carte nouvelle pour permettre de prendre un data frame en entrée.
```r
dd<-data.frame(cities=c("Paris","Lyon"), longi=c(2.35, 4.85),lati=c(48.85,45.75))
m<-leaflet(dd)
m<-addTiles(m, url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png")
m<-addMarkers(m, lng=~longi,lat=~lati)
m

##fonction mise à jour :
newmap<-function(data=NULL) {
    map<-leaflet(data)
    map<-addTiles(map, url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png")
    return(map)
}
```

#### Passage à Lifemap (tuiles + données dans Solr)
Les tuiles de Lifemap ont pour url http://lifemap-ncbi.univ-lyon1.fr/osm_tiles/{z}/{x}/{y}.png. 

Les données additionnelles de Lifemap sont stockées dans deux 'coeurs' [Solr](https://lucene.apache.org/solr/). On y trouve les coordonnées de toutes les espèces et de tous les clades (un clade = un groupe monophylétique), ainsi que les noms latins, les synonymes, le nombre de descendants, leurs ascendants, etc. Aller voir directement sur http://lifemap-ncbi.univ-lyon1.fr:8983/solr/#/ pour mieux comprendre. 

Au NCBI, et donc dans Lifemap, les espèces et les clades sont identifiés par un identifiant unique appelé **taxid**. 

Une requête des taxid 2, 9443 et 2087 sur le "coeur" `taxo` (qui contient les données taxonomiques) se fait par l'url : http://lifemap-ncbi.univ-lyon1.fr:8983/solr/taxo/select?q=taxid:(2%209443%202087)&wt=json&rows=1000. Notons que le `%20` remplace l'espace.

> **Exo 4**
> - Visualiser Lifemap en lieu et place de la carte osm précédente et modifier la fonction qui crée une nouvelle carte vierge en conséquence.
> - Écrire une fonction pour récupérer les coordonnées des espèces (à partir d'un vecteur de **taxid**) en utilisant la fonction `fromJSON()` du package `jsonlite` (à installer) <sup>[Aller plus loin](#aller-plus-loin)</sup>
> - Récupérer les coordonnées des trois taxids de l'exemple et les représenter dans Lifemap sous forme forme de ronds (fonction `addCircleMarkers()`).
> - Jouer avec l'opacité, la présence de bordure, la couleur, la taille, etc. Taper `?addCircleMarker()` dans R pour avoir de l'aide.
> - [rigolo] : n'importe quelle image peut servir d'icône de marqueur. On peut par exemple mettre une [silhouette de cheval](https://svgsilh.com/png-512/156496-cddc39.png) à *Equus caballus* (taxid : 9796). Essayez si vous avez le temps (explication détaillée [ici](https://rstudio.github.io/leaflet/markers.html)).
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
    i<-i+100
  } 
  for (j in 1:ncol(DATA)) DATA[,j]<-unlist(DATA[,j])
  class(DATA$taxid)<-"character"
  return(DATA)
}

##test de la fonction
data<-GetCooFromTaxID(c(2,9443,2087))

##Mettre les repères sur la carte sous forme de ronds rouuges
m<-newmap(data)
m<-addCircleMarkers(m, lng=~lon, lat=~lat, stroke = FALSE, 
fillOpacity = 0.5,color="red", label=~sci_name)
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
Le but est de récupérer des données génomiques que nous pouvons associer aux espèces de l'arbre de la vie et visualiser sur Lifemap. 
Nous nous intéressons ici :
- aux données de séquençage de génomes: quantité, représentativité, qualité
- aux données génomiques à proprement parler: nombre de chromosomes, nombre de gènes annotés, taux de GC dans les génomes

Les données seront récupérées directement sur le ftp du NCBI avec la fonction `read.table()`. Nous nous intéresserons aux données, eucaryotes, moins volumineuses (moins de génomes séquencés que chez les bactéries). Le fichier à récupérer est disponible à l'url suivante : ftp://ftp.ncbi.nlm.nih.gov/genomes/GENOME_REPORTS/eukaryotes.txt. L'explication des champs présents dans ce fichier est la suivante :

|Column|Description|
|---|---|
| #Organism/Name | Organism name at the species level   |
| BioProject | BioProject Accession number (from BioProject database) |
|Group |         Commonly used organism groups:  Animals, Fungi, Plants, Protists;|
|SubGroup|       NCBI Taxonomy level below group: Mammals, Birds, Fishes, Flatworms, Insects, Amphibians, Reptiles, Roundworms, Ascomycetes, Basidiomycetes, Land Plants, Green Algae, Apicomplexans, Kinetoplasts;|
|Size (Mb)     | Total length of DNA submitted for the project |
|GC%     |       Percent of nitrogenous bases (guanine or cytosine) in DNA submitted for the project|
|Assembly      | Name of the genome assembly (from NCBI Assembly database)|
|Chromosomes  |  Number of chromosomes submitted for the project       |
|Organelles  |   Number of organelles submitted for the project |
|Plasmids     |  Number of plasmids submitted for the project |
|WGS          |  Four-letter Accession prefix followed by version as defined in WGS division of GenBank/INSDC|
|Scaffolds |     Number of scaffolds in the assembly|
|Genes      |    Number of Genes annotated in the assembly|
|Proteins    |   Number of Proteins annotated in the assembly | 
|Release Date |  First public sequence release for the project|
|Modify Date  |  Sequence modification date for the project|
|Status   |      Highest level of assembly: <br> Chromosomes: one or more chromosomes are assembled<br> Scaffolds or contigs: sequence assembled but no chromosomes <br>SRA or Traces: raw sequence data available<br> No data: no data is connected to the BioProject ID

*Attention: le nom des colonnes change après l'import dans R !*

> **Exo 5**
> - Récupérer les données sur le ftp du NCBI. 
  Attention avec la fonction `read.table()` : les séparateurs de champs sont des tabulations (utiliser `sep="\t"`), il y a un header (utiliser `header=TRUE`), les apostrophes ne doivent pas être considéré comme des guillemets contrairement aux vrais guillemets (`"`) (utiliser `quote="\""`) et la ligne commençant par un `#` ne devrait pas être traitée comme une ligne de commentaires puisque c'est celle qui contient le nom des colonnes (utiliser `comment.char=""`).  Note : il est possible d'utiliser l'url du fichier directement dans la fonction `read.table()` sans avoir à télécharger le fichier sur le disque préalablement.
> - compter le nombre total de génomes séquencés par espèce (fonction `table()`)
> - récupérez les coordonnées des taxids ayant au moins un génome séquencé.
> - visualiser le nombre de génomes séquencés par espèce sous forme de cercles de taille proportionnelle à cette valeur.
> - visualiser le nombre de génomes séquencés par espèce sous forme de cercles de même diamètre mais avec un dégradé de couleurs. 
> - compter puis visualiser ensuite le **nombre** de génomes entièrement assemblés (`Status == "Chromosome"`)
> - Visualiser la **proportion** de génomes entièrement assemblés par espèce.  
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

### 4. Aller plus loin dans l'interactivité avec shiny 
Shiny vous a été présenté dans une séance antérieure. Un tutoriel peut aussi être trouvé ici : https://shiny.rstudio.com/tutorial/

Le code ci-dessous (modifié depuis https://rstudio.github.io/leaflet/shiny.html) permet de créer une application web pour visualiser les données de 1000 éruptions volcaniques survenues dans les îles Fidgi depuis 1964, avec leur localisation, leur intensité, leur profondeur. Différents types de boutons permettent de filtrer ce qui est affiché sur la carte. 
Notez l'utilisation de la fonction `leafletProxy()` qui remplace la fonction `leaflet()` pour permettre de ne pas avoir à recharger l'ensemble de la carte quand seulement des filtres ou de nouveaux styles sont appliqués à quelques layers. 

Notez l'utilisation de `reactive` et `observe`. 
La différence entre les deux est ténue mais on peut dire que : 
- la fonction `reactive()` surveille si des données en entrée changent, et renvoie une valeur utilisée à l'extérieur. 
- la fonction `observe()` ne renvoie pas de données en dehors. Elle surveille simplement si les données à l'intérieur changent. 

```r
library(shiny)
library(leaflet)
library(viridis)
library(RColorBrewer)

ui <- bootstrapPage(
  tags$style(type = "text/css", "html, body {width:100%;height:100%}"),
  leafletOutput("map", width = "100%", height = "100%"),
  absolutePanel(top = 10, right = 10,
    sliderInput("range", "Magnitudes", min(quakes$mag), max(quakes$mag),
      value = range(quakes$mag), step = 0.1
      ),
    selectInput("depth", "Depth of earthquakes (km)",c("all","<200","200-500",">500"))
    )
  )

server <- function(input, output, session) {
##set palette function
  pal<-colorNumeric("magma", quakes$mag)
  # Reactive expression for the data subsetted to what the user selected
  filteredData <- reactive({
    k<-quakes[quakes$mag >= input$range[1] & quakes$mag <= input$range[2],]
    if (input$depth!='all') {
      if (input$depth=="<200") k<-k[k$depth<200,]
      if (input$depth=="200-500") k<-k[k$depth>=200&k$depth<=500,]
      if (input$depth==">500") k<-k[k$depth>500,]
    }
    k
  })

  output$map <- renderLeaflet({
  # Use leaflet() here, and only include aspects of the map that
    # won't need to change dynamically (at least, not unless the
    # entire map is being torn down and recreated).
    m<-leaflet(quakes)
    m<-addTiles(m)
    m<-fitBounds(m, ~min(long), ~min(lat), ~max(long), ~max(lat))
    m<-addLegend(m, position = "bottomright", pal = pal, values = ~mag)
  })
  # Incremental changes to the map (in this case, replacing the
  # circles when a new color is chosen) should be performed in
  # an observer. Each independent set of things that can change
  # should be managed in its own observer.
  observe({
    proxy<-leafletProxy("map", data = filteredData())
    proxy<-clearShapes(proxy)
    proxy<-addCircles(proxy, radius = ~10^mag/10, weight = 1, color = "#777777", fillColor = ~pal(mag), fillOpacity = 0.7, popup = ~paste(mag))
  })
}

shinyApp(ui, server)
```

> **Exo 6**
> - En vous inspirant de ce code, et en vous reposant sur ce que vous avez vu dans les exercices précédents, créez une application Shiny fonctionnelle permettant de visualiser des données biologiques (celles vues dans les exercices précédents) sur Lifemap. 
> Attention : Ne pas oublier de mettre des légendes.

___
##### Aller plus loin
- Exo 4 : Si vous avez le temps, créez aussi une fonction permettant de récupérer les coordonnées à partir du nom latin et pas du taxid. En tolérant éventuellement les fautes de frappe, etc. (solr permet cela !!)
- 