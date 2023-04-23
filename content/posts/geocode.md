---
title: "Geopy: Geolocate Addresses Easily"
date: 2023-04-02T10:26:48+02:00
draft: false
cover:
    image: img/geopy.JPG
    alt: 'Geopy'
tags: ["Geocode", "API"]
categories: ["Python"]
---

## Context

Hello there fellow data enthusiasts! Today, I want to tell you about a really cool Python code that I came across recently. But first, let me tell you a little bit about my journey.

As a junior data analyst, I have tried to use different Geocoding APIs such as Google Maps and Bing Maps, but I quickly discovered that there were limitations on the number of requests I could make. So, I decided to look for other options that would allow me to experiment with large datasets.

I tried to use Excel via Power Query to generate a Geocode function, which worked to a certain extent, but using Python just gave me more flexibility. This led me to discover the Nominatim geocoding service provided by OpenStreetMap.

The goal of my project was to compare two files that had company addresses, but were not written in the same way (not normalized or standardized). This code helped me retrieve latitude and longitude coordinates for French cities listed in an Excel file so that I could perform the comparison with ease.

## The code

So, let's dive into the code, shall we?

### Import Nominatim
```
pip install geopy

from geopy.geocoders import Nominatim
geolocator = Nominatim(user_agent="my-app")
```

First, we import the Nominatim geocoding module from the geopy library and initialize a geolocator object. 

### Defining and testing the function
```
location = "D'Yvrac"
reverse_location = geolocator.geocode(location)
reverse_location
```

Then we define a geocode function that sends an HTTP GET request to the Nominatim service with an address parameter to retrieve location data in JSON format.

In this case, I was testing the location 'D'Yvrac'. Yvrac is a French town with less than 3,000 citizens, and I wanted to see if OpenStreetMap could find it. I also used the 'D' before the name of the town because my file sometimes contained inputs such as 'Ville D'Yvrac' or 'Mairie D'Yvrac' (town of Yvrac/ city hall of Yvrac), and I wanted to check if that would disturb the geolocator.

The output I got was this :

`Location(Yvrac, N 89, Les Tabernottes, Yvrac, Bordeaux, Gironde, Nouvelle-Aquitaine, France métropolitaine, 33370, France, (44.8696002, -0.4663823, 0.0))`

```
lat = reverse_location.latitude
lon = reverse_location.longitude
adresse_test = geolocator.geocode(location)[0]

print(adresse_test, lat, lon)
```

`Mairie, Rue Principale de Cassaber, Cassaber, Carresse-Cassaber, Oloron-Sainte-Marie, Pyrénées-Atlantiques, Nouvelle-Aquitaine, France métropolitaine, 64270, France 43.491092800000004 -1.0111930352817071`

Here I wanted to play with the different variables I could extract from the function, using reverse_location long/lat, and the address element from the request, which is basically the full address.

### Avoiding some errors

```
def geocode(address):
    url = 'https://nominatim.openstreetmap.org/search'
    params = {'q': address, 'format': 'json', 'limit': 1}
    response = requests.get(url, params=params)
    if response.status_code == 200:
        data = response.json()
        if len(data) > 0:
            return data[0]
    return None
```
Here's a block of code that has been added to deal with the cases where nothing was found on OpenStreetMap.

We are good to go!

### Adding the data

```
import pandas as pd
import requests
import time

df = pd.read_excel('file.xlsx')
```

### Initialize empty lists for the results
```
addresses = []
lats = []
lons = []
```
We read an Excel file into a pandas DataFrame and initialize empty lists to store the results. 

### Loop through the cities
```
for city in df['Commune']:
    address = city + ', France'
    result = geocode(address)
    if result is not None:
        addresses.append(result['display_name'])
        lats.append(result['lat'])
        lons.append(result['lon'])
    else:
        addresses.append(None)
        lats.append(None)
        lons.append(None)
    time.sleep(1)
```

We then loop through the cities listed in the DataFrame (the column in the file is called "Commune") and use the geocode() function to retrieve the latitude, longitude, and display name of each location. We append the results to the appropriate list while making sure to avoid hitting the rate limit by using the time.sleep() function to pause the script for one second after each request.

### Add the results to the dataframe
```
df['Adresse postale'] = addresses
df['Latitude'] = lats
df['Longitude'] = lons
```

### Filter out rows where any of the three columns are empty
```
df_filtered = df.dropna(subset=['Adresse postale', 'Latitude', 'Longitude'])
```

Finally, we add the results to the DataFrame, filter out any rows where the three columns (Adresse postale, Latitude, and Longitude) are empty, and save the filtered DataFrame to Excel.

In this case, I wanted to add the new columns to my Excel file, but you could simply create a new file if you wish to.

## Final thoughts
Overall, this code is incredibly useful for a variety of applications beyond just comparing address files. You could use it to generate maps or location-based analytics, or even to plot out a road trip.

So there you have it folks, a nifty little code that can help make your data analysis projects a breeze. As always, happy coding!