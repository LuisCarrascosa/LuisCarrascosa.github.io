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
df.head(3)
```
![Image](images/fig1.jpg)
