# Using ML To Detect Bias in Criminal Sentencing Data
An R script that uses machine learning via random forests to examine [Cook County, IL criminal sentencing data](https://datacatalog.cookcountyil.gov/Courts/Sentencing/tg8v-tm6u) for human decision bias.

## Accessing the Data
The data has been made available on the Github repo as a zipped CSV file. However, the easiest way to access the data is to query the Socratic Open Data API (SODA API) using the [R package](https://cran.r-project.org/web/packages/RSocrata/RSocrata.pdf) `RSocrata` (developed by Hugh Devlin, Ph. D., Tom Schenk, Jr., Gene Leynes, Nick Lucius, John Malc, Mark Silver- berg, and Peter Schmeideskamp). See the [website](https://dev.socrata.com/consumers/getting-started.html) for more detail. 

To query the data, first install the `RSocrata` R package:

```
install.packages("RSocrata")
```

Once installed, load the `RSocrata` package:

```
library(RSocrata)
```

Navigate to the Cook County Open Data [website](https://datacatalog.cookcountyil.gov/Courts/Sentencing/tg8v-tm6u) and under the "API" tab beside the API Endpoint URI box is the word "JSON." Click on "JSON" and an option beneath it for "CSV" will appear. Choose the CSV option and then copy the URI, which should be <https://datacatalog.cookcountyil.gov/resource/tg8v-tm6u.csv>. 

Then assign the copied URI to an object named `apiEndpoint`:

```
# Query API and assign to `apiEndpoint`
apiEndpoint = 
  "https://datacatalog.cookcountyil.gov/resource/tg8v-tm6u.csv"
```

After assigning the endpoint URI to the object, use the `read.socrata()` function to query the data from SODA API, assigning the returned data to a data frame object named `sentences`:

```
# Assign the API endpoint URI to `apiEndpoint`
apiEndpoint = 
  "https://datacatalog.cookcountyil.gov/resource/tg8v-tm6u.csv"
 
# Query the SODA API and assign the returned data to data frame object `sentences`
sentences = 
  read.socrata(apiEndpoint)
```

As downloaded from the SODA API, the data set comprises 221,170 observations and 39 features, which each row representing a charge that resulted in decision of 'guilty', , The return data should look similar to the structure of the following output:

```
R> head(sentences)
      case_id case_participant_id   charge_id charge_version_id primary_charge       offense_title       chapter act   section class
1 61601638368         1.13581e+11 82263540230       88097103639          false FIRST DEGREE MURDER            38   - 9-1(a)(2)     X
2 61601638368         1.13581e+11 82273857348       99965682145          false FIRST DEGREE MURDER            38   - 9-1(a)(3)     X
3 61601638368         1.13581e+11 82273939230       99960728707          false FIRST DEGREE MURDER            38   - 9-1(a)(3)     X
4 61601638368         1.13581e+11 82274184876       74277365982          false       HOME INVASION 38-12-11-A(1)                   X
5 61601638368         1.13581e+11 82274184876       74277365982          false       HOME INVASION 38-12-11-A(1)                   X
6 61601638368         1.13581e+11 82263703994       74432974693          false       HOME INVASION 38-12-11-A(2)                   X
        aoic             dispo_date               sentence_phase          sentence_date    sentence_judge sentence_type
1 0000001607 12/17/2014 12:00:00 AM          Original Sentencing   6/2/1986 12:00:00 AM     John  Mannion    Conversion
2 0000001608 12/17/2014 12:00:00 AM          Original Sentencing   6/2/1986 12:00:00 AM     John  Mannion    Conversion
3 0000001608 12/17/2014 12:00:00 AM          Original Sentencing   6/2/1986 12:00:00 AM     John  Mannion    Conversion
4 0000001846 12/17/2014 12:00:00 AM Amended/Corrected Sentencing 10/16/2014 12:00:00 AM Clayton Jay Crane        Prison
5 0000001846 12/17/2014 12:00:00 AM          Original Sentencing   6/2/1986 12:00:00 AM     John  Mannion    Conversion
6 0000001847 12/17/2014 12:00:00 AM          Original Sentencing   6/2/1986 12:00:00 AM     John  Mannion    Conversion
                     commitment_type commitment_term commitment_unit charge_disposition charge_disposition_reason
1                       Natural Life            <NA>                    Nolle On Remand                          
2                       Natural Life            <NA>                    Nolle On Remand                          
3                       Natural Life            <NA>                    Nolle On Remand                          
4 Illinois Department of Corrections              30         Year(s)     Plea Of Guilty                          
5 Illinois Department of Corrections              30         Year(s)     Plea Of Guilty                          
6 Illinois Department of Corrections              30         Year(s)    Nolle On Remand                          
            court_name     court_facility length_of_case_in_days age_at_incident gender  race      offense_type  incident_begin_date
1 District 6 - Markham Markham Courthouse                    619              27   Male Black PROMIS Conversion 8/9/1984 12:00:00 AM
2 District 6 - Markham Markham Courthouse                    619              27   Male Black PROMIS Conversion 8/9/1984 12:00:00 AM
3 District 6 - Markham Markham Courthouse                    619              27   Male Black PROMIS Conversion 8/9/1984 12:00:00 AM
4 District 6 - Markham Markham Courthouse                  10982              27   Male Black PROMIS Conversion 8/9/1984 12:00:00 AM
5 District 6 - Markham Markham Courthouse                    619              27   Male Black PROMIS Conversion 8/9/1984 12:00:00 AM
6 District 6 - Markham Markham Courthouse                    619              27   Male Black PROMIS Conversion 8/9/1984 12:00:00 AM
  incident_end_date           arrest_date law_enforcement_agency unit incident_city         received_date      arraignment_date
1                   8/15/1984 12:00:00 AM    CHICAGO POLICE DEPT <NA>          <NA> 8/15/1984 12:00:00 AM 9/21/1984 12:00:00 AM
2                   8/15/1984 12:00:00 AM    CHICAGO POLICE DEPT <NA>          <NA> 8/15/1984 12:00:00 AM 9/21/1984 12:00:00 AM
3                   8/15/1984 12:00:00 AM    CHICAGO POLICE DEPT <NA>          <NA> 8/15/1984 12:00:00 AM 9/21/1984 12:00:00 AM
4                   8/15/1984 12:00:00 AM    CHICAGO POLICE DEPT <NA>          <NA> 8/15/1984 12:00:00 AM 9/21/1984 12:00:00 AM
5                   8/15/1984 12:00:00 AM    CHICAGO POLICE DEPT <NA>          <NA> 8/15/1984 12:00:00 AM 9/21/1984 12:00:00 AM
6                   8/15/1984 12:00:00 AM    CHICAGO POLICE DEPT <NA>          <NA> 8/15/1984 12:00:00 AM 9/21/1984 12:00:00 AM
  current_sentence updated_offense_category charge_count
1             true                 Homicide            2
2             true                 Homicide            4
3             true                 Homicide            5
4             true                 Homicide           13
5            false                 Homicide           13
6             true                 Homicide           14
```
### Description of the Variables

Columns in the dataset include the following:

`CASE_ID`: Internal unique identifier for each case (Number)

`CASE_PARTICIPANT_ID`:Internal unique identifier for each person associated with a case (Number)

`CHARGE_ID`: Internal unique identifier for each charge filed (Number)

`CHARGE_VERSION_ID`: Internal unique identifier for each version of a charge associated with charges filed (Number)

`PRIMARY_CHARGE`: A flag for the top charge, usually the way the case is referred to (Checkbox)

`OFFENSE_TITLE`: The specific title of the charge offense (Plain Text)

`CHAPTER`: The legal chapter for the charge (Plain Text)

`ACT`: The legal act for the charge (Plain Text)

`SECTION`: The legal section for the charge (Plain Text)

`CLASS`: The legal class of the charge (Plain Text)

`AOIC`:	Administrative Office of the Illinois Courts ID for law of the charge (Plain Text)

`DISPO_DATE`: The date the charge was disposed of (Plain Text)

`SENTENCE_PHASE`: Sentencing phase explains when this version of the sentence was created (Plain Text)

`SENTENCE_DATE`: Date of when the charge was sentenced (Date "m-d-y h:mm")

`SENTENCE_JUDGE`: Judge who oversaw the sentencing (Plain Text)

`SENTENCE_TYPE`: A broad type of sentence issued (Plain Text)

`COMMITMENT_TYPE`: A more specific type of sentence issued (Plain Text)

`COMMITMENT_TERM`: The number associated with the sentence (use this with commitment_unit to understand length of sentence) (Plain Text)

`COMMITMENT_UNIT`: The unit associated with the sentence (use this with commitment_term to understand length of sentence) (Plain Text)

`CHARGE_DISPOSITION` The result of the charge (Plain Text)

`CHARGE_DISPOSITION_REASON`: Additional information about the result of the charge (Plain Text)

`COURT_NAME` The Circuit Court District the sentence was determined in (Plain Text)

`COURT_FACILITY`: The courthouse the sentence was determined in (Plain Text)

`LENGTH_OF_CASE_in_Days`: Number of days between a charge being arraigned and a charge being sentenced (Numeric)

`AGE_AT_INCIDENT`: Recorded age at the time of the incident (Integer)

`GENDER`: Recorded gender of the defendant (Factor)

`RACE`:	Recorded race of the defendant (Factor)

`OFFENSE_TYPE`: Broad offense type for charges filed in plain language (Text)

`INCIDENT_BEGIN_DATE`: Date of when the incident began (Date object "m-d-y h:mm")

`INCIDENT_END_DATE`: Date of when the incident ended (blank for incidents shorter than one day) (Date object "m-d-y h:mm")

`ARREST_DATE`: Date and time of arrest (Date-time object "m-d-y h:mm")

`LAW_ENFORCEMENT_AGENCY`: Law enforcement agency associated with the arrest (Plain Text)

`UNIT`: The law enforcement unit associated with the arrest (Plain Text)

`INCIDENT_CITY`: The city where the incident took place (Factor)

`RECEIVED_DATE`: Date when felony review received the case (Date object "m-d-y h:mm")

`ARRAIGNMENT_DATE`: Date of the arraignment (Date object "m-d-y h:mm")

`CURRENT_SENTENCE`: This is a flag which row represents a current sentence. (Checkbox)

`UPDATED_OFFENSE_CATEGORY`: This field is the offense category for the case updated based upon the top charge for the primary offender. It can differ from the first offense category assigned to the case in part because cases evolve (Plain Text)

`CHARGE_COUNT`: The charge count of the charged offense (Number)

It should be noted that the variable types listed above are not necessarily the state of the variables throughout the analysis to be performed in R e.g. (dates are transformed from "m-d-y" to "YYYY-MM-DD" ("2019-01-01") in the R script and excludes references to hours and seconds since that time granularity was not included in the data set. "Checkboxes" are treated as Boolean variables that have values of "TRUE" or "FALSE."
