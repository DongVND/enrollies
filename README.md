# enrollies
# HR Analytics: Job Change of Data Scientists
I made a simple ETL process on data of enrollies

1. **Context and Content**
A company which is active in Big Data and Data Science wants to hire data scientists among people who successfully pass some courses which conduct by the company.
Many people signup for their training. Company wants to know which of these candidates are really wants to work for the company after training or looking for a new employment because it helps to reduce the cost and time as well as the quality of training or planning the courses and categorization of candidates.
Information related to demographics, education, experience are in hands from candidates signup and enrollment.

2. **A description of the data that used**:
1. Enrollies' data
As enrollies are submitting their request to join the course via Google Forms, we have the Google Sheet that stores data about enrolled students, containing the following columns:
enrollee_id: unique ID of an enrollee
full_name: full name of an enrollee
city: the name of an enrollie's city
gender: gender of an enrollee
The source: https://docs.google.com/spreadsheets/d/1VCkHwBjJGRJ21asd9pxW4_0z2PWuKhbLR3gUHm-p4GI/edit?usp=sharing

2. Enrollies' education
After enrollment everyone should fill the form about their education level. This form is being digitalized manually. Educational department stores it in the Excel format here: https://assets.swisscoding.edu.vn/company_course/enrollies_education.xlsx

This table contains the following columns:
enrollee_id: A unique identifier for each enrollee. This integer value uniquely distinguishes each participant in the dataset.
enrolled_university: Indicates the enrollee's university enrollment status. Possible values include no_enrollment, Part time course, and Full time course.
education_level: Represents the highest level of education attained by the enrollee. Examples include Graduate, Masters, etc.
major_discipline: Specifies the primary field of study for the enrollee. Examples include STEM, Business Degree, etc.

3. Enrollies' working experience
Another survey that is being collected manually by educational department is about working experience.
Educational department stores it in the CSV format here: https://assets.swisscoding.edu.vn/company_course/work_experience.csv
This table contains the following columns:
enrollee_id: A unique identifier for each enrollee. This integer value uniquely distinguishes each participant in the dataset.
relevent_experience: Indicates whether the enrollee has relevant work experience related to the field they are currently studying or working in. Possible values include Has relevent experience and No relevent experience.
experience: Represents the number of years of work experience the enrollee has. This can be a specific number or a range (e.g., >20, <1).
company_size: Specifies the size of the company where the enrollee has worked, based on the number of employees. Examples include 50−99, 100−500, etc.
company_type: Indicates the type of company where the enrollee has worked. Examples include Pvt Ltd, Funded Startup, etc.
last_new_job: Represents the number of years since the enrollee's last job change. Examples include never, >4, 1, etc.

4. Training hours
Database credentials:
Database type: MySQL
Host: 112.213.86.31
Port: 3360
Login: etl_practice
Password: 550814
Database name: company_course
Table name: training_hours

5. City development index
Another source that can be usefull is the table of City development index.
The City Development Index (CDI) is a measure designed to capture the level of development in cities. It may be significant for the resulting prediction of student's employment motivation.
It is stored here: https://sca-programming-school.github.io/city_development_index/index.html

6. Employment
To retrieve the fact of employment. If student is marked as employed, it means that this student started to work in our company after finishing the course.

Database credentials:
Database type: MySQL
Host: 112.213.86.31
Port: 3360
Login: etl_practice
Password: 550814
Database name: company_course
Table name: employment

4. **Results**
We processed and output the datasets.


#### Process explained step-by-step:
###Part 1. Extract - gather data from multiple sources 
##Step1. Import Pandas to load data to a dataframe:
```python:
import pandas as pd
```
#extract enrollies data from google sheet:
```python:
google_sheet_id = '1VCkHwBjJGRJ21asd9pxW4_0z2PWuKhbLR3gUHm-p4GI'
url='https://docs.google.com/spreadsheets/d/' + google_sheet_id + '/export?format=xlsx'
enrollies_df = pd.read_excel(url, sheet_name ='enrollies')
```
#extract enrollies education and work experience from excel file and csv file accordingly:
```python:
enrollies_education_df = pd.read_excel('/content/enrollies_education.xlsx',sheet_name= 'enrollies_education')
work_experience_df = pd.read_csv('/content/work_experience.csv')
```
#import SQLAlchemy and install pymysql to load data from a MySQL database:
```python:
from sqlalchemy import create_engine
!pip install pymysql
```
create connection to a database named company_course:
- Driver: mysql+pymysql
- Login: etl_practice
- Password: 550814
- Host: 112.213.86.31
- Port: 3360
- Database Name: company_course
```python:
engine = create_engine('mysql+pymysql://etl_practice:550814@112.213.86.31:3360/company_course')
```
create dataframe from table1 named [training_hours]:
```python:
training_hours_df = pd.read_sql_table("training_hours", con=engine)
```
create dataframe from table2 named [employment]:
```python:
employment_df = pd.read_sql_table("employment", con=engine)
```

Load the first table from a web page for city development info:
```python:
page_url = "https://sca-programming-school.github.io/city_development_index/index.html"
city_development_index_df = pd.read_html(page_url)[0]
```

### Transform data with the following steps:
- fixing data types
- handling missing value 
- removing duplicate
- unifying (consistency) format

## First, let's start with enrollies data (enrollies_df)

check the info of enrollies data:
```
enrollies_df.info()
```
```
We got the result as:
RangeIndex: 19158 entries, 0 to 19157
Data columns (total 4 columns):
 #   Column       Non-Null Count  Dtype 
---  ------       --------------  ----- 
 0   enrollee_id  19158 non-null  int64 
 1   full_name    19158 non-null  object
 2   city         19158 non-null  object
 3   gender       14650 non-null  object
dtypes: int64(1), object(3)
```
--> there are 2 issues: missing values for gender and Dtype of (full_name, city and gender) are object.
Let's fill missing value and change columns 1-2 into string dtypes, while columns 3 should turn into category dtypes.

```
#fix data type for the enrollies_df full_name and city columns
enrollies_df.full_name = enrollies_df.full_name.astype('string')
enrollies_df.city = enrollies_df.city.astype('string')
```
```
#fill missing value for the enrollies_df.gender column
enrollies_df.gender = enrollies_df.gender.fillna(enrollies_df.gender.mode()[0])
enrollies_df.gender = enrollies_df.gender.astype('category')
```

The result after our fixes:
```
RangeIndex: 19158 entries, 0 to 19157
Data columns (total 4 columns):
 #   Column       Non-Null Count  Dtype   
---  ------       --------------  -----   
 0   enrollee_id  19158 non-null  int64   
 1   full_name    19158 non-null  string  
 2   city         19158 non-null  string  
 3   gender       19158 non-null  category
dtypes: category(1), int64(1), string(2)
```
As in the above info, data is fulfilled and dtypes are as supposed.

Next, let's check for duplicates in enrollies_df, if none we can skip:
```
enrollies_df.duplicated().sum()
```
-> The result is (0), so we don't have to remove duplicates.


## Now, we move on to the next set, enrollies_education.df
(Repeat the same steps as enrollies_df)
- Check info of the dataframe:
```
enrollies_education.info()
```
The result show us there are missing value for enrolled_university, education_level and major_discipline.
Also, their data types are not ideal.
```
RangeIndex: 19158 entries, 0 to 19157
Data columns (total 4 columns):
 #   Column               Non-Null Count  Dtype 
---  ------               --------------  ----- 
 0   enrollee_id          19158 non-null  int64 
 1   enrolled_university  18772 non-null  object
 2   education_level      18698 non-null  object
 3   major_discipline     16345 non-null  object
dtypes: int64(1), object(3)
```
Let's start with filling the missing value, those should be 'unknown'
```
enrollies_education.enrolled_university = enrollies_education.enrolled_university.fillna('unknown')
enrollies_education.education_level = enrollies_education.education_level.fillna('unknown')
enrollies_education.major_discipline = enrollies_education.major_discipline.fillna('unknown')
```
Those columns are also need to be set dtypes as 'category' for optimization.
```
cat_cols = ['enrolled_university','education_level','major_discipline']
enrollies_education[cat_cols] = enrollies_education[cat_cols].astype('category')
```

The result should be as below:
```
RangeIndex: 19158 entries, 0 to 19157
Data columns (total 4 columns):
 #   Column               Non-Null Count  Dtype   
---  ------               --------------  -----   
 0   enrollee_id          19158 non-null  int64   
 1   enrolled_university  19158 non-null  category
 2   education_level      19158 non-null  category
 3   major_discipline     19158 non-null  category
dtypes: category(3), int64(1)
```

### For the training_hours_df, employment_df and city_development_df, We didn't find any issue after checking.
We may use these following codes to check:
```
#checking traning hours data:
training_hours_df.info()
#checking employment data:
employment_df.info()
#checking city development data:
city_development_index_df.info()
```

#### Final steps, LOADING data to database:
```
#creat a new database and engine to load the newly processed data
db= 'data_warehouse.db'
target_db_engine = create_engine('sqlite:///data_warehouse.db')

#using method pandas.DataFrame.to_sql to load them:
employment_df.to_sql('Fact_employment', target_db_engine, if_exists='replace',index=False)
enrollies_df.to_sql('dim_enroll',target_db_engine, if_exists='replace',index=False)
enrollies_education_df.to_sql('dim_edu',target_db_engine, if_exists='replace',index=False)
training_hours_df.to_sql('dim_training_hours',target_db_engine, if_exists='replace',index=False)
work_experience_df.to_sql('dim_work_exp',target_db_engine, if_exists='replace',index=False)
city_development_index_df.to_sql('dim_city_dev',target_db_engine, if_exists='replace',index=False)
```

###$ We completed the ETL process of the datasets.




