
# Luxury restaurants in Rome

Such a business case is meant for people who are willing to invest in a new luxury restaurant in Rome.

## § Introduction/Business Problem

The problem we are going to tackle is the following:

> _where to open a new luxury restaurant in Rome?_

We will address such a business case by discussing the following main drivers:
- average income;
- population;
- transportation.

Since we are interested in investing in a luxury restaurant, we first restrict ourselves to just those "Municipi" that stand out for PCI, that is the per-capita income. Within such boroughs, we consider only neighborhoods having a population density higher than a certain predefined treeshold.  
The resulting "Zone" are then collected into clusters via some Machine Learning algorithm. Such clusters are built on top of a large dataframe containing the venues retrieved via the Foursquare API for each neighborhood. A filter is subsequently applied to select the more suitable cluster.  
Finally, an analysis of the transportation system is performed in order to determine the _best_ "Zona": this is the neighborhood that we will eventually propose to the investor.


## § Data


The data there we are going to exploit in order to provide a solution are the following:
- `Municipi_Roma_15_wgs84_1.shp`: the shapefile containing geographical infos about the boroughs of Rome;
- [Wikipedia](https://it.wikipedia.org/wiki/Municipi_di_Roma): to get boroughs names and their area in km2;
- the [official database](https://www.comune.roma.it/web/it/analisi-statistiche.page) of statistics of the Municipality of Rome;
- `ZU_COD.shp`: the shapefile containing geographical descriptions of the neighborhoods in Rome; 
- [Foursquare](https://www.foursquare.com/): to get a list of venues located within each neighborhood.

In particular, we will make use of the following tables from the official database of the Municipality of Rome:
- `income_by_municipio_2015.xls`, containing the PCI by borough;
- `Tab_3_ZUrb_T1_17.xlsx`, containing the demographic dataset of all the neighborhoods of Rome.

As for Foursquare, we will mainly make use of two GET methods:  
`https://api.foursquare.com/v2/venues/explore` and `https://api.foursquare.com/v2/venues/search`.  
The former will be called for any of the following arguments of the parameter `section`:
- 'food';
- 'drinks';
- 'coffee';
- 'sights';
- 'arts';
- 'outdoors';

in order to download all the checked-in venues nearby the selected neighborhoods, whereas the latter will be called to get info about the local transportation system. In particular, we are going to ask for venues whose Foursquare category is any of the following:
- 'Bus Stop';
- 'Metro Station';
- 'Tram Station';
- 'Parking'.

The aforementioned data will be then merged into two `pandas` DataFrame, the first one listing geographical\demographic info about each neighborhood and the second one grouping all the retrieved venues from Foursquare. 
