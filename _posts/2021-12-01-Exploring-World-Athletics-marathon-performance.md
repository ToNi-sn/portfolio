---
layout: post
title: Exploring World Athletics marathon performance
description: Can we detect potential doping cases in marathon discipline ?
summary: A personal work 
tags: DataCleaning DataVisualization
minute: 5
---

### Abstract

This work will provide some insights about World Athletics data and especially about marathon performance. It will try to answer two majors questions:

• Is it possible to predict a marathoner's best performance ? 

• Is it possible to detect suspicious performances ? (see post about anomaly detection)

To this end, all the steps from data collection to data exploitation will be presented.


### Motivation
As the marathon is becoming more and more popular, especially with Paris 2024 campaign, it could be interesting to learn more about this very specific discipline. As a runner myself (but over shorter distances) I have some experience about road running. I thought that the new abilities learned in data exploration and machine learning I could be able to provide some understanding about top athletes performances.


### Data collection
All of the data has been web-scrapped from World Athletics Website. It has been quite a challenge to collect this data, so I will not enter into details here (see post about webscrapping). In brief, BeautifulSoup module was sufficient enough to do the job. At the end, the dataset is 95 000 lines long and consists in records of marathon performance since 2001. Every record is unique but an athlete can have multiple records. For example, an athlete can have 2 records in 2010, 1 in 2011, 1 in 2013...

### Data Cleaning
After web-scrapping, the data collected were saved to a CSV file, in order not to launch an all new job.
Then the CSV file is read and convert into a dataframe and NA lines are dropped. The data frame then has a total of 71276 lines.
At this step it is important to understand what the columns are and what they refer to:

|                                                              |
|--------------------------------------------------------------|
| • Rank : Rank in the world in year of performance            |
| • Time in HH:mm:ss : Marathon time                           |
| • Full name                                                  |
| • Date of birth                                              |
| • Nationality                                                |
| • Position in the competition : Rank in the race             |
| • Place of the competition : Where the race took place       |
| • Date : When the race took place                            |
| • Result score : Score attribution based on WA scoring table |
| • athlete href : unique ID of athlete                        |
| • Gender                                                     |
| • profile link : https link to athlete profile               |

To facilitate the readability of the data, some columns where modified: Time is converted into seconds and for dates only the year is kept. Also additional columns were created :

|                                                                                                    |
|----------------------------------------------------------------------------------------------------|
| • Best performance                                                                                 |
| • Median performance                                                                               |
| • Mean performance                                                                                 |
| • Number of races : number of marathon run between 2001 and 2021                                   |
| • Period of activity : Time (in years) between first marathon and last marathon                    |
| • Nationality continent : continent where the athlete is coming from (to facilitate visualisation) |

As the number of features was quite small, additional data from athletes were combined with existing ones. Using the profile link column it is possible to get personal best performance (PB) . By web-scrapping again, we get (when existing) a dictionary with the discipline and the performance.

Example : {’5000-metres’: 902, ’3000-metres-steeplechase’: 1001, ’marathon’: 986}

As we are looking for road running performance, only the running discipline will be treated. Meaning that PB on long jump, javelin... will not be taken into account. It is rare for top athletes to be specialized at the same time in long distance running and jumps. However the 1500m PBs up to half-marathon PB’s could be interested. 4 more columns are then added :

|                                                     |
|-----------------------------------------------------|
| • 1500-metres: Personnel Best on on 1500m           |
| • 5000-metres: Personnel Best on on 1500m           |
| • 10000-metres: Personnel Best on on 10000m         |
| • half-marathon: Personnel Best on on half-marathon |


### Some Data Visualization

Plolty library was used to get the figures because of its responsiveness and hover feature. It makes it easier to "play" with graphs.


Evolution of performance            |  Evolution of the number of runners
:-------------------------:|:-------------------------:
![](<img src="https://github.com/ToNi-sn/portfolio/raw/gh-pages/Images/dataframe.png">
)  |  ![](<img src="https://github.com/ToNi-sn/portfolio/raw/gh-pages/Images/dataframe.png">
)




As shown in the two graphs, it appears that the mean performance in marathon decrease when the number of runner increase. In fact, the marathon has became a more accessible discipline to everyone thanks to coaching methods and marketing so not only world class athletes can take part in competition.

After 2010 we see a big drop of performance but in 2018 the performances explose. The reason of the first drop could be the used of prohibited products to enhance running performance. It is now well recognized that EPO was largely used by endurance athletes during this period. In a next part, anomaly detection using clustering will try to confirm this idea. The boom of performances after 2018 can be explained by the use of carbon plated shoes. Many brands have launched on the market shoes that improve running economy. It means less tiredness in muscles so a better final time in marathon.

In the next graphs, each bubble represent an athlete.
Even if the disciplines have been opened to many people, African athletes are still dominating the competition, especially Kenyan and Ethiopian athletes.
The Asian continent have an impressive amount of marathon runners with a lot of density. Quite surprisingly Europe and North America which have the best facilities to train and organise world major marathon are far behind...


![](/Images/overview.png)

Now if the data are reduced to only the top performers (males or females) the graphs completely change. A top performer is considered as someone with the potential to win a major marathon (Paris, New-York...) or to win an olympic medal. For simplification here the a top man performer has already ran under 2h 09min 20s and a top woman performer has already ran under 2h 26min 15s

![](/Images/overview_top.png)

Evolution of top performance |  Evolution of the number of top runners
:-------------------------:|:-------------------------:
![](/Images/top_evolution.png)  |  ![](/Images/count_top_evolution.png)

The boom of performances after 2018 is even more blatant here. The numbers of top performers has been almost multiplied by 3 in just one year. It can be explained by the use of carbon plated shoes. Many brands have launched on the market shoes that improve running economy. It means less tiredness in muscles so a better final time in marathon.

<!---
Evolution of top performance |  Evolution of the number of top runners  |  Evolution of the number of top runners
:-------------------------:|:-------------------------:|:-------------------------:
![](/Images/nike-air-zoom-alphafly-next-eliud-kipchoge.png)  |  ![](/Images/adidas-adizero-prime-x.png) |  ![](/Images/asics-metaspeed-sky.png)

--->

