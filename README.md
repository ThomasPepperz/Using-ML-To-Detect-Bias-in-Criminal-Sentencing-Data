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

## Cleaning the Data

### Setup
Begin by first loading (or downloading if not already installed) the required packages for the entire analysis:

```
# Stop R from printing using scientific notation
options(scipen = 999)

# Load packages
library(RSocrata)
library(dplyr)
library(janitor)
library(lubridate)
library(ggplot2)
library(gmodels)
library(forcats)
library(randomForest)
library(party)
library(rpart.plot)
library(reprtree)
```

### Transforming the Variables
Unfortunately, there are no enforced naming conventions for files, objects, functions, and variables for the R language. Google has published ["Google's R Style Guide"](https://google.github.io/styleguide/Rguide.xml), which encourages user's to use lower-cased names separated by periods for variable names and is the convention for variable-naming that is followed in the following analysis. Statistician Hadley Wickham, the Chief Scientist at RStudio as well as author and maintainer of popular R packages that include `tidyverse`, `tidyr`, `ggplot`, and `dplyr` amongst many others, prefers to use the underscore to separate elements within noun objects (data frame and variable names). See Wickham's brief [style guide](http://adv-r.had.co.nz/Style.html) for more details. Furthermore, an interesting study regarding R naming conventions is Rasmus Bååth's article ["The State of Naming Conventions in R"](https://journal.r-project.org/archive/2012-2/RJournal_2012-2_Baaaath.pdf) published in the *The R Journal*, Vol. 4/2, December 2012.

The following code uses the function `clean_names()` from the `janitor` package to format the variable names according to Google's R code style:

```
# Cooerce column names
sentences = 
  sentences %>% 
  clean_names()
# Replace underscore with a period
colnames(sentences) = 
  gsub("_", ".", colnames(sentences))
# Check column name formatting is consistent
colnames(sentences)

R> colnames(sentences)
[1] "case.id"                   "case.participant.id"       "charge.id"                 "charge.version.id"        
[5] "primary.charge"            "offense.title"             "chapter"                   "act"
...
```

The following section focuses on the transformation of the variable data types for each feature as well as the formatting of the associated values. The first variable to be transformed is `race`.

Inspection of the data revealed many inconsistencies in how values for `race` has been inputted, which most likely is attributable either to inconsistent data input formatting policies amongst arresting agencies or to individual officers with the responsibility of inputting data.

```
R> levels(as.factor(sentences$race))
 [1] ""                                 "American Indian"                  "Asian"                           
 [4] "ASIAN"                            "Biracial"                         "Black"                           
 [7] "HISPANIC"                         "Unknown"                          "White"                           
[10] "White [Hispanic or Latino]"       "White/Black [Hispanic or Latino]"
  
```  

Decisions on how to handle the inconsistently named values for `race` require great care and consideration since, afterall, the topic of the analysis is detecting bias (racial, gender, and age-related bias) in criminal sentencing data. It is an easy call to merge records with values "ASIAN" into "Asian"; however, it is much more difficult to justify decisions on how to treat categories such "White [Hispanic or Latino]," "White/Black [Hispanic or Latino]," and "Biracial." Evidence of the confused treatment and understanding of race and ethnicity by the American judicial system is illustrated by the choice of classification values for `race` in the Cook County, Illinois criminal sentencing data set. 

What is the difference between those with a race of "White" versus "White [Hispanic or Latino"? Should "White [Hispnaic or Latino] records be considered "White," "Hispanic," or as its own category? If treated as a separate category from "White" or "Hispanic," then what is the justification for treating it as a separate class?

What is the distinction between "White/Black [Hispanic or Latino] and other related categories? Should the category simply be merged into "Hispanic"? Given the inclusion of "White/Black" in the category, it is unacceptable to merge the category into either "White,", "Black," and "White [Hispanic or Latino]." If not included in "Hispanic," would it be acceptable to treat "White/Black [Hispanic or Latino]" as "Biracial" or is "Biracial" inclusive of only a mixed race consisting of a mixture between "White" and "Black" races?

Decisions regarding the naming of racial groups for census and statistical purposes reveals the racial tension and conflict within the American judicial system and broader culture. Moreover, the racial categories included in the Cook County, Illinois criminal sentencing data set demonstrate a cognitive dissonance that has resulted as a consequence of the tension between America's long-standing use of its historical racial concepts and the more recent, sociological-informed developements in American language and attitude toward race, gender, and class. Historically, language and legal, racial classifications blended reduced the notion of race from any distinguishing and ethnicity-identifying phenotypes to skin color alone. Moreover, America's historical use of race legally and linguistically inconcistently and arbitrarily confused race, ethnicity, and nationality. Moreover, late Twentieth Century developments in American sociology and cultural studies introduced a more sophisticated and precise language that recognized differences among race, ethnicity, culture, and nationality. Most academics in the Twenty-first Century would recognize that the terms "Hispanic" and "Latino" are ethnoyms and are not synonymous with each other. Classifications used in the Cook County data set such as "White [Hispanic or Latino]" and "White/Black [Hispanic or Latino]" are terms that combine racial and ethnic designations. 

However, the U.S. Census Bureau is required by regulation to adhere to the 1997 Office of Management and Budget (OMB) [standards](https://www.census.gov/topics/population/race/about.html) on race and ethnicity: 

"The Census Bureau collects racial data in accordance with guidelines provided by the U.S. Office of Management and Budget (OMB), and these data are based on self-identification. The racial categories included in the census questionnaire generally reflect a social definition of race recognized in this country and not an attempt to define race biologically, anthropologically, or genetically. In addition, it is recognized that the categories of the race item include racial and national origin or sociocultural groups." 

Additionally, the U.S. Census Bureau recognizes [five racial classications](https://www.census.gov/mso/www/training/pdf/race-ethnicity-onepager.pdf): "An individual can report as White, Black or African American, Asian, American Indian and Alaska Native, Native Hawaiian and Other Pacific Islander, or some other race."

The U.S. Census Bureau definites "ethnicity" in the following [statement](https://www.census.gov/mso/www/training/pdf/race-ethnicity-onepager.pdf): "Ethnicity determines whether a person is of Hispanic origin or not. For this reason, ethnicity is broken out in two categories, Hispanic or Latino and Not Hispanic or Latino. Hispanics may report as any race."

Using the language and taxonomy outlined by the U.S. Census Bureau, it can be concluded that the Cook County criminal sentencing data combines ethnic and racial categories together within the variable `race`. For purposes of the present analysis, the category "White [Hispanic or Latino]" is merged into "Hispanic", and the categories "White/Black Hispanic or Latino" and "Biracial" are merged as "Mixed Race." 

To begin, `race` is converted into a character in order to perform basic text cleaning such as removing brackets.

```
## Variable: `race`

# Transform `race` into a character variable type
sentences$race =
  as.character(sentences$race)
# Remove brackets from `race` categories
sentences$race = 
  gsub("[", "", sentences$race, fixed = T)
sentences$race = 
  gsub("]", "", sentences$race, fixed = T
```

It is an easy call to merge records with `race` value "ASIAN" with records of value "Asian." 

```
# Merge all records with `race`` value of "ASIAN" into `race` value "Asian"
sentences$race[sentences$race %in%
                 "ASIAN"] = "Asian"
```

Those records with a blank value for `race` are merged with records of `race` value "Unknown."

```
# Merge all records with `race`` value of "" (blank) into `race` value "Unknown"
sentences$race[sentences$race %in%
                 ""] = "Unknown"
```

As stated previously, all records with a  `race` value of "White [Hispanic or Latino]" are merged with "Hispanic." Additionally, like how the `race` value "ASIAN" is merged into the similar "Asian" category, values of "HISPANIC" are merged into the category of "Hispanic."

```
# Merge all records with `race` value of "White Hispanic or Latino" into `race value "Hispanic"
sentences$race[sentences$race %in%
                 "White Hispanic or Latino"] = "Hispanic"
# Merge all records with `race` value of "HISPANIC" into `race` value "Hispanic"
sentences$race[sentences$race %in%
                 "HISPANIC"] = "Hispanic"
```

Though it was a difficult, and somewhat arbitrary decision without any guidance, the `race` value of "White/Black Hispanic or Latino" is merged into the newly-created category of "Mixed Race" due to the inclusion of "White/Black" in the Cook County-named `race` category. Furthermore, "Biracial" is also merged into the "Mixed Race" category. It should be noted that no information regarding the definition of "Biracial" is included with the Cook County criminal sentencing data set, and it is unclear whether or not "Biracial" includes other mixed racial groups other than "white" and "black" or "African American." It could be argued that if the `race` value of "Biracial" indeed were to be confirmed to be inclusive of only those defendents with a mixed race of "white" and "black," then it would be appropriate to merge "Biracial" into the `race` category of "Black" since many believe that the contemporary American judicial system unjustly views biracial individuals as neither "biracial" nor "white" but as "black," a possible social consequence of perception inherited from the historical American notion embodied in what was known as the "One-Drop Rule," which was a legal principle used for demographic and statistical purposes to define any American individual with any identifiable `black` racial heritage as `black`, thereby permitting the discriminatory and racist laws written to oppress black Americans to those of mixed racial ancestry as well. However, because it is unclear from the data whether "Biracial" refers to only those of white-black mixed race, "Biracial" has been merged into "Mixed Race" for purposes of the present analysis.

```
# Merge all records with `race` value of "White/Black Hispanic or Latino" into `race` value "Mixed Race"
sentences$race[sentences$race %in%
                 "White/Black Hispanic or Latino"] = "Mixed Race"
# Merge all records with `race` value of "Biracial" into `race` value "Mixed Race"
sentences$race[sentences$race %in%
                 "Biracial"] = "Mixed Race"
```

Next, `race` is transformed into a factored variable, and tabulations of the number of observations for each category are printed out to the console. Additionally, percentages the total number of observations are calculated for each racial categories that seem particularly low after examining the table output.

```
# Transform `race` into a factor 
sentences$race =
  as.factor(sentences$race)
# Print the levels of factor `race`
levels(as.factor(sentences$race))
# View a tabulation of `race` frequencies
table(sentences$race)

R> # View a tabulation of `race` frequencies
R> table(sentences$race)

American Indian           Asian           Black        Hispanic      Mixed Race         Unknown           White 
            120            1322          146637           38838            1091            1499           31663 
            
R> # Determine what percentage of all observations is "American Indian"
R> paste0(round(nrow(sentences[sentences$race == 'American Indian',]) / nrow(sentences) * 100, 2), "%")
[1] "0.05%"
R> # Determine what percentage of all observations is "Asian"
R> paste0(round(nrow(sentences[sentences$race == 'Asian',]) / nrow(sentences) * 100, 2), "%")
[1] "0.6%"
R> # Determine what percentage of all observations is "Mixed Race"
R> paste0(round(nrow(sentences[sentences$race == 'Mixed Race',]) / nrow(sentences) * 100, 2), "%")
[1] "0.49%"
R> # Determine what percentage of all observations is "Unknown"
R> paste0(round(nrow(sentences[sentences$race == 'Unknown',]) / nrow(sentences) * 100, 2), "%")
[1] "0.68%"
```

Since the categories of "American Indian" (0.05%), "Asian" (0.6%), "Mixed Race" (0.49%), and "Unknown" (0.68%), each representing approximately only one-half of a percent and collectively less than two percent of the total data (1.82% collectively), they are removed from the data set as outliers and in order to simplify the analysis when examining the feature `race`. It should be further noted that the category of "Unknown" should be removed from the data since it is not a true racial designation and is useless for analytical purposes. Without the records of `race` value "Unknown," the total data to be removed based upon `race` represents only 1.14% of the original data set's total number of observations.

Before removing each racial category from the data set, the data for each category is assigned to a data frame object named after `race` value.

```
# Assign all records to a data frame where the race of the defendant is "Unknown" for future inspection
unknown_race = 
  dplyr::filter(sentences, sentences$race == 
                  "Unknown")

# Remove records with Unknown Race from the dataframe
sentences = 
  dplyr::filter(sentences, sentences$race != 
                  "Unknown")

# Assign all records to a data frame  where the race of the defendant is "Mixed Race" for future inspection
mixed_race = 
  dplyr::filter(sentences, sentences$race == 
                  "Mixed Race")

# Remove records with Mixed Race from the dataframe
sentences = 
  dplyr::filter(sentences, sentences$race != 
                  "Mixed Race")

# Assign all records to a data frame where the race of the defendant is "Asian" for future inspection
asian = 
  dplyr::filter(sentences, sentences$race == 
                  "Asian")

# Remove records with Mixed Race from the dataframe
sentences = 
  dplyr::filter(sentences, sentences$race != 
                  "Asian")

# Assign all records to a data frame where the race of the defendant is "American Indian" for future inspection
american_indian = 
  dplyr::filter(sentences, sentences$race == 
                  "American Indian")

# Remove records with Mixed Race from the dataframe
sentences = 
  dplyr::filter(sentences, sentences$race != 
                  "American Indian")

# Factor `race` and add levels and labels
sentences$race =
  factor(as.character(sentences$race,
                      levels = c(1:3),
                      labels = c(
                        'Black',
                        'Hispanic',
                        'White'
                      ),
                      exclude = NULL))
```

The next variable to be inspected and transformed is `gender`. Tabulations of the number of observations for each category are printed out to the console.

```
R> table(sentences$gender)

                                               Female                       Male        Male name, no gender given 
                       777                      26656                     193729                                 3 
                   Unknown             Unknown Gender 
                         4                          1 
```

Just as with `race`, inconsistencies among the values of `gender` abound. Those records where the `gender` value is blank are merged into the category of "Unknown." Additionally, the categor of "Unknown Gender" is merged into category "Unknown." Interestingly, three records exist with a value of "Male name, no gender given," representing an officer or arresting agency's attempt at some point to salvage data. The three records with `gender` value "Male name, no gender given" are also merged into the category of "Unknown."

```
# Transform `gender` into a character variable type
sentences$gender = 
  as.character(sentences$gender)
# Convert all blank `gender` entires into value of `Unknown`
sentences$gender[sentences$gender %in%
                 ""] = "Unknown"
# Convert all "Unknown Gender" `gender` entires into value of `Unknown`
sentences$gender[sentences$gender %in%
                   "Unknown Gender"] = "Unknown"
# Convert all "Male name, no gender given"  `gender` entires into value of `Unknown`
sentences$gender[sentences$gender %in%
                   "Male name, no gender given"] = "Unknown"
```

Just as in with `race`, those records with an "Unknown" `gender` value, which represent 0.02% of the observations remaining in the data set, are removed.

```
# Determine what percentage of all observations is "Unknown"
paste0(round(nrow(sentences[sentences$gender == 'Unknown',]) / nrow(sentences) * 100, 2), "%")

unknown_gender = 
  dplyr::filter(sentences, sentences$gender == 
                  "Unknown")

# Remove records with "Unknown" gender from the dataframe
sentences = 
  dplyr::filter(sentences, sentences$race != 
                  "Unknown")
```
Next, the variable `age.at.incident` is addressed. The variable refers to the age of the defendent at the time of the incident (as opposed to the time of arrest or sentencing) expressed as a whole number and in years. Upon inspection of the values represented in `age.at.incident`, it is discovered that the data ranges from "17" up through "130." 

```
## Variable: `age.at.incident`

# Printout levels and attempt to detect anomalies or data entry mistakes
levels(as.factor(sentences$age.at.incident))

R> levels(as.factor(sentences$age.at.incident))
 [1] "17"  "18"  "19"  "20"  "21"  "22"  "23"  "24"  "25"  "26"  "27"  "28"  "29"  "30"  "31"  "32"  "33"  "34"  "35"  "36"  "37" 
[22] "38"  "39"  "40"  "41"  "42"  "43"  "44"  "45"  "46"  "47"  "48"  "49"  "50"  "51"  "52"  "53"  "54"  "55"  "56"  "57"  "58" 
[43] "59"  "60"  "61"  "62"  "63"  "64"  "65"  "66"  "67"  "68"  "69"  "70"  "71"  "72"  "73"  "74"  "75"  "76"  "77"  "78"  "79" 
[64] "80"  "81"  "82"  "84"  "85"  "86"  "114" "117" "124" "127" "130"
```

Though there are individuals with verifiable ages of up to 122 years old at time of death, all records with reported ages greater than 100 are removed from the dataset out of concern for data input mistakes more so than concern for the outlier effect. The percentage of observations with a value greater than 100 for `age.at.incident` is 1.26% of the total remaining observations.

```
# Remove obviously wrong ages: ages > 100
sentences$age.at.incident = 
  as.numeric(sentences$age.at.incident)
# Determine what percentage of all observations is greater than 100 for `age.at.incident`
paste0(round(nrow(sentences[sentences$age.at.incident > 100,]) / nrow(sentences) * 100, 2), "%")
# Assign all records with an age greater than 100 at time of incident to a data frame object
age_errors = 
  dplyr::filter(sentences, age.at.incident > 100)
# Remove all ages where defendent's age is greater than 100 at time of incident
sentences = 
  dplyr::filter(sentences, age.at.incident <= 100)
```

The next variable of interest is `commitment.unit`, and is one of especial importance and interest to the analysis as `commitment.unit` and `commitment.term` can be used to derive the total sentence length of convicted defendents.

Upon inspection of the category values for `commitment.unit`, some unexpected and unusual units are present in the data. In addition to the expected temporal values such as "Days", "Months", and "Year(s)", non-temporal values are included such as those that seemingly reference monetary fines such as "Dollars" and "Pounds." Though "Pounds" could refer to the Great British Pound, there is doubt to the exact meaning of "Pounds" as included among the values for `commitment.unit` since the data set represents the criminal sentencing of an American county (Cook County, Illinois) and since other values present represent measurements of mass e.g. "Ounces" and "Kilos." Ambiguously, it is impossible to determine whether "Pounds" is a unit of currency or a unit of mass. Additionally, blank values are included as well as an undefined value of "Terms." Finally, a unit is included to represent life sentences "Natural Life." 

Later in the analysis, a new feature, `sentence.length`, is derived from combining `commitment.unit` with `commitment.term`. In order to achieve the derivation of the new variable, non-temporal and determinant values such as "Natural Life" and "Dollars" are removed in order to produce `sentence.length`

```
## Variable: `commitment.unit` 

# Inspect the categories included in `commitment.unit`
levels(as.factor(sentences$commitment.unit))

R> levels(as.factor(sentences$commitment.unit))
 [1] ""             "Days"         "Dollars"      "Hours"        "Kilos"        "Months"       "Natural Life" "Ounces"      
 [9] "Pounds"       "Term"         "Weeks"        "Year(s)"   
```


# View a tabulation of `gender` frequencies
table(sentences$commitment.unit)

```
# Determine what percentage of all observations is blank
paste0(round(nrow(sentences[sentences$commitment.unit == "",]) / nrow(sentences) * 100, 2), "%")
# Determine what percentage of all observations is Dollars
paste0(round(nrow(sentences[sentences$commitment.unit == "Dollars",]) / nrow(sentences) * 100, 2), "%")
# Determine what percentage of all observations is Hours
paste0(round(nrow(sentences[sentences$commitment.unit == "Hours",]) / nrow(sentences) * 100, 2), "%")
# Determine what percentage of all observations is Kilos
paste0(round(nrow(sentences[sentences$commitment.unit == "Kilos",]) / nrow(sentences) * 100, 2), "%")
# Determine what percentage of all observations is blank
paste0(round(nrow(sentences[sentences$commitment.unit == "Natural Life",]) / nrow(sentences) * 100, 2), "%")
# Determine what percentage of all observations is Ounces
paste0(round(nrow(sentences[sentences$commitment.unit == "Ounces",]) / nrow(sentences) * 100, 2), "%")
# Determine what percentage of all observations is Pounds
paste0(round(nrow(sentences[sentences$commitment.unit == "Pounds",]) / nrow(sentences) * 100, 2), "%")
# Determine what percentage of all observations is Term
paste0(round(nrow(sentences[sentences$commitment.unit == "Term",]) / nrow(sentences) * 100, 2), "%")
```

Since the `commitment.unit` categories of "Unknown" (), "" (0.6%), "" (%), and "" (), each representing approximately only one-half of a percent and collectively less than two percent of the total data (% collectively), they are removed from the data set as outliers and in order to simplify the analysis when examining the feature `race`. It should be further noted that the category of "Unknown" should be removed from the data since it is not a true `commitment.unit` designation and is useless for analytical purposes. Without the records of `commitment.unit` value "Unknown," the total data to be removed based upon ___ represents only 1.14% of the original data set's total number of observations. (address mistakes)

Before removing each racial category from the data set, the data for each category is assigned to a data frame object named after `commitment.unit` value.

```
# Transform `commitment.unit` to characters
sentences$commitment.unit =
  as.character(sentences$commitment.unit)

# Assign "Dollars" to a data frame object for further inspection
dollars_unit = 
  dplyr::filter(sentences, commitment.unit == "Dollars")
sentences = 
  dplyr::filter(sentences, commitment.unit != "Dollars")
nrow(sentences)

# Assign "Pounds" to a data frame object for further inspection
pound_unit = 
  dplyr::filter(sentences, commitment.unit == "Pounds")
sentences = 
  dplyr::filter(sentences, commitment.unit != "Pounds")
nrow(sentences)

# Assign "Term" to a data frame object for further inspection
term_unit = 
  dplyr::filter(sentences, commitment.unit == "Term")
sentences = 
  dplyr::filter(sentences, commitment.unit != "Term")
nrow(sentences)

# Assign blank values to a data frame object for further inspection
blank_unit = 
  dplyr::filter(sentences, commitment.unit == "")
sentences =
  dplyr::filter(sentences, commitment.unit != "")
nrow(sentences)

# Assign "Kilos" to a data frame object for further inspection
kilos_unit = 
  dplyr::filter(sentences, commitment.unit == "Kilos")
# Remove "Kilos"
sentences = 
  dplyr::filter(sentences, commitment.unit != "Kilos")
nrow(sentences)

# Assign "Ounces" to a data frame object for further inspection
ounces_unit = 
  dplyr::filter(sentences, commitment.unit == "Ounces")
# Remove "Ounces"
sentences = 
  dplyr::filter(sentences, commitment.unit != "Ounces")
```


# Subset Natural Life (Do I want to treat a certain number as natural life sentences or even natural life sentences as a number)
natural_life_unit = 
  dplyr::filter(sentences, commitment.unit == "Natural Life")
sentences = 
  dplyr::filter(sentences, commitment.unit != "Natural Life")
# 195,934 Rows
nrow(sentences)
```





Note: Please report any bugs, coding errors, or broken web links to Thomas A. Pepperz at email thomaspepperz@icloud.com
