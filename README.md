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

The creators of the Covid data use American style date format mm/dd/yyyy
instead of the more typical dd/mm/yyyy as used in Thailand. Weird.

When plotting the data, the pandas plot methods sometimes put too many
tick labels on the x-axis and display the full datetime object as "2021-01-04 00:00:00". 

To remove the time portion, specify a date formatter:
```python
ax.xaxis.set_major_formatter(matplotlib.dates.DateFormatter("%F"))
```
where `ax` is a reference to the plot.  There is also a `set_minor_formatter` method.


Ref: <https://izziswift.com/pandas-timeseries-plot-setting-x-axis-major-and-minor-ticks-and-labels/>
I didn't do
I'm interested in recent data, so keep only 
