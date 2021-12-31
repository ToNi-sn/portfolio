---
layout: post
title: WA Anomaly detection
description: Can we detect potential doping cases in marathon discipline ?
summary: A personal work 
tags: UnsupervisedLearning Clustering
minute: 10
---


The aim of this article is to try to highlight some suspicious marathon performances and potential doping of cases. In short, can machine learning helped to detect drug enhancement?
The dataset use here is coming from <a href="https://jonathansoma.com/lede/algorithms-2017/classes/networks/networkx-graphs-from-source-target-dataframe/">World Athletics Website</a> with records on other disciplines than marathon. However the focus is only ont top men performances (below 2h 12min 30s).


```python
import numpy as np
import pandas as pd
import datetime
import pycountry_convert as pc
from tqdm.auto import tqdm
tqdm.pandas()
import json
from collections import Counter
import matplotlib.pyplot as plt
from sklearn import preprocessing
```


```python
df = pd.read_csv('Marathon_V2_w_PB.csv', na_filter=True)
```

```python
df_top = df.loc[df['Gender'] == 'Male']
df_top = df_top.loc[df_top['Time in seconds'] < 7950]  #performance below 2h 12min 30s
df_top.reset_index(drop=True, inplace=True)
```

The method chosen to detect anomalies is the DBSCAN method. It is a method of unsupervised clustering. It takes a minimal distance ε and a minimum number of neighbors. It means that for each marathon performance, it will look if there are any closed (similar) performances based on the features and if yes, it will be added into a cluster. If the conditions are not met for a specific data point, then DBSCAN will identify it as an anomaly. 
Here to choose which ε and number of neighbors are adapted, I decided to have only one "big" cluster to gather the maximum number of performances and to highlight even more the anomalies. So each record can only have two different states. 

But first it is important to prepare and clean the data.

```python

def to_dict(string):
    
    try:
        string_converted = string.replace("'", "\"")
        d_string = json.loads(string_converted)
        return(d_string)
    except ValueError:
        pass

df_top['PB_list']=df_top['PB_list'].apply(lambda x: to_dict(x))
```


```python
#Replacing NA values in PB with median value

df_top = pd.concat([df_top, df_top['PB_list'].apply(pd.Series)[['1500-metres', '5000-metres', '10000-metres', 'half-marathon']]],axis=1)

median_1500 = df_top['1500-metres'].median()
df_top['1500-metres'].fillna(median_1500, inplace = True)

median_5000 = df_top['5000-metres'].median()
df_top['5000-metres'].fillna(median_5000, inplace = True)

median_10000 = df_top['10000-metres'].median()
df_top['10000-metres'].fillna(median_10000, inplace = True)

median_half_marathon = df_top['half-marathon'].median()
df_top['half-marathon'].fillna(median_half_marathon, inplace = True)
```
DBSCAN only work with numerical features and perform better when the data are reduced and centered.

```python
#Encoding

df_top_encoded = pd.get_dummies(df_top,columns=["Gender","Nationality"])
df_name = df_top[['Full name']]
df_top_encoded_reduce = df_top_encoded.drop(['Full name','Place of the competition','Nationality continent','Position in the competion','Result score', 'athlete_href', 'profile_link','Time in seconds','PB_list'], axis =1)
df_centre_reduit = pd.DataFrame(preprocessing.scale(df_top_encoded_reduce, with_mean=True, with_std=True),columns = df_top_encoded_reduce.columns)

```

So the dataframe ends up like this:




Now We can apply DBSCAN Method on it and try to find the optimum number of min samples.
```python
from sklearn.cluster import DBSCAN

min_samples_list = np.arange(0,51,1)
nb_clusters =[]

for k in range(51):
    dbscan_operator = DBSCAN(eps=10, min_samples=k)
    cluster = dbscan_operator.fit_predict(df_centre_reduit)
    nb_clusters.append(len(Counter(cluster)))
```

```python
figure(figsize=(15, 6))

plt.plot(min_samples_list, nb_clusters)
plt.title("Choice of min_samples for DBSCAN")
plt.xlabel("Number of clusters")
plt.ylabel("min_samples");
```

![](/Images/output_8_0.png)

In the light of the assumption made at athe beginning of this article, it appears that optimal value for min samples is 19.

```python
final_dbscan_operator = DBSCAN(eps=10, min_samples=19)
final_clusters = dbscan_operator.fit_predict(df_centre_reduit)
df_centre_reduit['Cluster'] = pd.Series(final_clusters)
```

```python
mapping = {0: "Belong to a cluster", -1:"Anomaly"}
df_centre_reduit = df_centre_reduit.replace({"Cluster": mapping})
df_anomaly_detection = pd.concat([df_name, df_centre_reduit], axis=1)
```
With this parameter, DBSCAN is able to detect 256 anomalies.

It is difficult to say if the algorithm performed well as there is not labelled data if an athlete is doping or not. (if you find suach a dataset do not hesitate to contact me to enrich my study).
However there are still some interesting cases.
For example, the athlete El Hassan EL ABBASSI has been suspended for suspected blood doping. There is also Mo FARAH with a lot of doping suspicions. Finally some anomaly may be explained by the fact that some athletes (especially athletes from east Africa) change nationality for European country.

