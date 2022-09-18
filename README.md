# Python_Project3

1. Importing the Data

New York City skyline 
Welcome to New York City (NYC), one of the most-visited cities in the world. As a result, there are many Airbnb listings to meet the high demand for temporary lodging for anywhere between a few nights to many months. In this notebook, we will take a look at the NYC Airbnb market by combining data from multiple file types like .csv, .tsv, and .xlsx.



We will be working with three datasets:

"datasets/airbnb_price.csv"

"datasets/airbnb_room_type.xlsx"

"datasets/airbnb_last_review.tsv"

</ol>


Our goals are to convert untidy data into appropriate formats to analyze, and answer key questions including:

What is the average price, per night, of an Airbnb listing in NYC?
How does the average price of an Airbnb listing, per month, compare to the private rental market?
How many adverts are for private rooms?
How do Airbnb listing prices compare across the five NYC boroughs?
In [62]:
import pandas as pd
import numpy as np
import datetime as dt

# Load airbnb_price.csv, prices
prices = pd.read_csv("datasets/airbnb_price.csv")

# Load airbnb_room_type.xlsx, xls
xls = pd.ExcelFile("datasets/airbnb_room_type.xlsx")

# Parse the first sheet from xls, room_types
room_types = xls.parse(0)

# Load airbnb_last_review.tsv, reviews
reviews = pd.read_csv("datasets/airbnb_last_review.tsv", sep="\t")

# Print the first five rows of each DataFrame
print(
    f"prices: {prices.head()}",
    "\n",
    f"room_types: {room_types.head()}",
    "\n",
    f"reviews: {reviews.head()}"
)
prices:    listing_id        price                nbhood_full
0        2595  225 dollars         Manhattan, Midtown
1        3831   89 dollars     Brooklyn, Clinton Hill
2        5099  200 dollars     Manhattan, Murray Hill
3        5178   79 dollars  Manhattan, Hell's Kitchen
4        5238  150 dollars       Manhattan, Chinatown 
 room_types:    listing_id                                description        room_type
0        2595                      Skylit Midtown Castle  Entire home/apt
1        3831            Cozy Entire Floor of Brownstone  Entire home/apt
2        5099  Large Cozy 1 BR Apartment In Midtown East  Entire home/apt
3        5178            Large Furnished Room Near B'way     private room
4        5238         Cute & Cozy Lower East Side 1 bdrm  Entire home/apt 
 reviews:    listing_id    host_name   last_review
0        2595     Jennifer   May 21 2019
1        3831  LisaRoxanne  July 05 2019
2        5099        Chris  June 22 2019
3        5178     Shunichi  June 24 2019
4        5238          Ben  June 09 2019
2. Cleaning the price column

Now the DataFrames have been loaded, the first step is to calculate the average price per listing by room_type.

You may have noticed that the price column in the prices DataFrame currently states each value as a string with the currency (dollars) following, i.e.,

price
225 dollars
89 dollars
200 dollars
We will need to clean the column in order to calculate the average price.

In [64]:
# Remove whitespace and string characters from prices column
prices["price"] = prices["price"].str.replace(" dollars", "")

# Convert prices column to numeric datatype
prices["price"] = pd.to_numeric(prices["price"])

# Print 1st 5 rows
print(prices["price"].head())

# Print descriptive statistics for the price column
print(prices["price"].describe())
0    225
1     89
2    200
3     79
4    150
Name: price, dtype: int64
count    25209.000000
mean       141.777936
std        147.349137
min          0.000000
25%         69.000000
50%        105.000000
75%        175.000000
max       7500.000000
Name: price, dtype: float64
3. Calculating average price

We can see three quarters of listings cost $175 per night or less.

However, there are some outliers including a maximum price of $7,500 per night!

Some of listings are actually showing as free. Let's remove these from the DataFrame, and calculate the average price.

In [66]:
prices.info()
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 25209 entries, 0 to 25208
Data columns (total 3 columns):
listing_id     25209 non-null int64
price          25209 non-null int64
nbhood_full    25209 non-null object
dtypes: int64(2), object(1)
memory usage: 590.9+ KB
In [67]:
# Subset prices for listings costing $0 named "free_listings"
free_listings = prices["price"] == 0
print(type(free_listings))
print(free_listings.shape)

# Update prices by removing all free listings from prices
# Similar to SQL's concept of "NOT IN"
prices = prices.loc[~free_listings]

# Calculate the average price and round to nearest 2 decimal places, avg_price
avg_price = round(prices["price"].mean(),2)

# Print the average price
print("The average price per night for an Airbnb listing in NYC is ${}.".format(avg_price))
<class 'pandas.core.series.Series'>
(25209,)
The average price per night for an Airbnb listing in NYC is $141.82.
4. Comparing costs to the private rental market

Now we know how much a listing costs, on average, per night, but it would be useful to have a benchmark for comparison. According to Zumper, a 1 bedroom apartment in New York City costs, on average, $3,100 per month. Let's convert the per night prices of our listings into monthly costs, so we can compare to the private market.

In [69]:
prices.info()
<class 'pandas.core.frame.DataFrame'>
Int64Index: 25202 entries, 0 to 25208
Data columns (total 3 columns):
listing_id     25202 non-null int64
price          25202 non-null int64
nbhood_full    25202 non-null object
dtypes: int64(2), object(1)
memory usage: 787.6+ KB
In [70]:
prices.head()
Out[70]:
listing_id	price	nbhood_full
0	2595	225	Manhattan, Midtown
1	3831	89	Brooklyn, Clinton Hill
2	5099	200	Manhattan, Murray Hill
3	5178	79	Manhattan, Hell's Kitchen
4	5238	150	Manhattan, Chinatown
In [71]:
# Add a new column to the prices DataFrame, price_per_month
prices["price_per_month"] = prices["price"] * 365 / 12
# print(type(prices["price_per_month"]))

# Calculate average_price_per_month
average_price_per_month = round(prices["price_per_month"].mean(),2)                     
                                                       
# Compare Airbnb and rental market
print("Airbnb monthly costs are ${}, while in the private market you would pay {}.".format(average_price_per_month, "$3,100.00"))
Airbnb monthly costs are $4313.61, while in the private market you would pay $3,100.00.
5. Cleaning the room type column

Unsurprisingly, using Airbnb appears to be substantially more expensive than the private rental market. We should, however, consider that these Airbnb listings include single private rooms or even rooms to share, as well as entire homes/apartments. 

Let's dive deeper into the room_type column to find out the breakdown of listings by type of room. The room_type column has several variations for private room listings, specifically:

"Private room"
"private room"
"PRIVATE ROOM"
We can solve this by converting all string characters to lower case (upper case would also work just fine).

In [73]:
room_types.info()
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 25209 entries, 0 to 25208
Data columns (total 3 columns):
listing_id     25209 non-null int64
description    25199 non-null object
room_type      25209 non-null object
dtypes: int64(1), object(2)
memory usage: 590.9+ KB
In [74]:
room_types.head()
Out[74]:
listing_id	description	room_type
0	2595	Skylit Midtown Castle	Entire home/apt
1	3831	Cozy Entire Floor of Brownstone	Entire home/apt
2	5099	Large Cozy 1 BR Apartment In Midtown East	Entire home/apt
3	5178	Large Furnished Room Near B'way	private room
4	5238	Cute & Cozy Lower East Side 1 bdrm	Entire home/apt
In [75]:
# Convert the room_type column to lowercase
room_types["room_type"] = room_types["room_type"].str.lower()

# Update the room_type column to category data type
# https://pandas.pydata.org/docs/user_guide/categorical.html
room_types["room_type"] = room_types["room_type"].astype("category")

# Create the variable room_frequencies
room_frequencies = room_types["room_type"].value_counts()

# Print room_frequencies
print(room_frequencies)
entire home/apt    13266
private room       11356
shared room          587
Name: room_type, dtype: int64
6. What timeframe are we working with?

It seems there is a fairly similar sized market opportunity for both private rooms (45% of listings) and entire homes/apartments (52%) on the Airbnb platform in NYC. 


Now let's turn our attention to the reviews DataFrame. The last_review column contains the date of the last review in the format of "Month Day Year" e.g., May 21 2019. We've been asked to find out the earliest and latest review dates in the DataFrame, and ensure the format allows this analysis to be easily conducted going forwards.

In [77]:
reviews.head()
Out[77]:
listing_id	host_name	last_review
0	2595	Jennifer	May 21 2019
1	3831	LisaRoxanne	July 05 2019
2	5099	Chris	June 22 2019
3	5178	Shunichi	June 24 2019
4	5238	Ben	June 09 2019
In [78]:
reviews.info()
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 25209 entries, 0 to 25208
Data columns (total 3 columns):
listing_id     25209 non-null int64
host_name      25201 non-null object
last_review    25209 non-null object
dtypes: int64(1), object(2)
memory usage: 590.9+ KB
In [79]:
# Change the data type of the last_review column to datetime
# https://pandas.pydata.org/docs/reference/api/pandas.to_datetime.html
reviews["last_review"] = pd.to_datetime(reviews["last_review"])
print(type(reviews["last_review"]))

# Create first_reviewed, the earliest review date
first_reviewed = reviews["last_review"].dt.date.min()

# Create last_reviewed, the most recent review date
last_reviewed = reviews["last_review"].dt.date.max()

# Print the oldest and newest reviews from the DataFrame
print("The latest Airbnb review is {}, the earliest review is {}".format(last_reviewed, first_reviewed))
<class 'pandas.core.series.Series'>
The latest Airbnb review is 2019-07-09, the earliest review is 2019-01-01
In [80]:
print(reviews.dtypes["last_review"])
reviews["last_review"].head()
datetime64[ns]
Out[80]:
0   2019-05-21
1   2019-07-05
2   2019-06-22
3   2019-06-24
4   2019-06-09
Name: last_review, dtype: datetime64[ns]
7. Joining the DataFrames.

Now we've extracted the information needed, we will merge the three DataFrames to make any future analysis easier to conduct. Once we have joined the data, we will remove any observations with missing values and check for duplicates.

In [82]:
# Merge prices and room_types to create rooms_and_prices
# https://pandas.pydata.org/docs/user_guide/merging.html
rooms_and_prices = pd.merge(prices, room_types, 
                            how="outer", 
                            on="listing_id")

# Merge rooms_and_prices with the reviews DataFrame to create airbnb_merged
airbnb_merged = pd.merge(rooms_and_prices, reviews, 
                         how="outer", 
                         on="listing_id")

# Drop missing values from airbnb_merged
airbnb_merged.dropna(inplace=True)

# Check if there are any duplicate values
print("There are {} duplicates in the DataFrame.".format(airbnb_merged.duplicated().sum()))
There are 0 duplicates in the DataFrame.
In [83]:
print(airbnb_merged.info())
airbnb_merged.head()
<class 'pandas.core.frame.DataFrame'>
Int64Index: 25184 entries, 0 to 25201
Data columns (total 8 columns):
listing_id         25184 non-null int64
price              25184 non-null float64
nbhood_full        25184 non-null object
price_per_month    25184 non-null float64
description        25184 non-null object
room_type          25184 non-null category
host_name          25184 non-null object
last_review        25184 non-null datetime64[ns]
dtypes: category(1), datetime64[ns](1), float64(2), int64(1), object(3)
memory usage: 1.6+ MB
None
Out[83]:
listing_id	price	nbhood_full	price_per_month	description	room_type	host_name	last_review
0	2595	225.0	Manhattan, Midtown	6843.750000	Skylit Midtown Castle	entire home/apt	Jennifer	2019-05-21
1	3831	89.0	Brooklyn, Clinton Hill	2707.083333	Cozy Entire Floor of Brownstone	entire home/apt	LisaRoxanne	2019-07-05
2	5099	200.0	Manhattan, Murray Hill	6083.333333	Large Cozy 1 BR Apartment In Midtown East	entire home/apt	Chris	2019-06-22
3	5178	79.0	Manhattan, Hell's Kitchen	2402.916667	Large Furnished Room Near B'way	private room	Shunichi	2019-06-24
4	5238	150.0	Manhattan, Chinatown	4562.500000	Cute & Cozy Lower East Side 1 bdrm	entire home/apt	Ben	2019-06-09
8. Analyzing listing prices by NYC borough

Now we have combined all data into a single DataFrame, we will turn our attention to understanding the difference in listing prices between New York City boroughs. We can currently see boroughs listed as the first part of a string within the nbhood_full column, e.g.,

Manhattan, Midtown
Brooklyn, Clinton Hill
Manhattan, Murray Hill
Manhattan, Hell's Kitchen
Manhattan, Chinatown
We will therefore need to extract this information from the string and store in a new column, borough, for analysis.

In [85]:
# Extract information from the nbhood_full column and store as a new column, borough
# Either use `.str.partition()` or `.str.split()`
airbnb_merged["borough"] = airbnb_merged["nbhood_full"].str.partition(",", expand=True)[0]

# Group by borough and calculate summary statistics
boroughs = airbnb_merged.groupby("borough")["price"].agg(["sum", "mean", "median", "count"])

# Round boroughs to 2 decimal places, and sort by mean in descending order
boroughs = boroughs.round(2).sort_values("mean", ascending=False)

# Print boroughs
print(boroughs)
                     sum    mean  median  count
borough                                        
Manhattan      1898417.0  184.04   149.0  10315
Brooklyn       1275250.0  122.02    95.0  10451
Queens          320715.0   92.83    70.0   3455
Staten Island    22974.0   86.04    71.0    267
Bronx            55156.0   79.25    65.0    696
9. Price range by borough

The above output gives us a summary of prices for listings across the 5 boroughs. In this final task we would like to categorize listings based on whether they fall into specific price ranges, and view this by borough. 

We can do this using percentiles and labels to create a new column, price_range, in the DataFrame. Once we have created the labels, we can then group the data and count frequencies for listings in each price range by borough. 

We will assign the following categories and price ranges:

label	price
Budget	$0-69
Average	$70-175
Expensive	$176-350
Extravagant	> $350
In [87]:
# Create labels for the price range, label_names
label_names = ["Budget", "Average", "Expensive", "Extravagant"]

# Create the label ranges, ranges
ranges = [0, 69, 175, 350, np.inf]

# Insert new column, price_range, into DataFrame
# Use `pd.cut` to segment and sort data values into bins
# Useful for going from a continuous variable to a categorical variable 
airbnb_merged["price_range"] = pd.cut(airbnb_merged["price"], bins=ranges, labels=label_names)

# Calculate borough and price_range frequencies, prices_by_borough
prices_by_borough = airbnb_merged.groupby(["borough", "price_range"])["price_range"].agg("count")
print(prices_by_borough)
borough        price_range
Bronx          Budget          381
               Average         285
               Expensive        25
               Extravagant       5
Brooklyn       Budget         3194
               Average        5532
               Expensive      1466
               Extravagant     259
Manhattan      Budget         1148
               Average        5285
               Expensive      3072
               Extravagant     810
Queens         Budget         1631
               Average        1505
               Expensive       291
               Extravagant      28
Staten Island  Budget          124
               Average         123
               Expensive        20
Name: price_range, dtype: int64
In [88]:
print(ranges)
print(type(ranges))
[0, 69, 175, 350, inf]
<class 'list'>
