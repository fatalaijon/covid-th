## Thailand Covid Data


### Source of Data

The Thailand Department of Disease Control makes data available through
a free API.
* Home page <https://covid19.th-stat.com/>
* API reference <https://covid19.th-stat.com/en/api> (for Thai use "th" instead of "en">

The URLs are:

* <https://covid19.th-stat.com/api/open/timeline> daily reports since 1/1/2020, including daily and cumulative values for: confirmed, recovered, hospitalized, and died. JSON format
* <https://covid19.th-stat.com/api/open/today> today's report, same format as the timeline (above).
* <https://covid19.th-stat.com/api/open/cases> data for each case, including gener, nationality, province, district, date confirmed, and quarantine date (if any).  Reverse chronological order. Dataset is **large** due to redundant encoding of both Thai and English for fields and use of full location names instead of codes. JSON format.
* <https://covid19.th-stat.com/api/open/area> a single JSON record containing the following (including typo and unnecessary backslashes):

```json
{"Data": [],
 "Source": "https:\/\/ddc.moph.go.th\/viralpneumonia\/",
 "DevBy": "https:\/\/www.kidkarnmai.com\/",
 "SeverBy": "https:\/\/smilehost.asia\/"
}
```

The `timeline` file for 1 May 2021 contains the following, reformatted for readability.
The actual file is all one line with no space between fields.
```json
{"UpdateDate": "01\/05\/2021 11:42",
 "Source": "https:\/\/covid19.th-stat.com\/",
 "DevBy": "https:\/\/www.kidkarnmai.com\/",
 "SeverBy": "https:\/\/smilehost.asia\/",
 "Data": [
    ...
 {"Date": "05\/01\/2021",
  "NewConfirmed": 1891,
  "NewRecovered": 1821,
  "NewHospitalized": 49,
  "NewDeaths": 21,
  "Confirmed": 67044,
  "Recovered": 38075,
  "Hospitalized": 28745,
  "Deaths": 224}
 ]
}
```

Another source of Covid data for Thailand is [Our World in Data](https://ourworldindata.org/coronavirus/).  The URL for Thai data is <https://ourworldindata.org/coronavirus/country/thailand>, but I don't see how to download for only a specific country.

### Pandas Syntax Used

The daily Covid data is inside a JSON array with key 'Data'.  We want to use that to create the Panda DataFrame (rather than everything in the timeline file).

1. Read the file and create a JSON object:
   ```python
   import json

   with open("data/timeline.json","r") as f:
        all_data = json.load(f)
   ```
2. Create a DataFrame object from records in the 'Data' element of the JSON object:
   ```python
   import pandas as pd

   covid = pd.DataFrame.from_records(all_data['Data'])
   ```

The columns (fields) in this dataframe are:
```
Date             60 non-null     object
NewConfirmed     60 non-null     int64         
NewRecovered     60 non-null     int64         
NewHospitalized  60 non-null     int64         
NewDeaths        60 non-null     int64         
Confirmed        60 non-null     int64         
Recovered        60 non-null     int64         
Hospitalized     60 non-null     int64         
Deaths           60 non-null     int64 
```

The `Date` field is just a string. Convert it to a timestamp using:
```
covid['date'] = pd.to_datetime(covid['Date'])
```
this creates a new column named 'date'. You could simply replace values
in the existing 'Date' column, but I kept it.

### Transforming data

The Covid data contains a field (column) of Dates, as strings.
We transform them to Timestamps using:
```python
covid['Date'] = pd.to_datetime(covid['Date'])
```

but the observations have only a date value, the time is always 0:00:00.

Pandas Series class has a [transform](https://pandas.pydata.org/docs/reference/api/pandas.Series.transform.html) method with syntax:
```python
result: Series = series.transform(func, axis=0, *args, **kwargs)
```
where `func` is a function to apply to each element in the series.  `*args` and `**kwargs` are passed to the `func`.

To transform Timestamp to datetime.date use:
```python
dates = covid['Date'].transform(pd.Timestamp.date)
```
instead of saving the result as separate Series object, we save it as a new column in the `covid` DataFrame.

## Formatting Dates and Tick Mark Spacing

The creators of the Thai Covid data use American style date format mm/dd/yyyy
instead of the more typical dd/mm/yyyy as used in Thailand. Weird. Pandas seems to detect this and convert dates correctly.

When plotting the data, the pandas plot methods put too many tick labels
on the x-axis and display the full datetime object as "2021-01-04 00:00:00". 

Fixes are:
* convert Timestamp to date
* use a DateFormatter
* specify frequency of major tick labels


Good reference is part of a free course:
<https://www.earthdatascience.org/courses/use-data-open-source-python/use-time-series-data-in-python/date-time-types-in-pandas-python/customize-dates-matplotlib-plots-python/>

The two methods used are:
```python
fig, ax = plt.subplots(...)

# another good format is "%F" (yyyy-mm-dd)
date_format = matplotlib.dates.DateFormatter("%m-%d")
ax.xaxis.set_major_formatter( date_formatter )

# interval=1 means 1 tick label each week
ax.xaxis.set_major_locator(matplotlib.dates.WeekdayLocator(interval=1)
```

where `ax` is a reference to the plot.  There is also a `set_minor_formatter` method.


Another, older reference: <https://izziswift.com/pandas-timeseries-plot-setting-x-axis-major-and-minor-ticks-and-labels/>
