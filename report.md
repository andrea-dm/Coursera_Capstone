
# Luxury restaurants in Rome

Such a business case is meant for people who are willing to invest in a new luxury restaurant in Rome.  
We recall that "Municipio" is the Italian for borough, and "Zona" is the Italian for "neighborhood".


## § Introduction/Business Problem


Rome is a worldwide recognized cultural center, and one can find many luxury restaurants there. An investor might then be interested in funding an already exiting restaurant, or he could even think to open a new one on his own.  
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
- `Municipi_Roma_15_wgs84_1.shp`: the shapefile containing geographical info about the boroughs of Rome;
- _Wikipedia_: to get boroughs names and their area in km2;
- the official database _"Analisi e dati statistici"_ of the Municipality of Rome;
- `ZU_COD.shp`: the shapefile containing geographical descriptions of the neighborhoods in Rome; 
- _Foursquare_: to get a list of venues located within each neighborhood.

In particular, we will make use of the following tables from the official database of the Municipality of Rome:
- `income_by_municipio_2015.xls`, containing the PCI by borough as of December 2015 ([here](https://github.com/andrea-dm/Coursera_Capstone/blob/master/resources/income_by_municipio_2015.xls));
- `Tab_3_ZUrb_T1_17.xlsx`, containing the demographics of all the neighborhoods of Rome as of December 2017 ([here](https://github.com/andrea-dm/Coursera_Capstone/blob/master/resources/Tab_3_ZUrb_T1_17.xlsx)).

The rows of the first table represent the boroughs of Rome, whereas the columns contain the PCI per year divided into three categories: total, Italian, foreign.  We should exploit such data by selecting the "total" column of the year 2015. As for the second one, it is a way more complicated table. It contains info about the number of married/unmarried people, singles, etc etc per each neighborhood and class of age. For the purposes of this assignment, we will mainly interested in the total number of people living in each neighborhood, no matter of the marital status.

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
At this point, we are completely aware of the intentions of our client, and the data needed to try a first attempt to tackle the problem have been gathered and loaded into the working dataframes.  


## § Methodology


The boroughs of Rome that have preliminarily selected were the ones whose Per-Capita Income (PCI) was above a fixed threshold, which has been set to 27,500€ per year. The rationale behind such a value is based on the following computation: supposing that a customer is willing to pay around 100.00€ for a meal in a high level restaurant, and assuming that such an amount is less than the 5% of his salary, then the target should earn more than 2,000€ per month, 13 months per year, yielding an annual income of 26,000€ on average. An extra of 1,500€ has been added to enlarge the confidence interval.  
Once the boroughs excluded by the previous filter have been dropped from the dataframe, the "Zone" within the selected "Municipi" have been taken under consideration. In particular, the list of neighborhoods has been shortened to just those "Zone" that are located within the GRA Junction, the reason being the better connection.  
Finally, an analysis of population density by neighborhood has yielded the list of candidate neighborhoods. The threshold value chosen to filter the neighborhoods out has been set to 1,000 inhabitants per km2. The scope was to target as many people as possible.  
A this stage, a clustering algorithm has been considered in order to get rid of uncorrelated neighborhoods. Such a relation was based on both the number and the category of the venues located in each "Zona". Since the goal was to clean data out of noise, the DBSCAN algorithm with parameters `eps=1`, `min_samples=5`, `metric='cityblock'` has been applied to the one-hot encoded dataframe of all such venues, that have been previously retrieved via the Foursquare API. The choice of the DBSCAN algorithm relies on its ability to detect efficiently outliers.  
The cleaned dataset has been further filtered out by excluding the neighborhoods where at no Opera Houses, Concert Halls and Theaters are located. In fact, such venues are generally attended by somewhat aristocratic/wealthy people, who are precisely the primary target.  
Next, the number of restaurants is counted for each neighborhood, non-Italian cuisine ignored.  
Finally, data about bus stops, metro stations, tram stations and parking have been retrieved via the Foursquare API, and the only the "Zone" where such infrastructures are located have been selected. Moreover, a weight has been assigned to each category of those. Based upon the impact on the transportation system that traffic jams usually have in Rome, the weight of metro and tram stations has been chosen to be heavier than both bus stops and parking. Further characteristics have been taken into consideration in the weights assignment such as the speed of the vehicles.  


## § Results


The following map shows the "Municipi" by PCI:  

![Municipi and PCI](https://github.com/andrea-dm/Coursera_Capstone/blob/master/resources/municipi_and_pci.jpg)  


The darker the purple, the higher the average income. The below map shows instead the population density per "Zona".  

![Zone_and_density](https://github.com/andrea-dm/Coursera_Capstone/blob/master/resources/zone_and_density.jpg)  

The results of the DBSCAN algorithm have been plotted on the following map of Rome  

![Zone_and_density](https://github.com/andrea-dm/Coursera_Capstone/blob/master/resources/neighborhoods_and_noise.jpg)  

where the yellow points represent the outliers.  
Considering venues such as Opera Houses, Concert Halls and Theaters has yielded the following chart, where the size of each bubble represents the area in km2 of the corresponding neighborhood:  

![Zone_and_theaters](https://github.com/andrea-dm/Coursera_Capstone/blob/master/resources/neighborhoods_and_theaters.jpg) 

Then Restaurants in each "Zona" have been counted, and their density computed. The interquartile range has been also calculated in order to select the optimal neighborhood.  

![Zone_and_restaurants](https://github.com/andrea-dm/Coursera_Capstone/blob/master/resources/zone_and_restaurants.jpg)   

As a matter of fact, it is a generally accepted principle that a new business must be located as close to its competitors as it can be, in order to leverage the marketing of the other businesses and increase competitiveness (check the article by Karen Spaeder in the "References" section). Hence, it is crucial to pick a neighborhoods where it is more likely to count the expected number of restaurants.  

Finally, the weighted average of distances of TPL from the centroids of the neighborhoods is computed.  
The chart below describes the relevant data about the transportation system. In particular, the size of each bubble represents the number of bus stops, metro/tram stations and parkings which are located nearby the corresponding neighborhood.   

![Zone_and_tpl](https://github.com/andrea-dm/Coursera_Capstone/blob/master/resources/zone_and_tpl.jpg)  

Putting everything together, we get the following table:

![Candidate Neighborhoods](https://github.com/andrea-dm/Coursera_Capstone/blob/master/resources/candidate_neighborhoods.jpg)  


## § Discussion


In order to infer the optimal neighborhood, only the columns `Restaurants per km2`, `TPL per km2` and `Average Distance` of the above dataframe have been examined. In particular, they have been cut in quantiles.  

![Quartiles](https://github.com/andrea-dm/Coursera_Capstone/blob/master/resources/zone_and_quartiles.jpg)  

The following guidelines will yield the optimal choice:
- the density of restaurants per km2 should be in the interquartile range;
- the density of TPL per km2 should be above the median;
- the average distance from TPL should be as short as possible.

If the first of the above guidelines addresses the question of being as close to the competitors as possible, the second of them hints at the candidate neighborhood as a node of the transportation network, based on the assumption that the more connected it is, the better. The last guideline is meant to locate the new restaurant next to any TPL.  

Finally, a couple of remarks.  
The analysis that we have conducted is very far from being complete and exhaustive. For instance, 4/5 stars hotels could be considered in the study as well, for the reason that such venues come along with high level restaurants most of the times. Unfortunately, the limitations of the Foursquare API had the negative consequence of restricting the number of venues that we could fetch.  
Moreover, the information about population and income are not updated, hence they are susceptible to changes that might affect dramatically the results. As an example, data about PCI are dated 2015.  
One can clearly overcome such obstacles by appealing to other API and by looking at other economic/financial data. We defer similar investigations for they would go beyond the scope of such analysis.  


## § Conclusion

The result of the above analysis is clear: the optimal neighborhood is "Zona Prati".  
Even if the data we got have been proved to be somewhat limited, the methods and the techniques applied mathematical-based, hence solid. This makes the result valid and reliable.


## § References


- *Municipality of Rome*, **Sistema Informativo Geografico**. URL: [websit.cittametropolitanaroma.gov.it](http://websit.cittametropolitanaroma.gov.it/Download.aspx).
- *Wikipedia*, **Municipi di Roma**. URL: [it.wikipedia.org/wiki/Municipi_di_Roma](https://it.wikipedia.org/wiki/Municipi_di_Roma).
- *Municipality of Rome*, **Analisi e dati statistici**. From the database "Dati e statistiche". URL: [www.comune.roma.it/web/it/analisi-statistiche.page](https://www.comune.roma.it/web/it/analisi-statistiche.page).
- *Foursquare*, **Documentation**. URL: [developer.foursquare.com/docs](https://developer.foursquare.com/docs).
- *Karen E. Spaeder*, **How to Find the Best Location**. In: Entrepreneur Europe. URL: [www.entrepreneur.com/article/73784](https://www.entrepreneur.com/article/73784).
