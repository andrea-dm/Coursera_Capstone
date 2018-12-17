
# Luxury restaurants in Rome

Such a business case is meant for people who are willing to invest in a new luxury restaurant in Rome.


## § Introduction/Business Problem


Rome is a worldwide recognized cultural center, and one can find many luxury restaurants there. An investor might then be interested in funding an already exiting restaurant, or he could even open one by his own.  
It is then important to know which is the optimal neighborhood in Rome where to find/open a luxury restaurant to invest in.

Hence, we are going to tackle the following problem:

> _where to open a new luxury restaurant in Rome?_

We will address such a business case by discussing the following main drivers:
- average income;
- population;
- transportation.

Since we are interested in investing in a luxury restaurant, we first restrict ourselves to just those "Municipi" that stand out for PCI, that is the per-capita income. Within such boroughs, we consider only neighborhoods having a population density higher than a certain predefined treeshold.  

The bunch of "Zone" resulting from the previous wrangling process is then cleaned out of noise via some clustering algorithm, which is applied to a dataframe containing the venues retrieved via the Foursquare API. A filter is subsequently applied to select the more suitable set of neighborhoods.  
Finally, an analysis of the transportation system is performed in order to determine the optimal "Zona": this is the neighborhood that we will eventually suggest to the investor.


## § Data


The data there we are going to exploit in order to provide a solution are the following:
- `Municipi_Roma_15_wgs84_1.shp`: the shapefile containing geographical infos about the boroughs of Rome;
- [Wikipedia](https://it.wikipedia.org/wiki/Municipi_di_Roma): to get boroughs names and their area in km2;
- the [official database](https://www.comune.roma.it/web/it/analisi-statistiche.page) of statistics of the Municipality of Rome;
- `ZU_COD.shp`: the shapefile containing geographical descriptions of the neighborhoods in Rome; 
- [Foursquare](https://www.foursquare.com/): to get a list of venues located within each neighborhood.

In particular, we will make use of the following tables from the official database of the Municipality of Rome:
- `income_by_municipio_2015.xls`, containing the PCI by borough as of December 2015 ([here](https://github.com/andrea-dm/Coursera_Capstone/blob/master/resources/income_by_municipio_2015.xls));
- `Tab_3_ZUrb_T1_17.xlsx`, containing the demographics of all the neighborhoods of Rome as of December 2017 ([here](https://github.com/andrea-dm/Coursera_Capstone/blob/master/resources/Tab_3_ZUrb_T1_17.xlsx)).

The rows of the first table represent the boroughs of Rome, whereas the columns contain the PCI per year divided into three categories: total, Italian, foreign.  We should exploit such data by selecting the "total" column of the year 2015. As for the second one, it is a way more complicated table. It contains info about the number of married/unmarried people, singles, etc etc per each neighborhood and class of age. For the porpouses of this assignment, we will mainly interested in the total number of people living in each neighborhood, no matter of the marital status.

As for Foursquare, we will mainly make use of two GET methods:  
`https://api.foursquare.com/v2/venues/explore` and `https://api.foursquare.com/v2/venues/search`.  
The former will be called for any of the following arguments of the parameter `section`:
- 'food';
- 'drinks';
- 'coffee';
- 'shops';
- 'sights';
- 'arts';
- 'outdoors';

in order to download all the checked-in venues nearby the selected neighborhoods, whereas the latter will be called to get info about the local transportation system. In particular, we are going to ask for venues whose Foursquare category is any of the following:
- 'Bus Stop';
- 'Metro Station';
- 'Tram Station';
- 'Parking'.

The aforementioned data will be then merged into two `pandas` DataFrame, the first one listing geographical\demographic info about each neighborhood and the second one grouping all the retrieved venues from Foursquare. 


## § Methodology


At this point, we are completely aware of the intentions of our client, and the data needed to try a first attempt to takle the problem have been gathered and loaded into the working dataframes. Now is time to start the wrangling process!  
First of all, we select only the boroughs of Rome where the Per-Capita Income (PCI) is above a fixed threshold, which has been previously set to 27.500 EUR per year. The following map shows the "Municipi" by PCI.
![Municipi and PCI](https://github.com/andrea-dm/Coursera_Capstone/blob/master/resources/municipi_and_pci.jpg)  

Next, we consider all the neighborhoods within the boroughs resulting from the previous selection. In particulat, we are restricting ourselves to those "Zone" that are located within the GRA Junction.  
Furthermore, an analysis of population density by neighborhood yields the list of candidate "Zone", which are represented in the following map.

![Zone_and_density](https://github.com/andrea-dm/Coursera_Capstone/blob/master/resources/zone_and_density.jpg)  

The threshold value chosen to filter the neighborhoods out is 1000 inhabitants per km2.  

We are ready to clean out the noise!  
To do so, we run the DBSCAN algorithm with parameters `eps=1`, `min_samples=5`, `metric='cityblock'` on the one-hot encoded dataframe of all the venues retrieved via the Foursquare API.  

![Zone_and_density](https://github.com/andrea-dm/Coursera_Capstone/blob/master/resources/neighborhoods_and_noise.jpg)  

Finally, we inspect the core cluster.  
The analysis is based on the following two considerations:
- Opera Houses, Concert Halls and Theaters are generally attended by somewhat aristocratic/wealthy people;
- it is a generally accepted principle that a new business must be located as close to its competitors as it can be.  

![Zone_and_theaters](https://github.com/andrea-dm/Coursera_Capstone/blob/master/resources/neighborhoods_and_theaters.jpg)  

![Zone_and_restaurants](https://github.com/andrea-dm/Coursera_Capstone/blob/master/resources/zone_and_restaurants.jpg)  

Furthermore, 
