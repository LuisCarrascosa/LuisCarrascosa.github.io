# Analysing Madrid Airbnb data using Plotly, Mapbox and Geopandas
![Image](images/airbnb-2384737_1280.jpg)

## Introduction
Airbnb is the world’s biggest accommodation-sharing site. Its rapid growth and impact on vacation rentals has generated heated discussions about its effect in several of the world's largest cities. So much that regulations have emerged against the free use of this website:

* Amsterdam: the rental of complete houses is limited to 60 days a year and this will be reduced by half
* Barcelona: short-term rentals must have a license and new licenses are not being issued
* Berlin: owners need a permission to rent 50% or more of their main residence for a short period of time
* London: short-term rentals of entire houses are restricted to 90 days a year
* Palma: the mayor has announced the ban on short-term rentals
* New York: It is usually illegal to rent apartments for 30 consecutive days or less, unless the host is present
* Paris: short-term rentals are limited to 120 days a year
* San Francisco: hosts must register as a company and obtain certificates for short-term rental. The rental of entire properties is limited to 90 days a year
* Singapore: the minimum period of public housing rental is six consecutive months
* Tokyo: home sharing was legalized only in 2017 and is limited to 180 days a year
<div style="text-align: right"><i>Source: https://www.bbc.com/mundo/noticias-45355426</i></div>
Given the [Airbnb dataset of Madrid](http://insideairbnb.com/get-the-data.html), we propose to obtain the following information:
## Objectives
The data will be processed to clean them and get as much information as possible. A statistical analysis will attempt to solve the following questions:
* Which areas have the most Airbnb properties, and which are the most expensive?
* Which amenities increase the price of an Airbnb listing?
* Influence of host features in the median price

## Processing the data
![Image](images/blur-business-coffee-commerce-273222.jpg)

### Import the data
The dataset used for this project comes from Insideairbnb.com. The dataset was scraped on 2019-11-09 and contains information on all Madrid Airbnb listings that were live on the site on that date (20.539)

I will not import free text fields and I will remove the currency symbol from fields with amounts. "smart_location", "zipcode" are redundant having latitude and longitude. "reviews_per_month", "number_of_reviews_ltm" with "number_of_reviews" too. url fields does not add value to the model. I don't import "host_name", "host_location" and "host_about" too.

```python
datasetDir = "/home/luis/datasets/MadridAirbnbData"
locale.setlocale(locale.LC_ALL, 'en_US.UTF8') 
conv = locale.localeconv()
fconv_prices = lambda x : locale.atof(x.strip(conv['currency_symbol'])) if x else np.nan 

def fdate_parser(x, dtime_format = "%d-%m-%Y"):
    return datetime.strptime(x, dtime_format) 
    
df = pd.read_csv(
    f'{datasetDir}/listings_detailed.csv', sep = ',', index_col = 'id', 
    converters = {
        'price': fconv_prices, 'weekly_price': fconv_prices, 'monthly_price': fconv_prices, 'security_deposit': fconv_prices,
        'cleaning_fee': fconv_prices, 'extra_people': fconv_prices
    },
    parse_dates = ['first_review', 'last_review', 'host_since'],
    usecols = lambda column : column not in [
        'zipcode', 'scrape_id', 'last_scraped', 'name', 'summary', 'space', 'description', 
        'neighborhood_overview', 'notes', 'transit', 'access', 'interaction', 'house_rules', 
        'thumbnail_url', 'medium_url', 'picture_url', 'xl_picture_url', 'host_url', 'host_name', 
        'host_location', 'host_about', 'host_thumbnail_url', 'host_picture_url', 'host_neighbourhood', 
        'host_verifications', 'calendar_last_scraped', 'listing_url', 'smart_location',
        'reviews_per_month', 'number_of_reviews_ltm', 'host_id'
    ]
)
```
<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>experiences_offered</th>
      <th>host_since</th>
      <th>host_response_time</th>
      <th>host_response_rate</th>
      <th>host_acceptance_rate</th>
      <th>host_is_superhost</th>
      <th>host_listings_count</th>
      <th>host_total_listings_count</th>
      <th>host_has_profile_pic</th>
      <th>host_identity_verified</th>
      <th>...</th>
      <th>jurisdiction_names</th>
      <th>instant_bookable</th>
      <th>is_business_travel_ready</th>
      <th>cancellation_policy</th>
      <th>require_guest_profile_picture</th>
      <th>require_guest_phone_verification</th>
      <th>calculated_host_listings_count</th>
      <th>calculated_host_listings_count_entire_homes</th>
      <th>calculated_host_listings_count_private_rooms</th>
      <th>calculated_host_listings_count_shared_rooms</th>
    </tr>
    <tr>
      <th>id</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>6369</th>
      <td>none</td>
      <td>2009-04-16</td>
      <td>within a few hours</td>
      <td>100%</td>
      <td>NaN</td>
      <td>f</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>t</td>
      <td>f</td>
      <td>...</td>
      <td>NaN</td>
      <td>f</td>
      <td>f</td>
      <td>flexible</td>
      <td>f</td>
      <td>f</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>21853</th>
      <td>none</td>
      <td>2010-02-21</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>f</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>t</td>
      <td>t</td>
      <td>...</td>
      <td>NaN</td>
      <td>f</td>
      <td>f</td>
      <td>strict_14_with_grace_period</td>
      <td>f</td>
      <td>f</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>23001</th>
      <td>none</td>
      <td>2010-02-17</td>
      <td>within an hour</td>
      <td>100%</td>
      <td>NaN</td>
      <td>f</td>
      <td>9.0</td>
      <td>9.0</td>
      <td>t</td>
      <td>f</td>
      <td>...</td>
      <td>NaN</td>
      <td>f</td>
      <td>f</td>
      <td>moderate</td>
      <td>f</td>
      <td>f</td>
      <td>5</td>
      <td>5</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>3 rows × 74 columns</p>
</div>



### Cleaning the features

I will drop columns with more than 85% nulls


```python
cols_to_drop = set(df.columns[df.isna().mean()>0.85])
print(cols_to_drop)
```

    {'host_acceptance_rate', 'monthly_price', 'square_feet', 'weekly_price', 'jurisdiction_names'}



```python
df.drop(cols_to_drop, axis='columns', inplace=True)
```

We only use latitude, longitude and neighbourhood_cleansed location. As we are studying Madrid, the city and country columns do not contribute anything


```python
df_gps = df[['latitude', 'longitude']]
df_gps.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>latitude</th>
      <th>longitude</th>
    </tr>
    <tr>
      <th>id</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>6369</th>
      <td>40.45628</td>
      <td>-3.67763</td>
    </tr>
    <tr>
      <th>21853</th>
      <td>40.40341</td>
      <td>-3.74084</td>
    </tr>
    <tr>
      <th>23001</th>
      <td>40.38695</td>
      <td>-3.69304</td>
    </tr>
    <tr>
      <th>24805</th>
      <td>40.42202</td>
      <td>-3.70395</td>
    </tr>
    <tr>
      <th>24836</th>
      <td>40.41995</td>
      <td>-3.69764</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.drop(
    ['neighbourhood', 'neighbourhood_group_cleansed', 'street', 'city', 'state', 'country', 
     'country_code', 'is_location_exact', 'latitude', 'longitude']
    , axis='columns', inplace=True
)
```