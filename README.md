# Using ML To Detect Bias in Criminal Sentencing Data
An R script that uses machine learning via random forests to examine [Cook County, IL criminal sentencing data](https://datacatalog.cookcountyil.gov/Courts/Sentencing/tg8v-tm6u) for human decision bias.

## Abstract

Increasingly, machine learning is praised for its ability to learn from data and apply its modeling power for predictive purposes. Similarly, machine learning has also increasingly been criticized for its tendency to learn and acquire the human bias and prejudices present in the data from which it learns. Since defining bias in human decision-making and identifying human bias di- rectly in data is notoriously difficult, the following paper establishes a framework for detecting and evaluating racial bias in the criminal sentencing procedures and practices of the American judicial system by identifying bias present in machine learning models trained on data instead of attempting to measure bias in data directly. The following paper details the application of linear regression & random forests to Cook County, Illinois criminal sentencing data in order to model and predict sentencing length & outcomes of defendants in terms of total days sentenced to prison (linear regression) and prison versus non-prison sentencing outcomes (random forests). By examining the coefficients associated with the variable of race from a linear regression model built to predict the total sentence length of a crime, one is able to audit judicial bias and overreliance upon irrelevant features of a criminal defendant such as gender and race. Furthermore, by examining the mean accuracy decrease and decrease in Gini impurity associated with the random forest models, one is able to understand the model's dependence upon individual variables such as the crime's class and the defendant's gender, race, and age. The following paper formalizes, generalizes, and applies the discussed techniques of machine learning model auditing as a tool for the detection of human bias and prejudice in human decision-making with the end goal of mobilizing data to ensure social justice whenever possible.

## Introduction

### Accessing the Data
The data has been made available on the Github repo as a zipped CSV file. However, the easiest way to access the data is to query the Socratic Open Data API (SODA API) using the [R package](https://cran.r-project.org/web/packages/RSocrata/RSocrata.pdf) `RSocrata` (developed by Hugh Devlin, Ph. D., Tom Schenk, Jr., Gene Leynes, Nick Lucius, John Malc, Mark Silver- berg, and Peter Schmeideskamp). See the [website](https://dev.socrata.com/consumers/getting-started.html) for more detail. 

To query the data, first install the `RSocrata` R package:

```r
install.packages("RSocrata")
```

Once installed, load the `RSocrata` package:

```r
library(RSocrata)
```

Navigate to the Cook County Open Data [website](https://datacatalog.cookcountyil.gov/Courts/Sentencing/tg8v-tm6u) and under the "API" tab beside the API Endpoint URI box is the word "JSON." Click on "JSON" and an option beneath it for "CSV" will appear. Choose the CSV option and then copy the URI, which should be <https://datacatalog.cookcountyil.gov/resource/tg8v-tm6u.csv>. 

Then assign the copied URI to an object named `apiEndpoint`:

```r
# Query API and assign to `apiEndpoint`
apiEndpoint = 
  "https://datacatalog.cookcountyil.gov/resource/tg8v-tm6u.csv"
```

After assigning the endpoint URI to the object, use the `read.socrata()` function to query the data from SODA API, assigning the returned data to a data frame object named `sentences`:

```r
# Query the SODA API and assign the returned data to data frame object `sentences`
sentences = 
  read.socrata(apiEndpoint)
```

As downloaded from the SODA API, the data set comprises 221,170 observations and 39 features, which each row representing a charge that resulted in decision of 'guilty', , The return data should look similar to the structure of the following output:

```r
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

### Setup
Begin by first loading (or downloading if not already installed) the required packages for the entire analysis:

```r
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

## Variables

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

### Cleaning the Data
Unfortunately, there are no enforced naming conventions for files, objects, functions, and variables for the R language. Google has published ["Google's R Style Guide"](https://google.github.io/styleguide/Rguide.xml), which encourages user's to use lower-cased names separated by periods for variable names and is the convention for variable-naming that is followed in the following analysis. Statistician Hadley Wickham, the Chief Scientist at RStudio as well as author and maintainer of popular R packages that include `tidyverse`, `tidyr`, `ggplot`, and `dplyr` amongst many others, prefers to use the underscore to separate elements within noun objects (data frame and variable names). See Wickham's brief [style guide](http://adv-r.had.co.nz/Style.html) for more details. Furthermore, an interesting study regarding R naming conventions is Rasmus Bååth's article ["The State of Naming Conventions in R"](https://journal.r-project.org/archive/2012-2/RJournal_2012-2_Baaaath.pdf) published in the *The R Journal*, Vol. 4/2, December 2012.

The following code uses the function `clean_names()` from the `janitor` package to format the variable names according to Google's R code style:

```r
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

#### Variable `race`
Inspection of the data reveals many inconsistencies in how values for `race` has been inputted, which most likely is attributable either to inconsistent data input formatting policies amongst arresting agencies or to individual officers with the responsibility of inputting data.

```r
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

```r
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

```r
# Merge all records with `race`` value of "ASIAN" into `race` value "Asian"
sentences$race[sentences$race %in%
                 "ASIAN"] = "Asian"
```

Those records with a blank value for `race` are merged with records of `race` value "Unknown."

```r
# Merge all records with `race`` value of "" (blank) into `race` value "Unknown"
sentences$race[sentences$race %in%
                 ""] = "Unknown"
```

As stated previously, all records with a  `race` value of "White [Hispanic or Latino]" are merged with "Hispanic." Additionally, like how the `race` value "ASIAN" is merged into the similar "Asian" category, values of "HISPANIC" are merged into the category of "Hispanic."

```r
# Merge all records with `race` value of "White Hispanic or Latino" into `race value "Hispanic"
sentences$race[sentences$race %in%
                 "White Hispanic or Latino"] = "Hispanic"
# Merge all records with `race` value of "HISPANIC" into `race` value "Hispanic"
sentences$race[sentences$race %in%
                 "HISPANIC"] = "Hispanic"
```

Though it was a difficult, and somewhat arbitrary decision without any guidance, the `race` value of "White/Black Hispanic or Latino" is merged into the newly-created category of "Mixed Race" due to the inclusion of "White/Black" in the Cook County-named `race` category. Furthermore, "Biracial" is also merged into the "Mixed Race" category. It should be noted that no information regarding the definition of "Biracial" is included with the Cook County criminal sentencing data set, and it is unclear whether or not "Biracial" includes other mixed racial groups other than "white" and "black" or "African American." It could be argued that if the `race` value of "Biracial" indeed were to be confirmed to be inclusive of only those defendents with a mixed race of "white" and "black," then it would be appropriate to merge "Biracial" into the `race` category of "Black" since many believe that the contemporary American judicial system unjustly views biracial individuals as neither "biracial" nor "white" but as "black," a possible social consequence of perception inherited from the historical American notion embodied in what was known as the "One-Drop Rule," which was a legal principle used for demographic and statistical purposes to define any American individual with any identifiable `black` racial heritage as `black`, thereby permitting the discriminatory and racist laws written to oppress black Americans to those of mixed racial ancestry as well. However, because it is unclear from the data whether "Biracial" refers to only those of white-black mixed race, "Biracial" has been merged into "Mixed Race" for purposes of the present analysis.

```r
# Merge all records with `race` value of "White/Black Hispanic or Latino" into `race` value "Mixed Race"
sentences$race[sentences$race %in%
                 "White/Black Hispanic or Latino"] = "Mixed Race"
# Merge all records with `race` value of "Biracial" into `race` value "Mixed Race"
sentences$race[sentences$race %in%
                 "Biracial"] = "Mixed Race"
```

Next, `race` is transformed into a factored variable, and tabulations of the number of observations for each category are printed out to the console. Additionally, percentages the total number of observations are calculated for each racial categories that seem particularly low after examining the table output.

```r
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

```r
# Assign all records to a data frame where the race of the defendant is "Unknown" for future inspection
unknown_race = 
  dplyr::filter(sentences, sentences$race == 
                  "Unknown")
# Remove records from main data frame 'sentences'
sentences = 
  dplyr::filter(sentences, sentences$race != 
                  "Unknown")
# Assign all records to a data frame  where the race of the defendant is "Mixed Race" for future inspection
mixed_race = 
  dplyr::filter(sentences, sentences$race == 
                  "Mixed Race")
# Remove records from main data frame 'sentences'
sentences = 
  dplyr::filter(sentences, sentences$race != 
                  "Mixed Race")
# Assign all records to a data frame where the race of the defendant is "Asian" for future inspection
asian = 
  dplyr::filter(sentences, sentences$race == 
                  "Asian")
# Remove records from main data frame 'sentences'
sentences = 
  dplyr::filter(sentences, sentences$race != 
                  "Asian")
# Assign all records to a data frame where the race of the defendant is "American Indian" for future inspection
american_indian = 
  dplyr::filter(sentences, sentences$race == 
                  "American Indian")
# Remove records from main data frame 'sentences'
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
                      
# Count of prison defendants by `race`
ggplot(sentences, mapping = 
         aes(x = fct_infreq(sentences$race), 
             fill = race)) +
  scale_y_continuous(label = comma) +
  geom_bar(stat = 'count', width = 0.5)  +
  labs(x = "Race", 
       y = "Count", 
       title = "Count of Defendants by Race",
       fill = "Legend") +
  theme_bw() +
  theme(text=element_text(family = "Times New Roman", 
                          face = "bold", 
                          size = 12,
                          hjust = 0.5),
        plot.title = element_text(hjust = 0.5)) + 
  scale_fill_discrete(guide = FALSE)

```

![alt text7](https://github.com/ThomasPepperz/Using-ML-To-Detect-Bias-in-Criminal-Sentencing-Data/blob/master/count-defendants-race.png)

### Variable `gender`
The next variable to be inspected and transformed is `gender`. Tabulations of the number of observations for each category are printed out to the console.

```r
R> table(sentences$gender)

                                               Female                       Male        Male name, no gender given 
                       777                      26656                     193729                                 3 
                   Unknown             Unknown Gender 
                         4                          1 
```

Just as with `race`, inconsistencies among the values of `gender` abound. Those records where the `gender` value is blank are merged into the category of "Unknown." Additionally, the categor of "Unknown Gender" is merged into category "Unknown." Interestingly, three records exist with a value of "Male name, no gender given," representing an officer or arresting agency's attempt at some point to salvage data. The three records with `gender` value "Male name, no gender given" are also merged into the category of "Unknown."

```r
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

```r
# Determine what percentage of all observations is "Unknown"
paste0(round(nrow(sentences[sentences$gender == 'Unknown',]) / nrow(sentences) * 100, 2), "%")

unknown_gender = 
  dplyr::filter(sentences, sentences$gender == 
                  "Unknown")

# Remove records with "Unknown" gender from the dataframe
sentences = 
  dplyr::filter(sentences, sentences$gender != 
                  "Unknown")
                  
# Count of defendants by `gender`
ggplot(sentences, mapping = 
         aes(x = fct_infreq(sentences$gender), 
             fill = gender)) +
  scale_y_continuous(label = comma) +
  geom_bar(stat = 'count', width = 0.5)  +
  labs(x = "Gender", 
       y = "Count", 
       title = "Count of Defendants by Gender",
       fill = "Legend") +
  theme_bw() +
  theme(text=element_text(family = "Times New Roman", 
                          face = "bold", 
                          size = 12,
                          hjust = 0.5),
        plot.title = element_text(hjust = 0.5)) + 
  scale_fill_discrete(guide = FALSE)
```

![alt text8](https://github.com/ThomasPepperz/Using-ML-To-Detect-Bias-in-Criminal-Sentencing-Data/blob/master/count-defendants-gender.png)

### Variable `age.at.incident`
Next, the variable `age.at.incident` is addressed. The variable refers to the age of the defendent at the time of the incident (as opposed to the time of arrest or sentencing) expressed as a whole number and in years. Upon inspection of the values represented in `age.at.incident`, it is discovered that the data ranges from "17" up through "130." 

```r
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

```r
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
  
# Distribution of defendants (age at time of incident)
ggplot(sentences, mapping = aes(age.at.incident, fill = age.at.incident)) +
  geom_histogram(fill="orange2", bins = 40) +
  scale_y_continuous(label = comma) +
  labs(x = "Age", 
       y = "Frequency", 
       title = "Distribution of Defendant Ages at Time of Incident") +
  theme_bw() +
  theme(text=element_text(family = "Times New Roman", 
                          face = "bold", 
                          size = 12,
                          hjust = 0.5),
        plot.title = element_text(hjust = 0.5)) + 
  scale_fill_discrete(guide = FALSE)
```

![alt text9](https://github.com/ThomasPepperz/Using-ML-To-Detect-Bias-in-Criminal-Sentencing-Data/blob/master/distribution-defendants-age.png)

### Variables `commitment.unit` & `commitment.term`
The next variables of interest are `commitment.unit` and `commitment.term`, and is one of special importance and interest to the analysis as `commitment.unit` and `commitment.term` can be used to derive the total sentence length of convicted defendents.

Upon inspection of the category values for `commitment.unit`, some unexpected and unusual units are present in the data. In addition to the expected temporal values such as "Days", "Months", and "Year(s)", non-temporal values are included such as those that seemingly reference monetary fines such as "Dollars" and "Pounds." Though "Pounds" could refer to the Great British Pound, there is doubt to the exact meaning of "Pounds" as included among the values for `commitment.unit` since the data set represents the criminal sentencing of an American county (Cook County, Illinois) and since other values present represent measurements of mass e.g. "Ounces" and "Kilos." Ambiguously, it is impossible to determine whether "Pounds" is a unit of currency or a unit of mass. Additionally, blank values are included as well as an undefined value of "Terms." Finally, a unit is included to represent life sentences "Natural Life." 

Later in the analysis, a new feature, `sentence.length`, is derived from combining `commitment.unit` with `commitment.term`. In order to achieve the derivation of the new variable, non-temporal and determinant values such as "Natural Life" and "Dollars" are removed in order to produce `sentence.length`

```r
## Variable: `commitment.unit` 

# Inspect the categories included in `commitment.unit`
levels(as.factor(sentences$commitment.unit))

R> levels(as.factor(sentences$commitment.unit))
 [1] ""             "Days"         "Dollars"      "Hours"        "Kilos"        "Months"       "Natural Life" "Ounces"      
 [9] "Pounds"       "Term"         "Weeks"        "Year(s)"   
```

The following code chunk computes and prints out the percentage of the total number of remaining observations in the data set for each of the non-temporal `commitment.unit` values. Except for the "Term" commitment unit (1.04%), each of the other non-temporal commitment units represent approximately one-half or less of a percent of the total number of observations in the data set. Since each commitment unit type constitutes a relatively small percentage of the total data set and because each is a non-temporal commitment unit, they are removed from the main data frame `sentences` are are assigned to a data frame named after each unit for further inspection.

```r
R> # Determine what percentage of all observations is blank
R> paste0(round(nrow(sentences[sentences$commitment.unit == "",]) / nrow(sentences) * 100, 2), "%")
[1] "0.63%"
R> # Determine what percentage of all observations is Dollars
R> paste0(round(nrow(sentences[sentences$commitment.unit == "Dollars",]) / nrow(sentences) * 100, 2), "%")
[1] "0.03%"
R> # Determine what percentage of all observations is Hours
R> paste0(round(nrow(sentences[sentences$commitment.unit == "Hours",]) / nrow(sentences) * 100, 2), "%")
[1] "0.01%"
R> # Determine what percentage of all observations is Kilos
R> paste0(round(nrow(sentences[sentences$commitment.unit == "Kilos",]) / nrow(sentences) * 100, 2), "%")
[1] "0%"
R> # Determine what percentage of all observations is blank
R> paste0(round(nrow(sentences[sentences$commitment.unit == "Natural Life",]) / nrow(sentences) * 100, 2), "%")
[1] "0.31%"
R> # Determine what percentage of all observations is Ounces
R> paste0(round(nrow(sentences[sentences$commitment.unit == "Ounces",]) / nrow(sentences) * 100, 2), "%")
[1] "0%"
R> # Determine what percentage of all observations is Pounds
R> paste0(round(nrow(sentences[sentences$commitment.unit == "Pounds",]) / nrow(sentences) * 100, 2), "%")
[1] "0%"
R> # Determine what percentage of all observations is Term
R> paste0(round(nrow(sentences[sentences$commitment.unit == "Term",]) / nrow(sentences) * 100, 2), "%")
[1] "1.04%"
```

For each non-temporal `commitment.unit` value, the associated records are first assigned to a data frame named after the unit before being removed from the main data frame `sentences`.

```r
#### Non-temporal `commitment.unit` values ####

# Transform `commitment.unit` to characters
sentences$commitment.unit =
  as.character(sentences$commitment.unit)

# Assign "Dollars" to a data frame object for further inspection
dollars_unit = 
  dplyr::filter(sentences, commitment.unit == "Dollars")
# Remove "Dollars" values from the main data frame
sentences = 
  dplyr::filter(sentences, commitment.unit != "Dollars")

# Assign "Pounds" to a data frame object for further inspection
pound_unit = 
  dplyr::filter(sentences, commitment.unit == "Pounds")
# Remove "Pounds" values from the main data frame
sentences = 
  dplyr::filter(sentences, commitment.unit != "Pounds")

# Assign "Term" to a data frame object for further inspection
term_unit = 
  dplyr::filter(sentences, commitment.unit == "Term")
# Remove "Term" values from the main data frame
sentences = 
  dplyr::filter(sentences, commitment.unit != "Term")

# Assign blank values to a data frame object for further inspection
blank_commitment_unit = 
  dplyr::filter(sentences, commitment.unit == "")
# Remove blank values from the main data frame
sentences =
  dplyr::filter(sentences, commitment.unit != "")

# Assign "Kilos" to a data frame object for further inspection
kilos_unit = 
  dplyr::filter(sentences, commitment.unit == "Kilos")
# Remove "Kilos" from the main data frame
sentences = 
  dplyr::filter(sentences, commitment.unit != "Kilos")

# Assign "Ounces" to a data frame object for further inspection
ounces_unit = 
  dplyr::filter(sentences, commitment.unit == "Ounces")
# Remove "Ounces" from the main data frame
sentences = 
  dplyr::filter(sentences, commitment.unit != "Ounces")
```

The `commitment.unit` value of "Natural Life" is an interesting and unique commitment unit as it is a temporal unit but unlike hours, days, or weeks, "Natural Life" is an indeterminate value and cannot be quantified in a useful and standardized manner. Natural life sentences are discussed later in the analysis along with a discussion of determinate, temporal commitment terms that are long enough that the resulting sentence length is for all intents and purposes a natural life sentence.

```r
# Assign "Natural Life" to a data frame object for further inspection
natural_life = 
  dplyr::filter(sentences, commitment.unit == "Natural Life")
# Remove "Ounces"
sentences = 
  dplyr::filter(sentences, commitment.unit != "Natural Life")
```

For each temporal commitment unit (hours, days, weeks, months, and years), the observations for the associated values are subsetted and assigned to a data frame named after the unit before being removed from the main data frame `sentences`. Then, the variable `commitment.term` for each data frame is inspected for errors by examining the levels produced from factoring the variable. In many of the data frames, the commitment term values include leading zeroes, which either indicate data input errors or merely represent the idiosyncratic data input formatting of an individual officer or arresting agency. Leading zeroes are removed and those non-zero values that remain are left included in the data set and treated as non-errors. However, those values that were represented as either "0" or "00" are removed from the data frame and treated as errors. Finally, each value in `commitment.term` is re-inspected for errors before the variable is transformed from a character to a numeric variable.

```r
#### Temporal `commitment.unit` values ####`

# Hours: Assign "Hours" to a data frame object for further inspection
hours = 
  dplyr::filter(sentences, commitment.unit == "Hours")
# Remove "Hours" from the main data frame
sentences = 
  dplyr::filter(sentences, commitment.unit != "Hours")
# Inspect `commitment.term` for segment 'hours'
levels(as.factor(as.character(hours$commitment.term)))
# Remove all leading zeros from `commitment.term` for 'days'
hours$commitment.term = 
  gsub("(^|[^0-9])0+", "\\1", hours$commitment.term, perl = TRUE)
# Assign any errors to a data frame object for further inspection
hours_errors = 
  dplyr::filter(hours, commitment.term == "")
# Remove any errors from the segmented `commitment.unit` data frame
hours = 
  dplyr::filter(hours, commitment.term != "")
# Re-inspect `commitment.term` for segment 'hours'
levels(as.factor(as.character(hours$commitment.term)))
# Convert `commitment.term` for segment 'hours' into a numeric variable
hours$commitment.term = 
  as.numeric(as.character(hours$commitment.term))


# Days: Assign "Days" to a data frame object for further inspection
days = 
  dplyr::filter(sentences, commitment.unit == "Days")
# Remove "Days" from the main data frame
sentences = 
  dplyr::filter(sentences, commitment.unit != "Days")
# Inspect `commitment.term` for segment 'days'
levels(as.factor(as.character(days$commitment.term)))
# Remove all leading zeros from `commitment.term` for 'days'
days$commitment.term = 
  gsub("(^|[^0-9])0+", "\\1", days$commitment.term, perl = TRUE)
# Assign any errors to a data frame object for further inspection
days_errors = 
  dplyr::filter(days, commitment.term == "")
# Remove any errors from the segmented `commitment.unit` data frame
days = 
  dplyr::filter(days, commitment.term != "")
# Re-inspect `commitment.term` for segment 'days'
levels(as.factor(as.character(days$commitment.term)))
# Convert `commitment.term` for segment 'days' into a numeric variable
days$commitment.term = 
  as.numeric(as.character(days$commitment.term))


# Weeks: Assign "Weeks" to a data frame object for further inspection
weeks = 
  dplyr::filter(sentences, commitment.unit == "Weeks")
# Remove "Weeks" from the main data frame
sentences = 
  dplyr::filter(sentences, commitment.unit != "Weeks")
# Inspect `commitment.term` for segment 'weeks'
levels(as.factor(as.character(weeks$commitment.term)))
# Remove all leading zeros from `commitment.term` for 'weeks'
weeks$commitment.term = 
  gsub("(^|[^0-9])0+", "\\1", weeks$commitment.term, perl = TRUE)
# Assign any errors to a data frame object for further inspection
weeks_errors = 
  dplyr::filter(weeks, commitment.term == "")
# Remove any errors from the segmented `commitment.unit` data frame
weeks = 
  dplyr::filter(weeks, commitment.term != "")
# Re-inspect `commitment.term` for segment 'weeks'
levels(as.factor(as.character(weeks$commitment.term)))
# Convert `commitment.term` for segment 'weeks' into a numeric variable
weeks$commitment.term = 
  as.numeric(as.character(weeks$commitment.term))


# Months: Assign "Months" to a data frame object for further inspection
months = 
  dplyr::filter(sentences, commitment.unit == "Months")
# Remove "Months" from the main data frame
sentences = 
  dplyr::filter(sentences, commitment.unit != "Months")
# Inspect `commitment.term` for segment 'months'
levels(as.factor(as.character(months$commitment.term)))
# Remove all leading zeros from `commitment.term` for 'months'
months$commitment.term = 
  gsub("(^|[^0-9])0+", "\\1", months$commitment.term, perl = TRUE)

# For errors that included the `commitment.unit` along with the `commitment.term`, replace with only the number
months$commitment.term[months$commitment.term %in% 
                         "30 months"] = "30"
months$commitment.term[months$commitment.term %in% 
                         "24 wrap"] = "24"
months$commitment.term[months$commitment.term %in% 
                         "18 months"] = "18"
# Assign any errors to a data frame object for further inspection
months_errors = 
  dplyr::filter(months, commitment.term == "")
# Remove any errors from the segmented `commitment.unit` data frame
months = 
  dplyr::filter(months, commitment.term != "")
# Re-inspect `commitment.term` for errors in segment 'months'
levels(as.factor(as.character(months$commitment.term)))
# Convert `commitment.term` for segment 'months' into a numeric variable
months$commitment.term = 
  as.numeric(as.character(months$commitment.term))


# Years: Assign "Years" to a data frame object for further inspection
years = 
  dplyr::filter(sentences, commitment.unit == "Year(s)")
# Remove "Years" from the main data frame
sentences = 
  dplyr::filter(sentences, commitment.unit != "Year(s)")
# Years: Rename `commitment.unit` value "Year(s)" to "years"
years$commitment.unit[years$commitment.unit %in% 
                "Year(s)"] = "Years"
# Inspect `commitment.term` for errors in segment 'years'
levels(as.factor(as.character(years$commitment.term)))
# Change to "02032012" to "0" for removal
years$commitment.term[years$commitment.term %in%
                        "02032012"] = "0"
# Change to "two" to "2" for removal
years$commitment.term[years$commitment.term %in%
                        "two"] = "2"
# Remove all leading zeros from `commitment.term` for 'years'
years$commitment.term = 
  gsub("(^|[^0-9])0+", "\\1", years$commitment.term, perl = TRUE)

# Assign any errors to a data frame object for further inspection
years_errors = 
  dplyr::filter(years, 
                commitment.term == "" |
                commitment.term == "2`"
                  )
# Remove any errors from the segmented `commitment.unit` data frame
years = 
  dplyr::filter(years, 
                years$commitment.term != "2`")
years = 
  dplyr::filter(years, 
                years$commitment.term != "")
                
# Re-inspect `commitment.term` for errors in segment 'years'
levels(as.factor(as.character(years$commitment.term)))
# Convert `commitment.term` for segment 'years' into a numeric variable
years$commitment.term = 
  as.numeric(years$commitment.term)
```

### Variable `sentence.length`
As indicated in the Cook County criminal sentencing data documentation, a sentence length can be determined for each charge by considering the `commitment.unit` (hours, days, weeks, et cetera) and the `commitment.term`. For example, using the documentation instructions, one could take a charge with a commitment unit of "Weeks" and a commitment term of "4" and determine that the criminal defendent was sentenced to "4 Weeks" of whatever `commitment.type` (Prison, probation, rehabilitation, et cetera).  More over, if one wanted to determine what the sentence length is in terms of days, a coefficient of "7" days could be used in the case of weeks and multiplied with the commitment term to arrive at 28 days using the previous example. 

For the present analysis, values for each charge are derived by creating a new variable `sentence.length` for each `commitment.unit` segment and standardizing into a common unit of "Days" by computing a `commitment.unit` coefficient for each segment (hours, days, weeks, months, and years) and applying the coefficient with multiplication or division where appropriate to the `commitment.term`. 

For data frame `hours`, a coefficient of 24 (hours) is used to divide each `commitment.term` value in order to standardize the sentence length into day units (e.g. 48 hours / 24 hours [1 day] = 2 days). 

For data frame `days`, no coefficient is applied to the `commitment.term` values as `days` is necessarily already standardized to day units (e.g. 4 days * 1 day = 4 days).

For data frame `weeks`, a coefficient of 7 (days) is used to multiply each `commitment.term` value in order to standardize the sentence length into day units (e.g. 4 weeks * 7 days = 28 days). 

For data frame `months`, a coefficient of 30.45 (days) is used to multiply each `commitment.term` value in order to standardize the sentence length into day units (e.g. 2 months * 30.45 days = 60.90 days). Note: 30.45 is chosen as the coefficient to represent months since the number of days per month varies depending on the month of the year and whether or not the month is February and the year a leap year. In a non-leap year, each month has a mean duration of 30.42 days (365 days / 12 months = 30.42 days). However, in a leap year, each month has a mean duration of 30.50 days (366 days / 12 months = 30.50 days). Since it is impractical for the purposes of this analysis to determine whether or not the sentenced months of a criminal defendent fall within a leap year or along particular months, each month is valued as 3.44 days in duration [(30.42 days * 3 years) + (30.50 days * 1 year) / 4 years = 30.44 day)].

For data frame `years`, a coefficient of 365.2425 (days) is used to multiply each `commitment.term` value in order to standardize the sentence length into day units (e.g. 2 years * 365.28 days = 730.485 days). Note: each year is valued at 35.2425 days since the mean value of a month used in the analysis is 30.44 and (1 year * (12 months * 30.44 days) = 365.28 days.

After each `commitment.term` for each `commitment.unit` data frame segment is standardized into days, the five data frames (`hours`, `days`, `weeks`, `months`, and `years`) are merged into one data frame `sentences`. Moreover, since each `commitment.term` value in the data frame has been standardized into days, all of the values for `commitment.unit` are converted to "Days" in order to reflect the change in units.

```r
#### Create new variable `sentence.length` and standardize `commitment.term` to days ####

# Hours (Divide by 24)
hours$sentence.length =
  (hours$commitment.term / 24)

# Days (No standardization required)
days$sentence.length =
  days$commitment.term

# Weeks (Multiply by 7)
weeks$sentence.length =
  (weeks$commitment.term * 7)

# Months (Multiply by 30.44)
months$sentence.length =
  (months$commitment.term * 30.44)

# Years (Multiply by 365.28)
years$sentence.length =
  (years$commitment.term * 365.28)

# Recombine all time-based `commitment.unit` data frames into one data frame 'sentences'
sentences = 
  rbind(hours,
        days,
        weeks,
        months,
        years
        )

# Now that each `commitment.term` is standardized to days, reassign all `commitment.units` to "Days"
sentences$commitment.unit = "Days"

# Distribution of sentences by length (in days) 
ggplot(sentences, mapping = aes(sentence.length, fill = sentence.length)) +
  geom_histogram(fill="blue2", bins = 100) +
  scale_x_continuous(limits = c(0, 20000), label = comma) +
  scale_y_continuous(label = comma) +
  labs(x = "Sentence Length (in Days)", 
       y = "Frequency", 
       title = "Distribution of Prison Sentences by Length (in days)") +
  theme_bw() +
  theme(text=element_text(family = "Times New Roman", 
                          face = "bold", 
                          size = 12,
                          hjust = 0.5),
        plot.title = element_text(hjust = 0.5)) + 
  scale_fill_discrete(guide = FALSE)
```

![alt text11](https://github.com/ThomasPepperz/Using-ML-To-Detect-Bias-in-Criminal-Sentencing-Data/blob/master/distribution-sentence-lengths.png)


```r
# Plot mean sentence length for each race.
means.df.1 = 
  aggregate(formula = sentence.length ~ race, data = sentences, FUN = mean)

# Mean sentence length by race
ggplot(means.df.1,aes(x=race , y=sentence.length, fill=race)) + 
  geom_bar(stat = 'identity', width=0.5) +
  scale_y_continuous(label = comma) +
  labs(x = "Race", 
       y = "Sentence Length (days)", 
       title = "Mean Sentence Length by Race",
       fill = "Legend") +
  theme_bw() +
  theme(text=element_text(family = "Times New Roman", 
                          face = "bold", 
                          size = 12,
                          hjust = 0.5),
        plot.title = element_text(hjust = 0.5)) +
  scale_fill_discrete(guide = FALSE)
```

![alt text12](https://github.com/ThomasPepperz/Using-ML-To-Detect-Bias-in-Criminal-Sentencing-Data/blob/master/mean-sentence-length-race.png)

### Variable `primary.charge`
Because there may exist significant differences in the judicial treatment of criminal charges that are classified as the primary charge, the data frame 'sentences' is further segmented according to whether or not it is a primary charge. Those observations where the value of `primary.charge` is "false" are assigned to data frame `primary_charge_false` and those with the value of 'true' are assigned to `primary_charge_true`. However, prior to segmentation, the values of each row are transformed such that those with with the value "true" are converted to "1" and those with the value "false" are converted to "0." `primary.charge` is first factored before the data frame is segmented.

```r
#### Primary Charge ####

# Transform `primary.charge` into a character to prepare to replace values
sentences$primary.charge = 
  as.character(sentences$primary.charge)
# Replace 'true' values with "1"
sentences[sentences$primary.charge == 'true',]$primary.charge = "1"
# Replace 'false' values with 0
sentences[sentences$primary.charge == 'false',]$primary.charge = "0"
# Factor `primary.charge`
sentences$primary.charge = 
  as.factor(sentences$primary.charge)
# Check the levels of the factored variable
levels(sentences$primary.charge)
# Segment observations that are the primary charge
primary_charge_true = 
  dplyr::filter(sentences, primary.charge == "1")
# Segment observations that are not the primary charge
primary_charge_false = 
  dplyr::filter(sentences, primary.charge == "0")
# Check that the amount of dataframe rows are equal to original
nrow(sentences) == nrow(primary_charge_false) + nrow(primary_charge_true)
```

### Variables `sentence.type` & `commitment.type`
The next variables considered are `sentence.type` and `commitment.type`, which both describes the type of punishment to which a criminal defendent is sentenced for a particular charge. Whereas the documentation describes `sentence.type` as a "broad type of sentence issued," it characterizes `commitment.type` as "a more specific type of sentence issued. There are fourteen different sentence types and twenty five commitment types present in the Cook County criminal sentencing data set, which are listed in the following R code output:

```r
#### Sentence Type Segmentation ####

# Print out levels of `sentence.type`
R> levels(sentences$sentence.type)
 [1] "2nd Chance Probation"                  "Conditional Discharge"                 "Conditional Release"                  
 [4] "Conversion"                            "Cook County Boot Camp"                 "Death"                                
 [7] "Inpatient Mental Health Services"      "Jail"                                  "Prison"                               
[10] "Probation"                             "Probation Terminated Instanter"        "Probation Terminated Satisfactorily"  
[13] "Probation Terminated Unsatisfactorily" "Supervision"  

R> # Print out levels of `commitment.type`
R> levels(sentences$commitment.type)
[1] ""                                         "2nd Chance Probation"                     "710/410 Probation"                       
 [4] "Conditional Discharge"                    "Conditional Release"                      "Cook County Boot Camp"                   
 [7] "Cook County Department of Corrections"    "Cook County Impact Incarceration Program" "Court Supervision"                       
[10] "Death"                                    "Domestic Violence Probation"              "Drug Court Probation"                    
[13] "Drug School"                              "Gang Probation"                           "Home Confinement"                        
[16] "Illinois Department of Corrections"       "Inpatient Mental Health Services"         "Intensive Drug Probation Services"       
[19] "Intensive Probation Services"             "Juvenile IDOC"                            "Mental Health Probation"                 
[22] "Natural Life"                             "Periodic Imprisonment"                    "Probation"                               
[25] "Repeat Offender Probation"                "Sex Offender Probation"                   "Veteran's Court Probation" 
```
To produce visualizations of the data, use the following code:

```r
# Count of charges by `sentence.type`
ggplot(sentences, mapping = 
         aes(x = fct_infreq(sentences$sentence.type), 
             fill = sentence.type)) +
  scale_y_continuous(label = comma) +
  geom_bar(stat = 'count', width = 0.5)  +
  coord_flip() +
  labs(x = "Sentence Type", 
       y = "Count", 
       title = "Count of Charges by Sentence Type",
       fill = "Legend") +
  theme_bw() +
  theme(text=element_text(family = "Times New Roman", 
                          face = "bold", 
                          size = 12,
                          hjust = 0.5),
        plot.title = element_text(hjust = 0.5)) + 
  scale_fill_discrete(guide = FALSE)
```

![alt text1](https://github.com/ThomasPepperz/Using-ML-To-Detect-Bias-in-Criminal-Sentencing-Data/blob/master/count-charges-commitment-type.png)

```r
# Count of charges by `commitment.type`
ggplot(sentences, mapping = 
         aes(x = fct_infreq(sentences$commitment.type), 
             fill = commitment.type)) +
  scale_y_continuous(label = comma) +
  geom_bar(stat = 'count', width = 0.5)  +
  coord_flip() +
  labs(x = "Commitmet Type", 
       y = "Count", 
       title = "Count of Charges by Commitment Type",
       fill = "Legend") +
  theme_bw() +
  theme(text=element_text(family = "Times New Roman", 
                          face = "bold", 
                          size = 12,
                          hjust = 0.5),
        plot.title = element_text(hjust = 0.5)) + 
  scale_fill_discrete(guide = FALSE)
```

![alt text2](https://github.com/ThomasPepperz/Using-ML-To-Detect-Bias-in-Criminal-Sentencing-Data/blob/master/count-charges-sentence-type.png)

It is stated in the *Cook County State Attorney 2017 Data Report* (Foxx, M. Kimberly 2018) that "Sentencing is the judgment imposed by the court on people who have been convicted. Each count for which there is a conviction receives a separate sentence; depending on the circumstances those sentences may be served concurrently or consecutively." 

> Prison: a sentence of one year or more of incarceration, served in the Illinois Department of Corrections. <br /> <br />Jail: a sentence of less than one year served in county jail; a sentence of felony probation may also include a requirement to serve time in Cook County Jail. <br /> <br />Boot Camp: a program of military activities, physical exercise, labor-intensive work, and substance abuse treatment; successful completion of boot camp may lead to a sentence reduced to time served and placement on supervision. <br /> <br />Probation: mandatory compliance with court-ordered conditions for a specific period of time, monitored by a probation officer. <br /> <br />Conditional discharge: mandatory compliance with court-ordered conditions for a specific period of time, usually without the supervision of a probation officer. <br /> <br />Supervision: compliance with court-ordered conditions while conviction is suspended. Successful completion results in release without a conviction. <br /> <br />Note: only misdemeanors can receive a supervision sentence; while this report does not include misdemeanor charges, a case may receive supervision if it was initially charged as a felony then reduced to a misdemeanor through a plea or a finding of guilty on a lesser offense.

Though the documentation does not define nor describe each of the possible sentence types, each sentence type can be classified according to whether or not the defendent was incarcerated as a result of the sentencing. 

#### Sentencing: Death

The first sentence-commitment type examined and segmented is "Death." Interestingly, "Death" is both a value for `sentence.type` and `commitment.type`. There are 59 charges that resulted in a sentence type of "Death." 

```r
# Death Penalty 1: Assign rows where `sentence.type` is "Death" to data frame
death1 = 
  dplyr::filter(sentences, sentence.type == "Death")
# Print out the number of rows in data frame 'death1'
nrow(death1)
# Death Penalty 1: Remove rows where `sentence.type` is "Death" from main data frame
sentences = 
  dplyr::filter(sentences, sentence.type != "Death")
 ```

Confusingly, when examining the commitment type for those records with a sentence type of "Death," only a portion of those records have "Death" listed as the commitment type as well. 

```r
R> # Print out levels of `commitment.type` for 'death1'
R> levels(as.factor(as.character(death1$commitment.type)))
[1] ""                                      "Cook County Department of Corrections" "Death"                                
[4] "Illinois Department of Corrections"   
```

Too add to the confusion, there is one observation where the more specific sentencing variable of `commitment.type` is "Death" but the broader `sentence.type` is not "Death" but instead listed as "Conversion."

```r
# Death Penalty 2: Assign rows where `commitment.type` is "Death" to data frame
death2 =
  dplyr::filter(sentences, commitment.type == "Death")
# Print out the number of rows in data frame 'death2'
nrow(death2)
# Death Penalty 2: Remove rows where `commitment.type` is "Death" from main data frame
sentences = 
  dplyr::filter(sentences, commitment.type != "Death")
R> # Print out levels of `commitment.type` for 'death1'
R> levels(as.factor(as.character(death2$sentence.type)))
[1] "Conversion"
```

Data frames `death1` and `death2` are then combined into one data frame, `death`. The number of observations that resulted in a sentencing of death represent 0.03% of the entire number of rows from the Cook County criminal sentencing data set. 

```r
# Death: Combine data frames 'death1' and 'death2' into one data frame 'death'
death = 
  rbind(death1,
        death2)
# Determine what percentage of all observations resulted in a sentencing of "Death"
paste0(round(nrow(death) / nrow(sentences) * 100, 2), "%")

# Count of death commitment.type charges by `race`
ggplot(death, mapping = 
         aes(x = fct_infreq(death$race), 
             fill = race)) +
  scale_y_continuous(label = comma) +
  geom_bar(stat = 'count', width = 0.5)  +
  labs(x = "Race", 
       y = "Count", 
       title = "Count of Charges Resulting in a Death Sentencing by Race",
       fill = "Legend") +
  theme_bw() +
  theme(text=element_text(family = "Times New Roman", 
                          face = "bold", 
                          size = 12,
                          hjust = 0.5),
        plot.title = element_text(hjust = 0.5)) + 
  scale_fill_discrete(guide = FALSE)
```

![alt text5]( https://github.com/ThomasPepperz/Using-ML-To-Detect-Bias-in-Criminal-Sentencing-Data/blob/master/count-death-sentences-race.png)

It should be noted that Illinois Governor Pat Quinn signed legislation in 2011 that abolished the death penalty in the state and the last execution occured in 1999. Fifteen convicted prisoners were on "death row" at the time of the abolition of the dealth penalty in Illinois, and the death sentences of all fifteen convicted prisoners were commuted to life sentences by Governor Quinn.

#### Sentencing: Conditional Discharge & Conditional Release

Talk ABOUT CONDITIONAL DISCHARGE AND RELEASES

```r
#### Conditional Sentences ####

# Segment conditional discharge sentence types
cond_discharge = 
  dplyr::filter(sentences, sentence.type == "Conditional Discharge")
# Segment conditional discharge sentence types resulting in imprisonment
cond_discharge_prison = 
  dplyr::filter(cond_discharge, 
              cond_discharge$commitment.type == "Illinois Department of Corrections" |
              cond_discharge$commitment.type == "Cook County Department of Corrections")
# Segment conditional discharge sentence types not resulting in imprisonment
cond_discharge_no_prison = 
  dplyr::filter(cond_discharge, 
                commitment.type != "Illinois Department of Corrections" |
                commitment.type != "Cook County Department of Corrections")

# Segment conditional release sentence types
cond_release = 
  dplyr::filter(sentences, sentence.type == "Conditional Release")
# Segment conditional release sentence types resulting in imprisonment
cond_release_prison = 
  dplyr::filter(cond_release, 
                commitment.type == "Illinois Department of Corrections" |
                commitment.type == "Cook County Department of Corrections")
# Segment conditional release sentence types not resulting in imprisonment
cond_release_no_prison = 
  dplyr::filter(cond_release, 
                commitment.type != "Illinois Department of Corrections" |
                commitment.type != "Cook County Department of Corrections")
```

No Prison yada yada

```r
#### No Prison Sentence Types ####

# 2nd Chance Probation
probation_2nd = 
  dplyr::filter(sentences, sentence.type == "2nd Chance Probation")

# Supervision
supervision = 
  dplyr::filter(sentences, sentence.type == "Supervision")

# Conversion 
conversion = 
  dplyr::filter(sentences, sentence.type == "Conversion")

# Cook County Boot Camp
bootcamp = 
  dplyr::filter(sentences, sentence.type == "Cook County Boot Camp")

# Inpatient Mental Health Services
mental_inpatient  = 
  dplyr::filter(sentences, sentence.type == "Inpatient Mental Health Services")

# Probation
probation = 
  dplyr::filter(sentences, sentence.type == "Probation")

# Probation Terminated Satisfactorily
probation_sat = 
  dplyr::filter(sentences, sentence.type == "Probation Terminated Satisfactorily")

# Probation Terminated Unsatisfactorily
probation_unsat = 
  dplyr::filter(sentences, sentence.type == "Probation Terminated Unsatisfactorily")

# Probation Terminated Instanter
probation_instanter = 
  dplyr::filter(sentences, sentence.type == "Probation Terminated Instanter")


# Combine all charges not resulting in imprisonment to data frame 'no_prison'
no_prison = 
  rbind(probation_2nd,
        cond_discharge_no_prison,
        cond_release_no_prison,
        conversion,
        bootcamp,
        mental_inpatient,
        probation,
        probation_sat,
        probation_unsat,
        probation_instanter
)

# Create new variable `outcome`
no_prison$outcome = 
  "No Prison"
  
# Factor `outcome`
no_prison$outcome =
  factor(no_prison$outcome)

# Count of no prison commitment.type charges by `race`
ggplot(no_prison, mapping = 
         aes(x = fct_infreq(no_prison$race), 
             fill = race)) +
  scale_y_continuous(label = comma) +
  geom_bar(stat = 'count', width = 0.5)  +
  labs(x = "Race", 
       y = "Count", 
       title = "Count of Charges Resulting in a Non-Prison Sentence by Race",
       fill = "Legend") +
  theme_bw() +
  theme(text=element_text(family = "Times New Roman", 
                          face = "bold", 
                          size = 12,
                          hjust = 0.5),
        plot.title = element_text(hjust = 0.5)) + 
  scale_fill_discrete(guide = FALSE)

```

![alt text4](https://github.com/ThomasPepperz/Using-ML-To-Detect-Bias-in-Criminal-Sentencing-Data/blob/master/count-no-prison-sentences-race.png)

Prison yada yad

```r
#### Prison Sentence Types ####

# Jail
jail = 
  dplyr::filter(sentences, sentence.type == "Jail")

# Prison
prison = 
  dplyr::filter(sentences, sentence.type == "Prison")

# Combine all charges resulting in imprisonment to data frame 'prison'
prison =
  rbind(cond_discharge_prison,
        cond_release_prison,
        jail,
        prison)
        
# Add a variable `outcome`
prison$outcome = 
  "Prison"
  
# Factor `outcome`
prison$outcome = 
  factor(prison$outcome)

# Count of charges resulting in prison sentence by race
ggplot(sentences, mapping = 
         aes(x = fct_infreq(sentences$sentence.judge), 
             fill = sentence.judge)) +
  scale_y_continuous(label = comma) +
  coord_flip() +
  geom_bar(stat = 'count', width = 0.5)  +
  labs(x = "Race", 
       y = "Count", 
       title = "Count of Charges Resulting in a Prison Sentence by Race",
       fill = "Legend") +
  theme_bw() +
  theme(text=element_text(family = "Times New Roman", 
                          face = "bold", 
                          size = 12,
                          hjust = 0.5),
        plot.title = element_text(hjust = 0.5)) + 
  scale_fill_discrete(guide = FALSE)

```

Include a visualization of prison by race

![alt text6](https://github.com/ThomasPepperz/Using-ML-To-Detect-Bias-in-Criminal-Sentencing-Data/blob/master/count-prison-sentences-race.png)

```r
#### Segment data into Jury versus Bench (Judge) Trials

# Segment data to jury trials
jury_prison = 
  filter(prison, sentence.judge == "")
jury_no_prison = 
  filter(no_prison, sentence.judge == "")

# Segment data to judge trials
judge_prison = 
  filter(prison, sentence.judge != "")
judge_no_prison = 
  filter(no_prison, sentence.judge != "")

# Add columns to indicate judge or jury trials
jury_prison$trial.type= "Jury Trial"
jury_no_prison$trial.type = "Jury Trial"

judge_prison$trial.type = "Bench Trial"
judge_no_prison$trial.type = "Bench Trial"

jury_trials =
  rbind(jury_prison,
        jury_no_prison)

bench_trials =
  rbind(judge_prison,
        judge_no_prison)

trials =
  rbind(bench_trials,
        jury_trials)

trials_table = 
  as.data.frame(table(trials$trial.type))

# Count of trials by type
ggplot(trials_table ,aes(x=Var1 , y=Freq, fill=Var1)) + 
  geom_bar(stat = 'identity', width=0.5) +
  scale_y_continuous(label = comma) +
  
  labs(x = "Trial Type", 
       y = "Count", 
       title = "Count of Trials by Trial Type",
       fill = "Legend") +
  theme_bw() +
  theme(text=element_text(family = "Times New Roman", 
                          face = "bold", 
                          size = 12,
                          hjust = 0.5),
        plot.title = element_text(hjust = 0.5)) +
  scale_fill_discrete(guide = FALSE)
```

![alt text10](https://github.com/ThomasPepperz/Using-ML-To-Detect-Bias-in-Criminal-Sentencing-Data/blob/master/count-trials-type.png)

```r
bench_trials_table = 
  as.data.frame(table(bench_trials$sentence.judge))

bench_trials_table = 
  bench_trials_table[bench_trials_table$Freq > 999,]

# Count of trials presided by judge with 1,000 or more trials
ggplot(bench_trials_table,aes(x=Var1 , y=Freq, fill=Var1)) + 
  geom_bar(stat = 'identity') +
  coord_flip() +
  scale_y_continuous(label = comma) +
  labs(x = "Name of Judge", 
       y = "Count of Trials", 
       title = "Count of Trials Presided Over Per Judge (Greater than 1,000)",
       fill = "Legend") +
  theme_bw() +
  theme(text=element_text(family = "Times New Roman", 
                          face = "bold", 
                          size = 12,
                          hjust = 0.5),
        plot.title = element_text(hjust = 0.5)) +
  scale_fill_discrete(guide = FALSE)
  ```

![alt text9](https://github.com/ThomasPepperz/Using-ML-To-Detect-Bias-in-Criminal-Sentencing-Data/blob/master/count-trials-judge.png)

# Detecting Bias with Machine Learning: Random Forests

## Methodology

The following section applies both linear regression and random forests to the data in order to model the data and examine it for the presence of bias in human decision making. 

Ordinary least-square linear regression in order to attempt to model the relationship between sentence length (the outcome variable) and several predictor variables such as `race`, `gender`, `age.at.incident`, and `class`. The coefficients produced by the model are of special interest as each is interpreted in order to determine the average expected value of a charge's resulting prison sentence length. Random forests are used to train a model to predict whether a charge woudl result in a prison or non-prison sentence for a defendant. The model's dependencies upon variables are examiend by producing a variable importance plot that displays the mean decrease in accuracy and the decrease in Gini impurity for each variable used in the model.

Each subset of data used is split into a training set (80% of the data) and a test set (20%) before each machine learning algorithm is applied using the following function:

```r
split_df <- function(dataframe, seed = 100) {
  if (!is.null(seed)) set.seed(seed)
  index <- 1:nrow(dataframe)
  trainindex <- sample(index, trunc(length(index) * 0.8))
  train <- dataframe[trainindex, ]
  test <- dataframe[-trainindex, ]
  list(train = train,test = test)
}
```

## Random Forests: Prison or No-Prison?

### Trials by Jury

```r
# Assign the results of partitioning the data to data frame 'split'
split = 
  split_df(jury_trials)

# Assign random sample to training set
train = 
  split$train

# Assign random sample to testing set
test = 
  split$test
 ```
 
 Explanation of jury trial model.

```r
#### Detecting Bias with Random Forests ####

set.seed(100)

# Jury Random Forest
jury_rf =
  randomForest(
  outcome ~
    race + 
    gender + 
    age.at.incident +
    primary.charge +
    class,
  importance = T,
  data = bench_trials, 
  ntree=500)
```

Analysis

```r
# Print out summary output from random forest
R> print(jury_rf)

Call:
 randomForest(formula = outcome ~ race + gender + age.at.incident +      primary.charge + class, data = jury_trials, importance = T,      ntree = 500) 
               Type of random forest: classification
                     Number of trees: 500
No. of variables tried at each split: 2

        OOB estimate of  error rate: 26.55%
Confusion matrix:
          Prison No Prison class.error
Prison       115       126   0.5228216
No Prison     41       347   0.1056701
```

```r
R> #Evaluate variable importance
R> importance(jury_rf)
                  Prison No Prison MeanDecreaseAccuracy MeanDecreaseGini
race            20.53392  12.58525             21.11962         17.37751
gender          23.91495  12.13507             22.26152         13.74164
age.at.incident 15.94157  15.69035             21.24803         53.14267
primary.charge  18.67515  25.68333             29.96176         12.27850
class           37.08941  28.94115             43.06146         37.81235
R> 
```

Analysis. Variable Importance Charts (Mean Decrease Accuracy & Gini Impurity Decrease) yields the following chart.

```r
# Generate variable importance charts
varImpPlot(jury_rf, col = c("blue","green","yellow4", "orange", "red"))
```

![alt text13](https://github.com/ThomasPepperz/Using-ML-To-Detect-Bias-in-Criminal-Sentencing-Data/blob/master/jury_rf_var_imp.png)

Out of Bag explanation

```r
# Out-of-Bag (OOB) Estimate of Error
error_df = 
  data.frame(error_rate = jury_rf$err.rate[, 'OOB'], 
             num_trees = 1:jury_rf$ntree)
             
# Create error rate chart
ggplot(error_df, aes(x=num_trees, y=error_rate)) +
  geom_line() +
  labs(x = "Number of Trees Grown", 
       y = "Error Rate", 
       title = "Progression of Error Rate By Increasing Number of Trees",
       fill = "Legend") +
  theme_bw() +
  theme(text=element_text(family = "Times New Roman", 
                          face = "bold", 
                          size = 12,
                          hjust = 0.5),
        plot.title = element_text(hjust = 0.5)) +
  scale_fill_discrete(guide = FALSE)
```

![alt text14](https://github.com/ThomasPepperz/Using-ML-To-Detect-Bias-in-Criminal-Sentencing-Data/blob/master/error-rate-num-trees.png)

```r
# Generate tree output for random forest object `jury_rf`
R> head(getTree(jury_rf, k=1, labelVar=T))
  left daughter right daughter       split var split point status prediction
1             2              3          gender         1.0      1       <NA>
2             4              5 age.at.incident        35.0      1       <NA>
3             6              7  primary.charge         0.5      1       <NA>
4             8              9           class      8189.0      1       <NA>
5            10             11 age.at.incident        54.0      1       <NA>
6            12             13 age.at.incident        45.5      1       <NA>
```

A visualization from a sample tree from the random forest:

```r
# Fetch tree and assign to tree object `tree_1`
tree_1 = 
  rpart(jury_rf$call, 
        data = 
          jury_trials,
        control = 
          rpart.control(
            minsplit = 20, 
            cp = 0))
            
# Generate tree visualization
prp(tree_1)   
```

![alt text17](https://github.com/ThomasPepperz/Using-ML-To-Detect-Bias-in-Criminal-Sentencing-Data/blob/master/jury-rf-tree-1.png)

### Trials by Judge (Bench Trials)




Note: Please report any bugs, coding errors, or broken web links to Thomas A. Pepperz at email thomaspepperz@icloud.com
