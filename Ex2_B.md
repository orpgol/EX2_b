

```python
---
title: "markdown_python"
author: "Orpaz Goldstein"
date: "April 10, 2016"
output: html_document
---

#Ex2_b

##L.A resteraunts along my route
###[Downloaded using google places API.](https://developers.google.com/places/web-service/intro)
####I wanted to find the best retaurants along my route in L.A. Using Google API i could retrieve all retaurants on route and check which are worth eating at.
```


```python
# Import pandas
import pandas as pd
import numpy as np
import math

import json
import pickle
from pandas.io.json import json_normalize

# Import matplotlib and Basemap
import matplotlib.pyplot as plt
import mpl_toolkits
import mpl_toolkits.basemap
from mpl_toolkits.mplot3d import Axes3D


from sklearn.cluster import KMeans
from sklearn import datasets

# Set iPython to display visualization inline
%matplotlib inline

import urllib.request
```


```python
##### I'm going to pull the data using one of my app keys on google and aggregating all results into a single dataframe
```


```python
def geo(lat, long, nextPage):
        response = urllib.request.urlopen('https://maps.googleapis.com/maps/api/place/nearbysearch/json?location=%s,%s&radius=50000&type=restaurant&key=AIzaSyDrfu5pGKZmkGQNBbGV2Xq93d1TSPwYXsg%s' % (lat, long, nextPage))
        #response = urllib.request.urlopen('https://maps.googleapis.com/maps/api/place/nearbysearch/json?location=%s,%s&radius=50000&type=restaurant&key=AIzaSyBY1xMxtE2CysRaQ1mr3FzmrPRDvZTGz20%s' % (lat, long, nextPage))
        #response = urllib.request.urlopen('https://maps.googleapis.com/maps/api/place/nearbysearch/json?location=%s,%s&radius=50000&type=restaurant&key=AIzaSyCGEkQhbkRnQXbiCqXkC7yj-Dppf7Hj0_Y%s' % (lat, long, nextPage))
        return response
```


```python
df = pd.DataFrame()
nextPage = ''
lat = 34
long = -118
for i in range(40):
    resp = geo(str(lat), str(long), nextPage)
    resp = resp.read().decode('utf-8')
    resp = json.loads(resp)
    #nextPage = '&pagetoken=' + resp['next_page_token']
    try:
        resp = json_normalize(resp['results'])
    except:
        break
    df = df.append(resp, ignore_index=True)
    lat += 0.1
    #long += 0.1
```


```python
##### The data collected contains a lot of interesting features i can use later on, like 'place_id' that i could use to get more information on a specific restaurant.
##### Also 'types' shows the types of food a restaurant is classified with.
##### I will focus on loaction, and rating of a restaurant to see how far i will have to stray from my course to eat at a good retaurant.

```


```python
df
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>geometry.location.lat</th>
      <th>geometry.location.lng</th>
      <th>icon</th>
      <th>id</th>
      <th>name</th>
      <th>opening_hours.open_now</th>
      <th>opening_hours.weekday_text</th>
      <th>photos</th>
      <th>place_id</th>
      <th>price_level</th>
      <th>rating</th>
      <th>reference</th>
      <th>scope</th>
      <th>types</th>
      <th>vicinity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>33.988638</td>
      <td>-117.933588</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>afa953bff9bc4ddbe3a1ecd8e12bf0f8c866652d</td>
      <td>Courtyard Los Angeles Hacienda Heights/Orange ...</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 1365, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJU1nvVYTVwoARzubYlyhAu58</td>
      <td>NaN</td>
      <td>4.1</td>
      <td>CpQBiQAAAOmxsD3VEwBwJEBYG0FJqeoUmYIjzr6my2cZf5...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, lodging, point_of_interest,...</td>
      <td>1905 South Azusa Avenue, Hacienda Heights</td>
    </tr>
    <tr>
      <th>1</th>
      <td>34.120917</td>
      <td>-118.133368</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>71591378ea719ff02cc1fec4f3759cc82d868b73</td>
      <td>The Langham Huntington, Pasadena, Los Angeles</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>[{'height': 709, 'html_attributions': ['&lt;a hre...</td>
      <td>ChIJw-P2b77EwoARMLLi0Xxs97E</td>
      <td>NaN</td>
      <td>4.5</td>
      <td>CpQBgQAAAMX_xFEvpAr24mpNsXZpdChcB4g21icDTJyq3x...</td>
      <td>GOOGLE</td>
      <td>[bar, gym, spa, restaurant, food, health, lodg...</td>
      <td>1401 South Oak Knoll Avenue, Pasadena</td>
    </tr>
    <tr>
      <th>2</th>
      <td>33.798911</td>
      <td>-117.917815</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>f582d3b08f6f6d45e235759b775713770a6c2958</td>
      <td>Anaheim Marriott</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 3756, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJAaioVOfX3IARrU_gB08tApE</td>
      <td>NaN</td>
      <td>4.1</td>
      <td>CnRkAAAAdWZNA3ateOwMFgZB41u0mA_U2DtCrlactjjW6M...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, lodging, point_of_interest,...</td>
      <td>700 Convention Way, Anaheim</td>
    </tr>
    <tr>
      <th>3</th>
      <td>33.797680</td>
      <td>-117.916600</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>05d0deaca66ea40be7d59d3fb8db3634b4a31e16</td>
      <td>Courtyard Anaheim Resort/Convention Center</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 423, 'html_attributions': ['&lt;a hre...</td>
      <td>ChIJSSV3pufX3IARlUARrJBn6_I</td>
      <td>NaN</td>
      <td>3.9</td>
      <td>CoQBfgAAAEI9FyPxFJkQqIsfSr_8FiwXDps9io-aszER3Q...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, lodging, point_of_interest,...</td>
      <td>2045 South Harbor Blvd, Anaheim</td>
    </tr>
    <tr>
      <th>4</th>
      <td>34.049626</td>
      <td>-118.255000</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>a6002da196e36f74c4b4d3d2e50838973f9a9cfe</td>
      <td>Hilton Checkers Los Angeles</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>[{'height': 2048, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJU9oTm7THwoARF9dZF0pdn0U</td>
      <td>3</td>
      <td>3.8</td>
      <td>CnRuAAAAf27VP5oa5eHsQ_S-lSU-660snyFCgkMJNlzCd_...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, lodging, point_of_interest,...</td>
      <td>535 South Grand Avenue, Los Angeles</td>
    </tr>
    <tr>
      <th>5</th>
      <td>34.050108</td>
      <td>-118.257048</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>732a4100780e3d0cbc0242119ac7cd4683bd2232</td>
      <td>The Standard Hotel, Downtown LA</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>[{'height': 1382, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJ71dGk7PHwoARHWr_QGbZIGU</td>
      <td>NaN</td>
      <td>3.6</td>
      <td>CoQBcgAAALMrRsq_qbvUs3ZZn4fuOfQAL248LZs0Hhp0LE...</td>
      <td>GOOGLE</td>
      <td>[night_club, restaurant, food, lodging, point_...</td>
      <td>550 South Flower Street, Los Angeles</td>
    </tr>
    <tr>
      <th>6</th>
      <td>33.811405</td>
      <td>-118.138299</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>15471fa27671a68fde84326411c77bc9af5d8060</td>
      <td>Long Beach Marriott</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 423, 'html_attributions': ['&lt;a hre...</td>
      <td>ChIJaxXXgyEy3YARo3UO0Y5o8ao</td>
      <td>NaN</td>
      <td>4.1</td>
      <td>CnRnAAAAcEM-yBZN2fMxjvwUtZp1ihcw3elmJXJhWiY9C0...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, lodging, point_of_interest,...</td>
      <td>4700 Airport Plaza Drive, Long Beach</td>
    </tr>
    <tr>
      <th>7</th>
      <td>33.804014</td>
      <td>-118.141761</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>588ce3f0481504486ba5e2eee9b398bf6ab2abb2</td>
      <td>Holiday Inn Long Beach Airport Hotel and Confe...</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 1365, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJGdQGnIkx3YARHcjYXaRh0NE</td>
      <td>NaN</td>
      <td>3.3</td>
      <td>CpQBjwAAAIrtS3KT0le8V9nU1leW3p279SBb-NTNurIOgN...</td>
      <td>GOOGLE</td>
      <td>[bar, restaurant, food, lodging, point_of_inte...</td>
      <td>2640 North Lakewood Boulevard, Long Beach</td>
    </tr>
    <tr>
      <th>8</th>
      <td>33.707053</td>
      <td>-117.919920</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>407fa707dbbd34fa84a659c71e5a394bb3c485f7</td>
      <td>Courtyard Costa Mesa South Coast Metro</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 1108, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJPW8tG6bY3IARbyoRXqhaoII</td>
      <td>NaN</td>
      <td>3.8</td>
      <td>CoQBegAAAFrZ5EDGkQ64xTAK_1ZK5fRL3u0I4l2vpSs_rW...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, lodging, point_of_interest,...</td>
      <td>3002 South Harbor Boulevard, Santa Ana</td>
    </tr>
    <tr>
      <th>9</th>
      <td>33.949045</td>
      <td>-118.385633</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>e5d8c8ec3d6bb811c3fa8fab4dbca0660795bde4</td>
      <td>Renaissance Los Angeles Airport Hotel</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 250, 'html_attributions': ['&lt;a hre...</td>
      <td>ChIJh_0uR9a2woARX0G6s9ArmVQ</td>
      <td>NaN</td>
      <td>4.0</td>
      <td>CoQBeAAAACCULHBXyaoa56Tt0LJiJLidh1R90JRkcdFUdt...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, lodging, point_of_interest,...</td>
      <td>9620 Airport Boulevard, Los Angeles</td>
    </tr>
    <tr>
      <th>10</th>
      <td>34.155906</td>
      <td>-118.345609</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>8e5f59dde7ee02ee83f56487091eaa3602e7b905</td>
      <td>Hotel Amarano</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>[{'height': 374, 'html_attributions': ['&lt;a hre...</td>
      <td>ChIJEVcDt9K_woAR2mKE2yPtaEc</td>
      <td>NaN</td>
      <td>3.7</td>
      <td>CmRgAAAAOigbf1c-toOULZj11PtflB0B-4HoMmfAyZd17p...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, lodging, point_of_interest,...</td>
      <td>322 North Pass Avenue, Burbank</td>
    </tr>
    <tr>
      <th>11</th>
      <td>34.023826</td>
      <td>-118.394103</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>53ba0a2174edf36eafc68549dcd780814bc05c18</td>
      <td>The Culver Hotel</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 512, 'html_attributions': ['&lt;a hre...</td>
      <td>ChIJA4I_vym6woAR3iRMtACs3ng</td>
      <td>NaN</td>
      <td>4.2</td>
      <td>CnRjAAAAeKfgyqoOVlb_PPveUNwtDwmgO4eqic4PFXZhkX...</td>
      <td>GOOGLE</td>
      <td>[bar, restaurant, food, lodging, point_of_inte...</td>
      <td>9400 Culver Boulevard, Culver City</td>
    </tr>
    <tr>
      <th>12</th>
      <td>33.923125</td>
      <td>-118.391442</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>5bfbf630167811f49d8736be31ff23ab2f69bd3a</td>
      <td>Courtyard Los Angeles LAX/El Segundo</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 1366, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJHateZ7O2woARlr8FueMZDt0</td>
      <td>NaN</td>
      <td>3.9</td>
      <td>CoQBeAAAANyPnK6vlllC03XVTDKYiAmuQ5yiv9NdvYUd-B...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, lodging, point_of_interest,...</td>
      <td>2000 East Mariposa Avenue, El Segundo</td>
    </tr>
    <tr>
      <th>13</th>
      <td>33.899980</td>
      <td>-118.388292</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>654ff39cbcfaea9dd9dcd03a86e5d271e20e5da8</td>
      <td>Manhattan Beach Marriott</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 423, 'html_attributions': ['&lt;a hre...</td>
      <td>ChIJPx5R50a0woAR0t5Xfcx6AKY</td>
      <td>NaN</td>
      <td>4.2</td>
      <td>CnRsAAAAwxgUOfOlpdAGLOCk573tIjKTV40_9fSSG9oILN...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, lodging, point_of_interest,...</td>
      <td>1400 Parkview Avenue, Manhattan Beach</td>
    </tr>
    <tr>
      <th>14</th>
      <td>34.067740</td>
      <td>-118.398923</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>2c7b599c61a555d581ca27ccf27cba093a6413c4</td>
      <td>Montage Beverly Hills</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 1533, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJ_wvqLf-7woAR1VZCoxWQsb0</td>
      <td>4</td>
      <td>4.6</td>
      <td>CnRpAAAAJqEwr7VlxZELQ932AUftIwD8z8OmolxsDI0y2-...</td>
      <td>GOOGLE</td>
      <td>[beauty_salon, spa, restaurant, food, health, ...</td>
      <td>225 North Canon Drive, Beverly Hills</td>
    </tr>
    <tr>
      <th>15</th>
      <td>34.071225</td>
      <td>-118.400316</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>383c8c04218143e726604b725e4bba64aee81132</td>
      <td>Boe at the Crescent Hotel</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>ChIJHbWPFQe8woARYHFncvtTEnA</td>
      <td>NaN</td>
      <td>4.0</td>
      <td>CnRsAAAAuVVEmCezv2SYFFgsR30Pl4s8yyYbgIrp8V90d3...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, lodging, point_of_interest,...</td>
      <td>403 North Crescent Drive, Beverly Hills</td>
    </tr>
    <tr>
      <th>16</th>
      <td>33.678328</td>
      <td>-117.849819</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>2257da9893e91b12174922a007d70e639720eb08</td>
      <td>Irvine Marriott</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 423, 'html_attributions': ['&lt;a hre...</td>
      <td>ChIJx6oOPoze3IAR4aCGakw8AA4</td>
      <td>NaN</td>
      <td>4.1</td>
      <td>CnRiAAAA7vWyX5tFhvyw-7plg8hqIaWmm5QciDYeh_miT3...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, lodging, point_of_interest,...</td>
      <td>18000 Von Karman Avenue, Irvine</td>
    </tr>
    <tr>
      <th>17</th>
      <td>33.678045</td>
      <td>-117.839938</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>9f3af30b828b8b2902fe5c59d970b9c271a69d89</td>
      <td>Hotel Irvine</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>[{'height': 406, 'html_attributions': ['&lt;a hre...</td>
      <td>ChIJxwbm3oXe3IARV0ohulXEqYQ</td>
      <td>NaN</td>
      <td>3.8</td>
      <td>CmRgAAAA3NoQiS58uJ2cruCZHFxawbjyU5c5_riXJ1uj-q...</td>
      <td>GOOGLE</td>
      <td>[bar, restaurant, food, lodging, point_of_inte...</td>
      <td>17900 Jamboree Road, Irvine</td>
    </tr>
    <tr>
      <th>18</th>
      <td>34.070412</td>
      <td>-117.574676</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>2e625232a0666b98e14792f54eb39e2eafb22ddb</td>
      <td>Ontario Airport Hotel &amp; Conference Center</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 250, 'html_attributions': ['&lt;a hre...</td>
      <td>ChIJh8pUSqA1w4ARnmmYTDrKXUg</td>
      <td>NaN</td>
      <td>4.0</td>
      <td>CoQBfAAAAMapLaW54NMysuDOKF4yZxClYdAlwJ5rQi1FTD...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, lodging, point_of_interest,...</td>
      <td>700 North Haven Avenue, Ontario</td>
    </tr>
    <tr>
      <th>19</th>
      <td>33.985170</td>
      <td>-118.440693</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>44dc6eeb410888e83b9a388b0c475f8bca1b100e</td>
      <td>Hotel MdR - A DoubleTree by Hilton Marina del Rey</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>[{'height': 1365, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJoaEnS4a6woARh_8TpCEd1HU</td>
      <td>NaN</td>
      <td>4.1</td>
      <td>CpQBhQAAANXUuvngKlL-yLNZyJdfvxHkUXFkPW5Bi9sTS0...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, lodging, point_of_interest,...</td>
      <td>13480 Maxella Avenue, Marina del Rey</td>
    </tr>
    <tr>
      <th>20</th>
      <td>34.120917</td>
      <td>-118.133368</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>71591378ea719ff02cc1fec4f3759cc82d868b73</td>
      <td>The Langham Huntington, Pasadena, Los Angeles</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>[{'height': 709, 'html_attributions': ['&lt;a hre...</td>
      <td>ChIJw-P2b77EwoARMLLi0Xxs97E</td>
      <td>NaN</td>
      <td>4.5</td>
      <td>CpQBgQAAAA40Jsg91-7hCYx9t7ZIW2oJjEfzaM5M2KVXA6...</td>
      <td>GOOGLE</td>
      <td>[bar, gym, spa, restaurant, food, health, lodg...</td>
      <td>1401 South Oak Knoll Avenue, Pasadena</td>
    </tr>
    <tr>
      <th>21</th>
      <td>33.988638</td>
      <td>-117.933588</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>afa953bff9bc4ddbe3a1ecd8e12bf0f8c866652d</td>
      <td>Courtyard Los Angeles Hacienda Heights/Orange ...</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 1365, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJU1nvVYTVwoARzubYlyhAu58</td>
      <td>NaN</td>
      <td>4.1</td>
      <td>CpQBiQAAACsfS411JtYOxAECwB-t_ECGIHEA4UiXG7EnEW...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, lodging, point_of_interest,...</td>
      <td>1905 South Azusa Avenue, Hacienda Heights</td>
    </tr>
    <tr>
      <th>22</th>
      <td>34.049626</td>
      <td>-118.255000</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>a6002da196e36f74c4b4d3d2e50838973f9a9cfe</td>
      <td>Hilton Checkers Los Angeles</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>[{'height': 2048, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJU9oTm7THwoARF9dZF0pdn0U</td>
      <td>3</td>
      <td>3.8</td>
      <td>CnRuAAAA6sthPHIi_3vcAhuzOK-_7iWpOWUGmSQ2TC_Usp...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, lodging, point_of_interest,...</td>
      <td>535 South Grand Avenue, Los Angeles</td>
    </tr>
    <tr>
      <th>23</th>
      <td>34.050108</td>
      <td>-118.257048</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>732a4100780e3d0cbc0242119ac7cd4683bd2232</td>
      <td>The Standard Hotel, Downtown LA</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>[{'height': 1382, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJ71dGk7PHwoARHWr_QGbZIGU</td>
      <td>NaN</td>
      <td>3.6</td>
      <td>CoQBcgAAAFyWjItK-KgJMh586vusY9g-EqdesuO7vMDhcV...</td>
      <td>GOOGLE</td>
      <td>[night_club, restaurant, food, lodging, point_...</td>
      <td>550 South Flower Street, Los Angeles</td>
    </tr>
    <tr>
      <th>24</th>
      <td>34.155906</td>
      <td>-118.345609</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>8e5f59dde7ee02ee83f56487091eaa3602e7b905</td>
      <td>Hotel Amarano</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>[{'height': 374, 'html_attributions': ['&lt;a hre...</td>
      <td>ChIJEVcDt9K_woAR2mKE2yPtaEc</td>
      <td>NaN</td>
      <td>3.7</td>
      <td>CmRgAAAAlfWt7GY5VOLXQMT0emZlJkXLq1MTXIn-gSpHQf...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, lodging, point_of_interest,...</td>
      <td>322 North Pass Avenue, Burbank</td>
    </tr>
    <tr>
      <th>25</th>
      <td>33.798911</td>
      <td>-117.917815</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>f582d3b08f6f6d45e235759b775713770a6c2958</td>
      <td>Anaheim Marriott</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 3756, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJAaioVOfX3IARrU_gB08tApE</td>
      <td>NaN</td>
      <td>4.1</td>
      <td>CnRkAAAANjjxXgkGPpsfHEVMlNQdzdw8YjcDVNfmtNqKjT...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, lodging, point_of_interest,...</td>
      <td>700 Convention Way, Anaheim</td>
    </tr>
    <tr>
      <th>26</th>
      <td>33.797680</td>
      <td>-117.916600</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>05d0deaca66ea40be7d59d3fb8db3634b4a31e16</td>
      <td>Courtyard Anaheim Resort/Convention Center</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 423, 'html_attributions': ['&lt;a hre...</td>
      <td>ChIJSSV3pufX3IARlUARrJBn6_I</td>
      <td>NaN</td>
      <td>3.9</td>
      <td>CoQBfgAAALFF85JY3RsXzOla5MZxDUnf1pq7_jPWJuJUua...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, lodging, point_of_interest,...</td>
      <td>2045 South Harbor Blvd, Anaheim</td>
    </tr>
    <tr>
      <th>27</th>
      <td>33.811405</td>
      <td>-118.138299</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>15471fa27671a68fde84326411c77bc9af5d8060</td>
      <td>Long Beach Marriott</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 423, 'html_attributions': ['&lt;a hre...</td>
      <td>ChIJaxXXgyEy3YARo3UO0Y5o8ao</td>
      <td>NaN</td>
      <td>4.1</td>
      <td>CnRnAAAA_PH3v-fjs4P_eiXrqkcLawZBsrmsHLNmCtvH5X...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, lodging, point_of_interest,...</td>
      <td>4700 Airport Plaza Drive, Long Beach</td>
    </tr>
    <tr>
      <th>28</th>
      <td>33.804014</td>
      <td>-118.141761</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>588ce3f0481504486ba5e2eee9b398bf6ab2abb2</td>
      <td>Holiday Inn Long Beach Airport Hotel and Confe...</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 1365, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJGdQGnIkx3YARHcjYXaRh0NE</td>
      <td>NaN</td>
      <td>3.3</td>
      <td>CpQBjwAAANyokVb_RmE7jsOGmPLr2DZCJ0dZpzWNQ7yawz...</td>
      <td>GOOGLE</td>
      <td>[bar, restaurant, food, lodging, point_of_inte...</td>
      <td>2640 North Lakewood Boulevard, Long Beach</td>
    </tr>
    <tr>
      <th>29</th>
      <td>34.067740</td>
      <td>-118.398923</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>2c7b599c61a555d581ca27ccf27cba093a6413c4</td>
      <td>Montage Beverly Hills</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 1533, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJ_wvqLf-7woAR1VZCoxWQsb0</td>
      <td>4</td>
      <td>4.6</td>
      <td>CnRpAAAANRcStuRnVG0A7wWXPqLu6-HSoUeHbakgXAWFTh...</td>
      <td>GOOGLE</td>
      <td>[beauty_salon, spa, restaurant, food, health, ...</td>
      <td>225 North Canon Drive, Beverly Hills</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>681</th>
      <td>37.375526</td>
      <td>-118.414051</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>1eea624e1d46e7b52ac5cc4a57ed378d7f3e342f</td>
      <td>Astorga's Mexican Restaurant</td>
      <td>False</td>
      <td>[]</td>
      <td>[{'height': 2448, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJn5acxRUWvoARxMrbdg7Q3Ko</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>CnRwAAAAZIsTjLJ-phyC4ph6YJ7R0fJ-uNM1bAKXXzDjHX...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>2206 North Sierra Highway, Bishop</td>
    </tr>
    <tr>
      <th>682</th>
      <td>37.364202</td>
      <td>-118.395553</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>8c35dcf20e1787b5e49b80f27c6c06905778e3a7</td>
      <td>Jack's Restaurant</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>[{'height': 4128, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJv_MTm_89voARjDU9pY8xAOM</td>
      <td>2</td>
      <td>4.1</td>
      <td>CnRlAAAAFmAJHQoYQa5GVo6_u59mrabjhoOo2ct7lcFmOQ...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>437 North Main Street, Bishop</td>
    </tr>
    <tr>
      <th>683</th>
      <td>37.371730</td>
      <td>-118.395002</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>2b9effad687992b97c8144883e540fc542f4716b</td>
      <td>Denny's</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 450, 'html_attributions': ['&lt;a hre...</td>
      <td>ChIJP07-GgcWvoARTcBvFyRBD38</td>
      <td>1</td>
      <td>3.1</td>
      <td>CmRaAAAA_atd9fnkNkwyNVJOlZ-9_llg3j3RD5I58HWXM_...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>1014 North Main Street, Bishop</td>
    </tr>
    <tr>
      <th>684</th>
      <td>37.369960</td>
      <td>-118.395644</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>e0add14751f9e41526a4c0b241dd4d531f57bdd0</td>
      <td>Aaron Schat's Roadhouse</td>
      <td>False</td>
      <td>[]</td>
      <td>[{'height': 2988, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJXRm3KwcWvoARXQ8yXM6UzWA</td>
      <td>NaN</td>
      <td>4.5</td>
      <td>CnRqAAAArgr33DuIRy6DoU74eYvK2fe2VJUk4CG6iprCYX...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>871 North Main Street, Bishop</td>
    </tr>
    <tr>
      <th>685</th>
      <td>37.369964</td>
      <td>-118.395648</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>e672df3a1b2472dac1a02e0cf61a6ca1606457f2</td>
      <td>Schat's Roadhouse</td>
      <td>False</td>
      <td>[]</td>
      <td>NaN</td>
      <td>ChIJ1VC2KwcWvoARMYADKn5nleQ</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>CnRlAAAAA0NpKTugWCEo2_fVdlqB6oD9Jur4YMEd_6cm0g...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>871 North Main Street, Bishop</td>
    </tr>
    <tr>
      <th>686</th>
      <td>37.165070</td>
      <td>-118.290078</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>dada77b04177db10c0cb2d51240654429e9bf4ee</td>
      <td>Country Kitchen</td>
      <td>False</td>
      <td>[]</td>
      <td>[{'height': 412, 'html_attributions': ['&lt;a hre...</td>
      <td>ChIJ3QbvUP49voARXk4HmXXeiAw</td>
      <td>NaN</td>
      <td>3.7</td>
      <td>CnRiAAAAV8phEJy1rIoplwMjDPH0xZs2CSkoMJEPeH0Hm5...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>181 Main Street, Big Pine</td>
    </tr>
    <tr>
      <th>687</th>
      <td>37.361507</td>
      <td>-118.424374</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>b90c36605f7dd18e70f099c980a65b610ff1abd7</td>
      <td>Bishop Burger Barn</td>
      <td>False</td>
      <td>[]</td>
      <td>[{'height': 2368, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJgVSPjyYWvoARUBGCbqnGtyQ</td>
      <td>NaN</td>
      <td>4.8</td>
      <td>CnRlAAAA2S6EKUaBwBIrNgbsP_nBnyQe04eIShfCKcPQyo...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>2675 West Line Street, Bishop</td>
    </tr>
    <tr>
      <th>688</th>
      <td>37.366104</td>
      <td>-118.395611</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>1f98be1229fd41dc1942a6cae4dc613e377b0e02</td>
      <td>Yamatani Japanese Restaurant &amp; Sushi Bar</td>
      <td>False</td>
      <td>[]</td>
      <td>[{'height': 2448, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJwTT_NQAWvoARPUFuaUggCjc</td>
      <td>2</td>
      <td>3.9</td>
      <td>CoQBewAAAEQnhuzU1_eiPYOxc7yI6gm815UArj-fA2RL3f...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>635 North Main Street, Bishop</td>
    </tr>
    <tr>
      <th>689</th>
      <td>37.371140</td>
      <td>-118.395100</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>e0c6765f904b6a444a93763e5cc507b0e853900d</td>
      <td>Pizza Factory</td>
      <td>False</td>
      <td>[]</td>
      <td>[{'height': 359, 'html_attributions': ['&lt;a hre...</td>
      <td>ChIJv2EaGAcWvoARCakbb2rjMEk</td>
      <td>NaN</td>
      <td>4.3</td>
      <td>CmRgAAAAeaywF1NVXyWEg7fnFVlVDjg-Er_Loxl5wEsCRJ...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>970 North Main Street, Bishop</td>
    </tr>
    <tr>
      <th>690</th>
      <td>37.376922</td>
      <td>-118.421880</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>65003073fb312dc151e9f084ac3d4cae6fc2aa29</td>
      <td>Subway</td>
      <td>True</td>
      <td>[]</td>
      <td>NaN</td>
      <td>ChIJrWZcST8WvoARipO2DmYakAQ</td>
      <td>1</td>
      <td>3.6</td>
      <td>CmRZAAAAz9hgayfjlKmk8BVi3IL-ugnWzCHSg5cjcxZit-...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>1341 Rocking West Drive, Bishop</td>
    </tr>
    <tr>
      <th>691</th>
      <td>37.373500</td>
      <td>-118.394704</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>7c3c9b5b9046290f8fe45a964f71c19f36c86e65</td>
      <td>The Upper Crust Pizza</td>
      <td>False</td>
      <td>[]</td>
      <td>[{'height': 1152, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJ__fVvQcWvoARDfmmpwxjxNc</td>
      <td>NaN</td>
      <td>4.3</td>
      <td>CnRpAAAATgffyHCZttkS-b_ZN_CaKen1gddaRfpvsNgIYx...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>1180 North Main Street #100, Bishop</td>
    </tr>
    <tr>
      <th>692</th>
      <td>37.368152</td>
      <td>-118.395067</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>4c6475fae54cfd69bd86c2ed5be40c74e7143ed6</td>
      <td>Holy Smoke Texas Style BBQ</td>
      <td>False</td>
      <td>[]</td>
      <td>[{'height': 2448, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJq31y7gAWvoARTGdUSQ3LXuk</td>
      <td>2</td>
      <td>4.4</td>
      <td>CnRuAAAAubFC1dM_1uJHzA12WtZkk6KB5TSOMr_vU8tgd8...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>772 North Main Street, Bishop</td>
    </tr>
    <tr>
      <th>693</th>
      <td>37.359986</td>
      <td>-118.395537</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>04915af7923d0414af0d448f794c86c03e8e0674</td>
      <td>Great Basin Bakery</td>
      <td>False</td>
      <td>[]</td>
      <td>[{'height': 210, 'html_attributions': ['&lt;a hre...</td>
      <td>ChIJxcqSgP49voARIeu36bck2SU</td>
      <td>NaN</td>
      <td>4.6</td>
      <td>CnRlAAAA6cOAnKK4uWWcfdS62bD2c8QJ0IcfSle1tbZ5OZ...</td>
      <td>GOOGLE</td>
      <td>[bakery, cafe, restaurant, store, food, point_...</td>
      <td>275 South Main Street, Bishop</td>
    </tr>
    <tr>
      <th>694</th>
      <td>37.370560</td>
      <td>-118.394991</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>6fed5affee8923f36d7137c0dd3eaed41a9fe7f7</td>
      <td>Taco Bell</td>
      <td>False</td>
      <td>[]</td>
      <td>NaN</td>
      <td>ChIJQ3hIUOI9voARupEWJtIXAII</td>
      <td>1</td>
      <td>3.2</td>
      <td>CmRdAAAArPlQtdqjn8NmMt4ZtpcoQ8SLs_UmHPEuTosVnD...</td>
      <td>GOOGLE</td>
      <td>[meal_takeaway, restaurant, food, point_of_int...</td>
      <td>926 North Main Street, Bishop</td>
    </tr>
    <tr>
      <th>695</th>
      <td>37.365282</td>
      <td>-118.395057</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>ea864643b31f9c3d449398954b513c08144bf428</td>
      <td>McDonald's</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 1265, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJr7fPJgAWvoARbq_hkVnGp5w</td>
      <td>1</td>
      <td>3.0</td>
      <td>CmReAAAAhyBuAoFE1u4AENr6v4lCNCqBjaFFb29M472XSE...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>562 North Main Street, Bishop</td>
    </tr>
    <tr>
      <th>696</th>
      <td>37.375325</td>
      <td>-118.426176</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>85f6d47b3d75511e8e1c9cc97752e2e92c9e337f</td>
      <td>TuKaNovie Restaurant</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 1034, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJi_wFBDgWvoAROHBJZ_4VpXQ</td>
      <td>NaN</td>
      <td>3.9</td>
      <td>CnRnAAAAf1-oAjD5Pt9EPS_0B_Rh6u05vMkCO_nI4tidLi...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, casino, point_of_interest, ...</td>
      <td>2742 North Sierra Highway, Bishop</td>
    </tr>
    <tr>
      <th>697</th>
      <td>37.367910</td>
      <td>-118.394892</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>29e154fbdf5ea519f6f00f6f5980863e4b75d066</td>
      <td>Green Burrito</td>
      <td>True</td>
      <td>[]</td>
      <td>NaN</td>
      <td>ChIJ5cVZ8QAWvoAReFHojUeiIfM</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>CnRhAAAAPHPNCjoCNmcPbZvHaFE4jUoQoA7aNh7fDYLqXn...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>768 North Main Street, Bishop</td>
    </tr>
    <tr>
      <th>698</th>
      <td>37.371503</td>
      <td>-118.395072</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>dca266f19c912fafcae59417c0df0addbd2998fe</td>
      <td>KFC</td>
      <td>False</td>
      <td>[]</td>
      <td>[{'height': 615, 'html_attributions': ['&lt;a hre...</td>
      <td>ChIJqb0-EQcWvoARV_fAtO1prCM</td>
      <td>1</td>
      <td>2.7</td>
      <td>CmRWAAAAWcdmnHJw4aAnwRPvjuIjzLYTJFnksN52cT4FFs...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>1006 North Main Street, Bishop</td>
    </tr>
    <tr>
      <th>699</th>
      <td>37.374548</td>
      <td>-118.391303</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>b1e52252c8bd546828d41bd5ba4c41662ff4d8fc</td>
      <td>Little Caesars Pizza</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>ChIJmWyhyQcWvoAR1niouvgksjg</td>
      <td>1</td>
      <td>NaN</td>
      <td>CnRnAAAA0xpJC8BLoElUXYk59y96N_yuVOf1al3WyG_2k8...</td>
      <td>GOOGLE</td>
      <td>[meal_takeaway, restaurant, food, point_of_int...</td>
      <td>1200 North Main Street, Bishop</td>
    </tr>
    <tr>
      <th>700</th>
      <td>37.355653</td>
      <td>-118.395620</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>446b3f8c285217bf824b0488deb1df3c892ab803</td>
      <td>Jack in the Box</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 931, 'html_attributions': ['&lt;a hre...</td>
      <td>ChIJGzLeJf09voAR8HlD_259pJQ</td>
      <td>1</td>
      <td>4.1</td>
      <td>CnRjAAAAusXZVn3BBPzOPvD7iKKIDGKnUB4zBH7_GO16jf...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>575 South Main Street, Bishop</td>
    </tr>
    <tr>
      <th>701</th>
      <td>37.375526</td>
      <td>-118.414051</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>1eea624e1d46e7b52ac5cc4a57ed378d7f3e342f</td>
      <td>Astorga's Mexican Restaurant</td>
      <td>False</td>
      <td>[]</td>
      <td>[{'height': 2448, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJn5acxRUWvoARxMrbdg7Q3Ko</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>CnRwAAAAF514SJtvIaRfMpYUG7GkqZiS2jQguSWKieoAmn...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>2206 North Sierra Highway, Bishop</td>
    </tr>
    <tr>
      <th>702</th>
      <td>37.364202</td>
      <td>-118.395553</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>8c35dcf20e1787b5e49b80f27c6c06905778e3a7</td>
      <td>Jack's Restaurant</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>[{'height': 4128, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJv_MTm_89voARjDU9pY8xAOM</td>
      <td>2</td>
      <td>4.1</td>
      <td>CnRlAAAAVRHpBaxFBTqBdmiXb5nTy-OEe2BgdY4Qz1Tn_r...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>437 North Main Street, Bishop</td>
    </tr>
    <tr>
      <th>703</th>
      <td>37.371730</td>
      <td>-118.395002</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>2b9effad687992b97c8144883e540fc542f4716b</td>
      <td>Denny's</td>
      <td>True</td>
      <td>[]</td>
      <td>[{'height': 450, 'html_attributions': ['&lt;a hre...</td>
      <td>ChIJP07-GgcWvoARTcBvFyRBD38</td>
      <td>1</td>
      <td>3.1</td>
      <td>CmRaAAAA6yXvxaQDmkG_sh8hTDyXa-kV7F9021KlLMwFun...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>1014 North Main Street, Bishop</td>
    </tr>
    <tr>
      <th>704</th>
      <td>37.369960</td>
      <td>-118.395644</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>e0add14751f9e41526a4c0b241dd4d531f57bdd0</td>
      <td>Aaron Schat's Roadhouse</td>
      <td>False</td>
      <td>[]</td>
      <td>[{'height': 2988, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJXRm3KwcWvoARXQ8yXM6UzWA</td>
      <td>NaN</td>
      <td>4.5</td>
      <td>CnRqAAAAigRHoMWblY_ZHbqswM6bjBNbgMQOWx_rNVN7xf...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>871 North Main Street, Bishop</td>
    </tr>
    <tr>
      <th>705</th>
      <td>37.369964</td>
      <td>-118.395648</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>e672df3a1b2472dac1a02e0cf61a6ca1606457f2</td>
      <td>Schat's Roadhouse</td>
      <td>False</td>
      <td>[]</td>
      <td>NaN</td>
      <td>ChIJ1VC2KwcWvoARMYADKn5nleQ</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>CnRlAAAAb_Jl-5hnYjla-8EV9TyF7FzVqqSYn0pw4sPSd4...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>871 North Main Street, Bishop</td>
    </tr>
    <tr>
      <th>706</th>
      <td>37.361507</td>
      <td>-118.424374</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>b90c36605f7dd18e70f099c980a65b610ff1abd7</td>
      <td>Bishop Burger Barn</td>
      <td>False</td>
      <td>[]</td>
      <td>[{'height': 2368, 'html_attributions': ['&lt;a hr...</td>
      <td>ChIJgVSPjyYWvoARUBGCbqnGtyQ</td>
      <td>NaN</td>
      <td>4.8</td>
      <td>CnRlAAAAdwJ19OC1vNGIf5hwmW468rPcrzFRNmH_f_3sPz...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>2675 West Line Street, Bishop</td>
    </tr>
    <tr>
      <th>707</th>
      <td>37.375651</td>
      <td>-118.416007</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>c13f13d41d32bc91dc4d6d547fa150c53ee75adf</td>
      <td>Petite Pantry</td>
      <td>False</td>
      <td>[]</td>
      <td>NaN</td>
      <td>ChIJC4b4ixUWvoAReS_Dulp5qsg</td>
      <td>2</td>
      <td>4.3</td>
      <td>CnRhAAAAJ_Xd7nodHjfhEpV6RxEFBDlzjNaP9Ct0yAa9Vh...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>2278 North Sierra Highway, Bishop</td>
    </tr>
    <tr>
      <th>708</th>
      <td>37.527288</td>
      <td>-118.365064</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>4b3cb694f2784931296355402f23e177c48a27fe</td>
      <td>Valley Dogs Catering</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>ChIJDfvYNXQSvoARZgSj7J336Y0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>CnRoAAAAfmgplSMRNvFRKMqzmY8DBtjrzh9nEDSg8M30L5...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>106 Mountain View Avenue, Bishop</td>
    </tr>
    <tr>
      <th>709</th>
      <td>37.372805</td>
      <td>-118.363904</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>7c79991e5ebffe9dfb1f3707b4bdb953c034b00a</td>
      <td>Thai Thai Restaurant</td>
      <td>False</td>
      <td>[]</td>
      <td>[{'height': 947, 'html_attributions': ['&lt;a hre...</td>
      <td>ChIJi42H1SA-voARQlA3iA6anNM</td>
      <td>1</td>
      <td>3.9</td>
      <td>CnRoAAAA7Zf5iR-_ralbYdKFWp9H4ICn9fYlbpOj--3TsF...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>703 Airport Road, Bishop</td>
    </tr>
    <tr>
      <th>710</th>
      <td>37.527288</td>
      <td>-118.365064</td>
      <td>https://maps.gstatic.com/mapfiles/place_api/ic...</td>
      <td>4b3cb694f2784931296355402f23e177c48a27fe</td>
      <td>Valley Dogs Catering</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>ChIJDfvYNXQSvoARZgSj7J336Y0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>CnRoAAAA8Ri0jmQzAzILaSSOZWf5hNJhlubvgqWmE1H2a8...</td>
      <td>GOOGLE</td>
      <td>[restaurant, food, point_of_interest, establis...</td>
      <td>106 Mountain View Avenue, Bishop</td>
    </tr>
  </tbody>
</table>
<p>711 rows × 15 columns</p>
</div>




```python
##### First, I will plot a 3D scatter to see if i can find interesting clusters.
##### I will plot by lat, long, and rating to see how the 3 correlate.
```


```python
fig = plt.figure(figsize=(10, 10))
ax = fig.add_subplot(111, projection='3d')
n = len(df)

xs = list(df['geometry.location.lat'])
ys = list(df['geometry.location.lng'])
zs = list(df['rating'])
ax.scatter(xs, ys, zs, c=c, marker=m)

ax.set_xlabel('Lat')
ax.set_ylabel('Long')
ax.set_zlabel('Rating')

plt.show()
```


![png](output_8_0.png)



```python
##### I can see 2 main clusters: one between lats 33 - 35, and one between lats 36 - 37
##### So these two areas are where i should look for good restaurants.
##### Next, i will want to know how far are these resstaurants from my point of origin, so i will plot in 2D the distance from origin vs rating
```


```python
y1 = [math.sqrt((lat - df['geometry.location.lat'][i])**2 + (long - df['geometry.location.lng'][i])**2) for i in range(len(df))]
x1 = df['rating']


fig = plt.figure(figsize=(30, 15))
ax1 = fig.add_subplot(111)
# ax2.spines["left"].set_position(("axes", -0.05))
# ax3.spines["left"].set_position(("axes", -0.1))
# ax4.spines['right'].set_color('yellow')
# ax3.spines['left'].set_color('green')
# ax2.spines['left'].set_color('blue')
# ax4.spines['left'].set_color('red')
#line1 = ax1.plot(x1,y1,'bo')
scatter = ax1.scatter(x1,y1,c=y1,s=50)
ax1.locator_params(axis = 'y', nbins = 40)
ax1.margins(0.05)
#plt.legend((line1,), ('Distance/Distance'))
plt.colorbar(scatter)
plt.show()
```


![png](output_10_0.png)



```python
##### I can see some good retaurants can be found near my origin (blue means closer) with good rating too! maybe i sould start by eating there.
##### Lastly, I would want to generate a map with all my collected restaurants so i could remember where i should stop and look for a good place to eat.
```


```python
fig = plt.figure(figsize=(50, 50))
themap = Basemap(projection='gall',
              llcrnrlon = -122,
              llcrnrlat = 31.592020,
              urcrnrlon = -112.857734,
              urcrnrlat = 39.145738,
              resolution = 'h',
              area_thresh = 0.1,
              )
themap.drawcoastlines()
themap.drawcountries()
themap.fillcontinents(color = 'gainsboro')
themap.drawmapboundary(fill_color='steelblue')

lats = list(df['geometry.location.lat'])
lons = list(df['geometry.location.lng'])
x,y = themap(lons, lats)
themap.plot(x, y, 
            'o',                    # marker shape
            color='Indigo',         # marker colour
            markersize=10            # marker size
            )

plt.show()

```


![png](output_12_0.png)



```python
##### To sum up, I managed to find high rated restaurants along my route, and filter them by distance and closeness to me. 
##### To continue, i will use the API to collect more data on restaurants above a certain threshhold, and cluster them by types of food.
##### I could then create a map with the clusters by type of food and decide where to eat by deciding what i feel like eating at that point.
```
