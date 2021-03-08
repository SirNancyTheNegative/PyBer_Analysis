# PyBer's Ride-Sharing

## Overview of Project

### Purpose

Pyber has been occupied with determining the effectiveness of their ride-sharing service in various areas, and have tasked us with pouring over the data accumulated in order to determine how successful they are in rural, suburban, and urban cities both in general, and on a week-by-week basis between January 1st and April 29th of 2019.

## Results

### Analysis of Average Fares

The first task ahead of us was to determine the average fares for each city type, both by rides and by driver. In order to do so, we need to find the total fares by city type, the number of rides, and the number of drivers. Our data is separated into two DataFrames: one containing the cities, number of drivers, and city type, and another containing all the ride data, including the city it happened in, the fare for each ride, and the ride ID. If we merge the DataFrames by city, we can have a singular DataFrame with which we can get each value according to city type.

For the total fare, we can sum each fare that belongs to each city type, after grouping the DataFrame by "type", as such:
```
# Get the total amount of fares by city type
pyber_fares = pyber_data_df.groupby(["type"])["fare"].sum()
```

The number of rides is similar; since each row in our data details the information for a ride, we can simply instead count the number of rows that belong to each "type":
```
# Get the total rides for each city type
pyber_ride_counts = pyber_data_df.groupby(["type"])["ride_id"].count()
```

For the number of drivers, however, we need to take a somewhat different approach. If we simply take the values from our merged DataFrame, we'll get [the following incorrect summary DataFrame.](https://github.com/SirNancyTheNegative/PyBer_Analysis/tree/main/Analysis/PyBer_Ave_summary.png)

The issue here is the number of drivers: While it could be expected that a ride-share service might have a few more drivers than rides on some days, it doesn't make any sense for there to be such a large amount of unused drivers, especially in urban areas. So, to fix this, instead of making use of the merged DataFrame, we can instead look at the city DataFrame in order to get a more accurate number of drivers.
```
# Get the total drivers for each city type
pyber_driver_counts = city_data_df.groupby(["type"])["driver_count"].sum()
```

From here, the resulting DataFrame can be done through mere arithmetic. Because pandas DataFrames can divide rows based on data type with a simple arithmetic operator, fares per ride and fares per driver are as simple as dividing pyber_fares by pyber_ride_counts and pyber_fares by pyber_driver_counts, respectively.

![The following DataFrame is produced once all pertinent data is assembled.](https://github.com/SirNancyTheNegative/PyBer_Analysis/tree/main/Analysis/PyBer_RealAve_summary.png)

From this we can glean a large number of facts: In terms of the number of rides, number of drivers and gross fares, the urban cities remain on top, and the rural areas produce very little revenue when viewed in this light, with suburban areas proving to be the middle of the road. We can also gather that although the urban cities have the highest gross fares, the number of rides and the number of drivers increase faster than the gross fares, causing the smallest fares per rides and by far the smallest fares per drivers. On the other hand, though rural areas have the lowest gross fares, their fares per ride and fares per driver are by far the highest of the three. Once more, suburban areas prove to be the middle ground, if not slightly skewed towards rural areas in that respect.

### Analysis of Total Fares by Week

In order to get the total fares of each city type by week, the entire format of the DataFrame needed to be changed. In order to do this, multiple steps needed to be taken.

The first was to group the data by city type and by date:
```
#  Using groupby() to create a new DataFrame showing the sum of the fares 
#  for each date where the indices are the city type and date.
pycd_fares = pyber_data_df.groupby(["type","date"]).sum()
```

By doing this, we set the index to be equal to the combination of city type and date, so all pertinent information is separated into bins based on the city type and then sorted by date.

The next step was to reorient the data such that the fares in each row served to be the new values for matching sets of city types and dates. However, this is rather messy when the index consists of both the new index we want and the columns we need. So, the next step consists of both fixing this step and pivoting the DataFrame:
```
# Reset the index on the DataFrame you created. This is needed to use the 'pivot()' function.
# df = df.reset_index()
pycd_fares = pycd_fares.reset_index()

# Create a pivot table with the 'date' as the index, the columns ='type', and values='fare' 
# to get the total fares for each type of city by the date. 
py_fares = pycd_fares.pivot(index="date",columns="type",values="fare")
```

From this, we get [the following DataFrame.](https://github.com/SirNancyTheNegative/PyBer_Analysis/tree/main/Analysis/PyBer_Weekly_Pivot.png) 

As can be seen, there are numerous NaN values in this DataFrame. Ordinarily, we would clean the data of these values, but considering we only care about the values that do exist, and the methods we take will ignore NaN values, we can simply move on.

Since we only care about the dates between January 1st and April 29th, we can separate the rows between those dates and put them into a new DataFrame:
```
# Create a new DataFrame from the pivot table DataFrame using loc on the given dates, '2019-01-01':'2019-04-29'.
pyfare_df = py_fares.loc['2019-01-01':'2019-04-29']
```

Now that we have the range we want, we need to be able to separate the data into weeks. There's a built-in function called resample() that will accept a signifier as to decide how to re-label the index, but it requires a DatetimeIndex datatype, and our current index isn't that. So first, we have to change the dates we have to the DatetimeIndex datatype before we can group them into weeks and sum them:
```
# Set the "date" index to datetime datatype. This is necessary to use the resample() method in Step 8.
# df.index = pd.to_datetime(df.index)
pyfare_df.index = pd.to_datetime(pyfare_df.index)

# Check that the datatype for the index is datetime using df.info()
pyfare_df.info()

# Create a new DataFrame using the "resample()" function by week 'W' and get the sum of the fares for each week.
pyfare_weekly = pyfare_df.resample('W').sum()
```

What this gives us is a [DataFrame where the indices are the end of a week](https://github.com/SirNancyTheNegative/PyBer_Analysis/tree/main/Analysis/PyBer_Resampled.png), and the values for each city type are the sums of the fares from each week. This gives us what we need to plot each on a graph. We can do so in an object-oriented fashion:
```
# 8. Using the object-oriented interface method, plot the resample DataFrame using the df.plot() function. 
# Import the style from Matplotlib.
from matplotlib import style
# Use the graph style fivethirtyeight.
style.use('fivethirtyeight')
pyfare_weekly.plot(title = "Total Fare by City Type",xlabel = "",ylabel = "Fare ($USD)",figsize=(12,4))
plt.tight_layout()
plt.savefig("Analysis/PyBer_fare_summary.png")
```

![This is the resulting graph.](https://github.com/SirNancyTheNegative/PyBer_Analysis/tree/main/Analysis/PyBer_fare_summary.png)

As could probably be expected, week by week, suburban and rural fares are consistently less urban fares. A large portion of this consistency is due to population density; because urban sprawls are usually heavily condensed, the number of potential customers drastically increases, and the fact that parking spaces are often much more contested in urban spaces than suburban or rural ones means that the prospect of having a car drop you off and leave without you needing to worry about parking and other such fees is a tempting prospect. On the other side of matters, more rural areas have much more sparse populations, owing much to agricultural development for their formation, and the need to travel potentially long distances to reach other people necessitates the use of a car, oftentimes a truck if said traveler happens to need to transport large objects or large quantities of objects. As such, folks from rural towns often have no need for ride-sharing services, especially if they happen to live in a part of the world where the gasoline used is much cheaper than the fare would be. As seen before with the fares per rides and fares per density, suburban areas are more middle-of-the-road. On account of higher population densities than in rural areas, the amount of revenue that could be made in a week is understandably higher, and according to the graph, this city type seems to be experiencing a faster boon of growth than that of the urban areas, as of April 29th, 2019.

## Summary

### Recommendations

From this data we can see that suburban areas seem to be a best fit for potential growth. There are still more rides given than drivers to give them, too, so the average fare per driver remains higher than the average fare per ride, and besides that, the market in those suburban areas seems to be growing faster than that within urban areas, so the first recommendation we could give is to promote development of ride-sharing services in suburban areas. Inversely, there remain more drivers than rides given in a given time frame in urban areas, causing each driver to accumulate fewer fares on average. As populations increase, it is likely that fares per driver will increase if no new drivers are added, so to spur on that potential growth the second recommendation is to increase the advertising budget for urban areas, but refrain from increasing the number of drivers the same for those areas until such a time where the average fares per driver pass the average fares per ride.

Unlike suburban and urban areas, rural areas are a bit more tricky. As mentioned previously, most of rural travel is done via automobile, but there's not as much impetus to turn to ride-sharing to circumvent some of the costs that exist in an urban setting. Growth additionally doesn't appear to be very constant, as the fares by week of rural areas wavered between $0 and $500 per week for the whole of the period, peaking at the beginning of April but quickly falling back down to the prices in January. As such, the third recommendation would be to observe rural areas further. If the market doesn't seem to improve, or if the revenue doesn't seem like it would exceed operating costs, there may be need to back out of rural areas to focus more on suburban and urban areas.
