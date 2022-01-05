---
layout: post
title: Network Graph on Marathon popularity around the world
description: Simple overview about the marathon famous places
summary: A personal work 
tags: DataCleaning DataVisualization
minute: 5
---

### Abstract

This work will provide some insights about Marathon famous places around the world and how they are linked with the nationality of their participants. NetworkX library was then used to perform graph visualization. Part of the code is largely inspired by Jonathan Soma <a href="https://jonathansoma.com/lede/algorithms-2017/classes/networks/networkx-graphs-from-source-target-dataframe/">website</a>.

```python
import numpy as np
import pandas as pd
from IPython.display import SVG
from scipy import sparse
import networkx as nx
```

### Data preparation
The data are coming from a previous work where I web-scrapped marathon performances from World Athletics Website.
Thanks to the dataset it is possible to get the place of the competition(=Marathon name) for each records.

```python
df = pd.read_csv('Marathon_V2.csv', na_filter=True)
df = df.drop('dummy', 1)
df = df.dropna(subset=['Place of the competition'])
df = df.dropna(subset=['Date'])
df = df.dropna(subset=['Position in the competion'])
df = df.dropna(subset=['Date of birth'])
df.drop(df.columns.difference(['Nationality','Place of the competition']), 1, inplace=True)
```

The first idea was to plot a link between a record (=a performance) and a Marathon name.
The problem is that there are tens of thousands records and almost 100 marathons. Visualizing so many links have not interest as too many datas will be plotted.

Another Idea is then to assign to each country (nationality in the dataframe) the most popular marathon. So a country will have one link (or more if several marathons are equally popular) and a marathon can have many links coming from countries.

```python
df['Weights']=1
df['GroupWeights'] = df.groupby(['Nationality','Place of the competition'])['Weights'].transform('sum')
df = df.drop_duplicates()
df.drop('Weights',1, inplace=True)
df.reset_index(drop=True, inplace=True)
df.head()
```

The data frame ends up like this :

![](/Images/dataframe.png)

### Creating NetworkX graph
Now the graph can be created :

```python
import matplotlib.pyplot as plt

plt.figure(figsize=(100, 100))

g = nx.from_pandas_edgelist(result, source='Nationality', target='Place of the competition')

layout = nx.spring_layout(g,iterations=40)

nx.draw_networkx_edges(g, layout, edge_color='#AAAAAA')

marathon_place = [node for node in g.nodes() if node in result['Place of the competition'].unique()]
nationality = [node for node in g.nodes() if node in result['Nationality'].unique()]

size = [g.degree(node) * 900 for node in g.nodes() if node in result['Place of the competition'].unique()]

marathon_place_dict = dict(zip(marathon_place, marathon_place))
nationality_dict = dict(zip(nationality, nationality))


high_degree_country = [node for node in g.nodes() if node in result['Nationality'].unique() and g.degree(node) > 1]

#draw node of Nationality
nx.draw_networkx_nodes(g, layout, nodelist=nationality, node_size=600, node_color='darkorange')

#draw nodes of Marathon
nx.draw_networkx_nodes(g, layout, nodelist=marathon_place, node_size=size, node_color='mediumturquoise')

#draw high degree country
nx.draw_networkx_nodes(g, layout, nodelist=high_degree_country, node_size=1200, node_color='darkorange')

nx.draw_networkx_labels(g, layout, labels=nationality_dict)
nx.draw_networkx_labels(g, layout, labels=marathon_place_dict)

test = nx.get_edge_attributes(g,'Nationality')

plt.axis('off')
plt.show()
```

By zooming in the center of the figure, the links are more distinguishable.
Blue edges represent marathons and orange ones country (ICO designation). The size of the edge is proportional to its number of links.

![](Images/graph_zoom.png)
<img src="/Images/graph_zoom.png">

https://github.com/ToNi-sn/portfolio/blob/gh-pages/Images/graph_zoom.png?raw=true
Berlin appears to be the most popular marathon with 9 nations linked to it. With a close look we can see some unsuspected correlation. For example, Dubaï marathon is the most ran marathon Ethiopian athletes (the winning prizes of 100k$ and its geographical position must be the reason of its popularity).
/Users/toni/portfolio/Images/graph_zoom.png