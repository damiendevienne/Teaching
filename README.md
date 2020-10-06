# Enseignement
Ici se trouvent les documents pour l'enseignement.

# Correction code R

```r

require(jsonlite)
require(leaflet)


###CRÉER UNE CARTE LIFEMAP VIERGE
newmap<-function(df=NULL) {
    map<-leaflet(df)
    map<-addTiles(map, url="http://lifemap-ncbi.univ-lyon1.fr/osm_tiles/{z}/{x}/{y}.png", options = tileOptions(maxZoom = 42))
    return(map)
}


###RÉCUPÉRER LES COORDONNÉES DES ESPÈCES
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
    url<-paste("http://lifemap-ncbi.univ-lyon1.fr/solr/taxo/select?q=taxid:(",taxids_sub,")&wt=json&rows=1000",sep="", collapse="")
    #do the request :
    data_sub<-fromJSON(url)
    DATA<-rbind(DATA,data_sub$response$docs[,c("taxid","lon","lat", "sci_name","zoom","nbdesc")])
    i<-i+100
  } 
  for (j in 1:ncol(DATA)) DATA[,j]<-unlist(DATA[,j])
  class(DATA$taxid)<-"character"
  return(DATA)
}

###RÉCUPÉRER LES DONNÉES
EukGenomeInfo<-read.table("ftp://ftp.ncbi.nlm.nih.gov/genomes/GENOME_REPORTS/eukaryotes.txt", sep="\t", header=T, quote="\"", comment.char="")
## liste unique des taxid
taxids<-unique(EukGenomeInfo$TaxID)

## RÉCUPÉRER LES COORDONNÉES
DF<-GetCooFromTaxID(taxids)

## CALCULER LE NOMBRE DE GÉNOMES SÉQUENCÉS POUR CHAQUE TAXID
nbGenomeSequenced<-table(EukGenomeInfo$TaxID)
## l'ajouter à DF
DF$nbGenomeSequenced<-as.numeric(nbGenomeSequenced[DF$taxid])

##CALCULER LE NB DE GENOMES ENTIEREMENT ASSEMBLES POUR CHAQUE TAXID
##le calcul pour un seul taxid nommé 'tid' serait :
sum(EukGenomeInfo[which(EukGenomeInfo$TaxID=='tid'),]$Status=="Chromosome")
##on peut utiliser la fonction sapply pour le faire pour chaque taxid 
nbGenomeAssembled<-sapply(DF$taxid, function(x,tab) sum(tab[which(tab$TaxID==x),]$Status=="Chromosome"), tab=EukGenomeInfo)
DF$nbGenomeAssembled<-nbGenomeAssembled

##CALCULER LE TAUX de GC MOYEN  
tauxgcmoyen<-sapply(DF$taxid, function(x,tab) mean(as.numeric(as.character(tab[which(tab$TaxID==x),]$GC.)), na.rm=TRUE), tab=EukGenomeInfo)
DF$tauxgcmoyen<-tauxgcmoyen #il y a des warnings car pour certaines espèces le taux de GC n'est pas estimé.

##CALCULER LA TAILLE MOYENNE DES GÉNOMES EN Mb
SizeGenomeMb<-sapply(DF$taxid, function(x,tab) mean(as.numeric(as.character(tab[which(tab$TaxID==x),]$Size..Mb.)), na.rm=TRUE), tab=EukGenomeInfo)
DF$SizeGenomeMb<-SizeGenomeMb 

### REGARDER OÙ SONT TOUS CES GÉNOMES SUR LA CARTE
m<-newmap(DF)
m<-addCircleMarkers(m, lng=~lon, lat=~lat, radius = 10, color = "red", stroke = TRUE, fillOpacity = 0.5, label=~sci_name)
m


```
