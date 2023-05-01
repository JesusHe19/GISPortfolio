### Import libraries


```python
import arcgis
import pandas as pd 
import os
import arcpy
import re
import numpy as np
import matplotlib.pyplot as plt
import difflib
```

## Clean and prepare the data from AAPP, i.e. the dataset that contains information about victims

### Load table data


```python
#Remember that the table data has to be located in the same folder
#as your arcgis project
current_path = os.getcwd()
print("The current working directory (where file is located):\n", current_path)
```

    The current working directory (where file is located):
     C:\Users\JesusHerrera\Desktop\SEMESTER4\Advanced GIS\FinalProject_jeherrer\Final_Project_jeherrer
    


```python
#Read the excel file as a pandas dataframe
victims_df = pd.read_excel("Victims_data.xlsx")

#Get a quick view of the dataframe, print the first 10 rows
victims_df.head(10)
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
      <th>#</th>
      <th>link_name</th>
      <th>id_number</th>
      <th>name</th>
      <th>name_in_burmese</th>
      <th>gender</th>
      <th>age</th>
      <th>parent's_name</th>
      <th>sector</th>
      <th>category</th>
      <th>township_town</th>
      <th>state_region</th>
      <th>deceased_date_view</th>
      <th>#.1</th>
      <th>special_condition_L</th>
      <th>event_detail_killed_L</th>
      <th>killed_event_type_L</th>
      <th>verification</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>15672_TheinWin</td>
      <td>THE-20220601-00055</td>
      <td>Thein Win</td>
      <td>သနိ  း် ဝငး်</td>
      <td>M</td>
      <td>42</td>
      <td>Unknown</td>
      <td>Civilian</td>
      <td>Unknown</td>
      <td>Lanmadaw</td>
      <td>Yangon</td>
      <td>2021-05-20 00:00:00</td>
      <td>1</td>
      <td>Detainment Prison</td>
      <td>He is living at (10) Street in Lanmadaw in Yan...</td>
      <td>Deceased</td>
      <td>Verified</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>15926_PyayThein</td>
      <td>PYA-20220601-00309</td>
      <td>Pyay Thein</td>
      <td>ေြပသနိ  း်</td>
      <td>M</td>
      <td>29</td>
      <td>Unknown</td>
      <td>Civilian</td>
      <td>Unknown</td>
      <td>Unknown</td>
      <td>Mon</td>
      <td>2021-08-04 00:00:00</td>
      <td>2</td>
      <td>Detainment Prison</td>
      <td>Pyay Thein, who was arrested while protesting ...</td>
      <td>Deceased</td>
      <td>Verified</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>16173_MyintWai</td>
      <td>MYI-20220601-00556</td>
      <td>Myint Wai</td>
      <td>ြမငေ့  ဝ</td>
      <td>M</td>
      <td>53</td>
      <td>Unknown</td>
      <td>Civilian</td>
      <td>Unknown</td>
      <td>Unknown</td>
      <td>Yangon</td>
      <td>2021-04-01 00:00:00</td>
      <td>3</td>
      <td>Detainment Prison</td>
      <td>Myint Wai, living in Thanlyin Township in Yang...</td>
      <td>Deceased</td>
      <td>Verified</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>16433_PhoeToke(aka)MyoK haingTun</td>
      <td>PHO-20220601-00816</td>
      <td>Phoe Toke (aka) Myo Khaing Tun</td>
      <td>ဖိ,းတ,တ် (ခ) မျိµးခငိ,  ထ်    ွနး်</td>
      <td>M</td>
      <td>44</td>
      <td>U Taw</td>
      <td>Education</td>
      <td>Teacher</td>
      <td>Unknown</td>
      <td>Magway</td>
      <td>2021-05-21 00:00:00</td>
      <td>4</td>
      <td>Detainment Prison</td>
      <td>A private boarding school principal from Myo T...</td>
      <td>Deceased</td>
      <td>Verified</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>16466_InHoke</td>
      <td>INH-20220601-00849</td>
      <td>In Hoke</td>
      <td>အငဟ်    ,တ်</td>
      <td>M</td>
      <td>68</td>
      <td>Unknown</td>
      <td>Activist</td>
      <td>Former Political Prisoner</td>
      <td>Unknown</td>
      <td>Ayeyarwady</td>
      <td>2021-10-26 00:00:00</td>
      <td>5</td>
      <td>Detainment Prison</td>
      <td>He was arrested on 3 June 2021. He died due to...</td>
      <td>Deceased</td>
      <td>Verified</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>16483_MaungMaungNyein Tun</td>
      <td>MAU-20220601-00866</td>
      <td>Maung Maung  Nyein Tun</td>
      <td>ေမာငေမာင(ငမိ  ်းထွနး်</td>
      <td>M</td>
      <td>45</td>
      <td>Unknown</td>
      <td>Medic</td>
      <td>Doctor</td>
      <td>Unknown</td>
      <td>Mandalay</td>
      <td>2021-08-08 00:00:00</td>
      <td>6</td>
      <td>Detainment Prison</td>
      <td>Doctor Maung Maung Nyein Tun, lecturer of surg...</td>
      <td>Deceased</td>
      <td>Verified</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>16492_MoeThu(aka)MoeTh uzarHtwe</td>
      <td>MOE-20220601-00875</td>
      <td>Moe Thu (aka) Moe Thuzar Htwe</td>
      <td>မ,ိးသ„ (ခ) မ,ိးသ„ဇာေထွး</td>
      <td>F</td>
      <td>42</td>
      <td>Unknown</td>
      <td>Civilian</td>
      <td>Volunteer</td>
      <td>Unknown</td>
      <td>Yangon</td>
      <td>2021-07-21 00:00:00</td>
      <td>7</td>
      <td>Detainment Prison</td>
      <td>Moe Thu a.k.a. Moe Thuzar Htwe, a resident fro...</td>
      <td>Deceased</td>
      <td>Verified</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8</td>
      <td>16493_AyeHla</td>
      <td>AYE-20220601-00876</td>
      <td>Aye Hla</td>
      <td>ေအးလ,</td>
      <td>M</td>
      <td>72</td>
      <td>U Phoe Kune</td>
      <td>Parties</td>
      <td>National League for De…</td>
      <td>Kyauktaga</td>
      <td>Bago</td>
      <td>2021-07-19 00:00:00</td>
      <td>8</td>
      <td>Detainment Prison</td>
      <td>Aye Hla, Vice Chairman of NLD of Kyauktaga Tow...</td>
      <td>Deceased</td>
      <td>Verified</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9</td>
      <td>16550_MaungHtay</td>
      <td>MAU-20220601-00933</td>
      <td>Maung Htay</td>
      <td>ေမာငေဌး</td>
      <td>M</td>
      <td>55</td>
      <td>Unknown</td>
      <td>Civilian</td>
      <td>Unknown</td>
      <td>Myingyan</td>
      <td>Mandalay</td>
      <td>2021-08-09 00:00:00</td>
      <td>9</td>
      <td>Detainment Prison</td>
      <td>Maung Htay from Kywe Chan Village in Myingyan ...</td>
      <td>Deceased</td>
      <td>Verified</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>16556_TunSein</td>
      <td>TUN-20220601-00939</td>
      <td>Tun Sein</td>
      <td>ထွနး် စိန်</td>
      <td>M</td>
      <td>Unkno…</td>
      <td>U Yan Shin</td>
      <td>Civilian</td>
      <td>Unknown</td>
      <td>Unknown</td>
      <td>Yangon</td>
      <td>2021-07-14 00:00:00</td>
      <td>10</td>
      <td>Detainment Prison</td>
      <td>He died while receiving medical treatment at a...</td>
      <td>Deceased</td>
      <td>Verified</td>
    </tr>
  </tbody>
</table>
</div>




```python
for col in victims_df.columns:
    print(col)
```

    #
    link_name
    id_number
    name
    name_in_burmese
    gender
    age
    parent's_name
    sector
    category
    township_town
    state_region
    deceased_date_view
    #.1
    special_condition_L
    event_detail_killed_L
    killed_event_type_L
    verification
    

### Inspect the columns and create a data dictionary

When cleaning datasets, it is important to understand what each column refers to<br>

#': Indicates the row number <br>

link_name: It is not clear what this variable indicates, it contains the name of the victim and a number. The AAPP website do not provide any information about the meaning of this variable. This variable will not be used in the analysis <br>

id_number: It is the unique identifier of the victim <br>

name: Name of the victim<br>

name in burmese: Name of the victim in Burmese language <br>

gender: Whether the victim's gender is Male (M); Female (F); Unknown; or LGBT <br>

age: Age of the victim <br>

parent's_name: Name of victim's parent. In Myanmar people do not have last name, therefore the parents name is used for identifying people  <br>

sector: It indicates to the working sector a victim belongs to<br>

category: It indicates the occupation of the victim<br>

township_town: Myanmar is divided into regions, states, districts, townships and villages <br>

state_region: Myanmar is divided into regions, states, districts, townships and villages <br>

deceased_date_view: Date the victim was killed <br>

#.1: No special meaning

special_condition_L: Relevant information about the victim <br>

event_detail_killed_L: Description on how the victim was murdered<br>

killed_event_type_L: Indicates type of victim, i.e. deceased <br>

verification: Indicates whether the event was verified, i.e. the event did occur <br>

Sources:<br>
Scott, James G. Law and Custom in Burma and the Burmese Family. Oxford University Press, 1963. <br>

Assistance Association for Political Prisoners, "Daily Briefing Since Coup", Accessed April 28, 2023, https://aappb.org/?cat=109<br>


### Drop columns not necessary for the analysis


```python
#Drop the column that indicates name in Burmese, the column #.1, and the link_name
victims_df = victims_df.drop(['#', "#.1", "link_name", 'name_in_burmese'], axis = 1)
```


```python
#AAPP classifies the victims of the Myanmar coup into: 
#deceased, prisoners, and detainees
#This dataset is only for deceased, therefore the column killed_event_type_L
#can be eliminated

victims_df = victims_df.drop(['killed_event_type_L'], axis = 1)
```

### Modify column names to make them more understandable


```python
victims_df = victims_df.rename(columns={'event_detail_killed_L': 'event_details', 
                                        'special_condition_L' : 'special_condition',
                                         'deceased_date_view' : 'event_date'})

```

### Inspect each categorical column and verify the information is correctly encoded
#### It will be necessary to count the number of times each unique value of a column appear, and also count the missing values


```python
def inspect(col):
    print("Column:", col, "\n")
    print("Count of unique values")
    print(victims_df[col].value_counts())
    print("\nThe number of missing values is:", str(victims_df["gender"].isnull().sum()))

```


```python
categorical_vars = ["gender", "sector", "category", "special_condition", "verification"]
for var in categorical_vars:
    inspect(var)
    print("\n")
```

    Column: gender 
    
    Count of unique values
    M          2695
    F           430
    Unknown     114
    LGBT          1
    Name: gender, dtype: int64
    
    The number of missing values is: 0
    
    
    Column: sector 
    
    Count of unique values
    Civilian                                          2763
    Resistance Group                                   176
    Education                                          154
    Parties                                             44
    Religious                                           22
    Public Service                                      17
    Medic                                               16
    Activist                                            10
    Artist                                               7
    NGO                                                  6
    Security Force                                       4
    Resistance Group\nEducation                          4
    Civilain                                             4
    Media                                                3
    Parties\nResistance Group                            2
    Education        Artist                              1
    Parties        NGO                                   1
    Member of Parliament\nParties                        1
    Media        Artist                                  1
    Member of Parliament Parties\nResistance Group       1
    Parties        Activist Artist                       1
    Activist        Artist                               1
    NGO        Activist                                  1
    Name: sector, dtype: int64
    
    The number of missing values is: 0
    
    
    Column: category 
    
    Count of unique values
    Unknown                                        2560
    Student                                         116
    People's Defence Force                           93
    Internally Displaced Pe…                         87
    National League for De…                          43
                                                   ... 
    Journalist        Poet                            1
    Freelance Photo Journ…                            1
    Ministry of Cooperativ… Deputy Officer\nCDM       1
    Owner of Htike Htike P…                           1
    Soldier                                           1
    Name: category, Length: 137, dtype: int64
    
    The number of missing values is: 0
    
    
    Column: special_condition 
    
    Count of unique values
    Shot                                                    700
    Detainment                                              535
    Artillery                                               292
    Protest crackdown\nShot                                 144
    Airstrike                                               143
                                                           ... 
    Detainment\nHuman shield Shot in head                     1
    Detainment Human shield\nShot in head                     1
    Detainment\nPhysical disability Mental illness            1
    Detainment Human shield Shot in head\nMental illness      1
    Detainment\nSet fire alive                                1
    Name: special_condition, Length: 86, dtype: int64
    
    The number of missing values is: 0
    
    
    Column: verification 
    
    Count of unique values
    Verified      2968
    Unverified     272
    Name: verification, dtype: int64
    
    The number of missing values is: 0
    
    
    

From the categorical variables, it can be seen that none of them have missing values, there is no need to handle missing data for these variables. <br> 
However, the sector, category, and special_condition columns need to be cleaned. There is an issue on how the information is encoded. <br>

Unfortunately, the string characters contained in the columns sector, category, and special condition cannot be fully downloaded from the AAPP open database. The data from AAPP cannot be downloaded as Excel file, it can only downloaded as a PDF. The predetermined width of the column cannot be modified, hence when the database is downloaded as a PDF file, the column widht is too narrow to fully contain all the string characters inside each row. <br>

The following example will explain the issue: <br>
* 'National League for De…' The string character for the category "National League for Democracy"
*  'Regional Hluttaw\nNational League for De…' The string character contains more than one category, categories are separated by an empty space, furthermore the string character is too long.

The solution for this issue: <br>
* Contact AAPP and request the dataset as an Excel file, this has already been done but no response has been received since 2 weeks ago. 
* For each string value, take only the first category, e.g., in this case 'Regional Hluttaw\nNational League for De…', the value will be modified to contain only "Regional Hluttaw". If after taking the first category, the category is still incomplete, the website of AAPP will be manually inspected to identify the correct category, e.g., after inspecting manuall the website it is conclude that 'Internally Displaced People…'  refers to "Internally Displaced People"


### Solve the issue in the sector, category, and special condition columns

#### Category column


```python
#convert to string type
victims_df["category"] = victims_df["category"].astype(str)
```


```python
#Eliminate everything what is after the first line break ""\n"
victims_df["category"] = victims_df["category"].str.split("\n").str[0]
```


```python
#Lets create an empty dictionary that contains all
#cases that look like this: "Teacher      Writer"
#The dictionary will help to update the column

long_spaces_str_cases_dict = {} #Create empty dictionary

for case in victims_df["category"].unique(): #iterate through all unique vals
    if re.search(r"\s{3,}", case): #find the cases
        first_word = case.split(" ")[0] #Take only the first word
        long_spaces_str_cases_dict[case] = first_word #Create the dictionary
```


```python
#Lets retrieve all cases where the string is incomplete
for case in victims_df["category"].unique():
    if re.search(r"…", case):
        print("\'"+case+'\'' +  " :")
```

    'National League for De…' :
    'Chin National Defence …' :
    'Ministry of Environmen… Forestry Officer' :
    'Ministry of Agriculture,…' :
    'City Development Com…' :
    'Ministry of Electricity a…' :
    'Former Ward and Villa…' :
    'Former Hundred-Hous…' :
    'Internally Displaced Pe…' :
    'Former Member of Nat…' :
    'Owner of Htike Htike P…' :
    'Ministry of Cooperativ… Deputy Officer' :
    'Freelance Photo Journ…' :
    'Ministry of Environmen…' :
    'Karenni National Peopl…' :
    'Ministry of Rail Transp…' :
    'Village Self-Defense Te…' :
    'General Administration…' :
    'Local People's Defence…' :
    'Former Chairman of All…' :
    'Ward and Village Admi…' :
    'Karenni Democratic Fr…' :
    'Free Funeral Service So…' :
    'Monywa People Defen…' :
    'Kachin Independence …' :
    'Karenni People's Defen…' :
    'Former Member of Pat…' :
    'Farmer’s Independence…' :
    'Kanbalu Underground …' :
    'Urban Underground R…' :
    'Salingyi People's Defen…' :
    'Kyaukse District People…' :
    'People's Defence Force…' :
    'Rubber Plantation Wor…' :
    


```python
#Find all those cases in the website and complete
incomplete_str_cases_dict = {'National League for De…' : 'National League for Democracy',
'Chin National Defence …' : 'Chin National Defence Force',
'Ministry of Environmen… Forestry Officer' : 'Ministry of Environmental Conservation and Forestry',
'Ministry of Agriculture,…' : 'Ministry of Agriculture, Livestock and Irrigation',
'City Development Com…' : 'City Development Committee',
'Ministry of Electricity a…' : 'Ministry of Electricity and Energy',
'Former Ward and Villa…' : 'Former Ward and Village Administrator',
'Former Hundred-Hous…' : 'Former Hundred-Household Administrator',
'Internally Displaced Pe…' : 'Internally Displaced People',
'Former Member of Nat…' : 'Former Member of National League for Democracy Party',
'Owner of Htike Htike P…' : 'Owner of Htike Htike Pure Water Factory',
'Ministry of Cooperativ… Deputy Officer' : 'Ministry of Cooperatives and Rural Development',
'Freelance Photo Journ…' : 'Freelance Photo Journalist',
'Ministry of Environmen…' : 'Ministry of Environmental Conservation and Forestry',
'Karenni National Peopl…' : 'Karenni National People\'s Liberation Front (Kalalata)',
'Ministry of Rail Transp…' : 'Ministry of Rail Transportation',
'Village Self-Defense Te…' : 'Village Self-Defense Team Leader',
'General Administration…' : 'General Administration Department',
'Local People\'s Defence…' : 'Local People\'s Defence Force',
'Former Chairman of All…' : 'Former Chairman of All Burma Federation of Student Union',
'Ward and Village Admi…' : 'Ward and Village Administrator',
'Karenni Democratic Fr…' : 'Karenni Democratic Front',
'Free Funeral Service So…' : 'Free Funeral Service Society',
'Monywa People Defen…' : 'Monywa People Defence’s Team',
'Kachin Independence …' : 'Kachin Independence Organization',
'Karenni People\'s Defen…' : 'Karenni People\'s Defence Force',
'Former Member of Pat…' : 'Former Member of Pathein Western Defense Force',
'Farmer’s Independence…' : 'Farmer’s Independence Force',
'Kanbalu Underground …' : 'Kanbalu Underground Warriors',
'Urban Underground R…' : 'Urban Underground Revolution Force',
'Salingyi People\'s Defen…' : 'Salingyi People\'s Defence Force',
'Kyaukse District People…' : 'Kyaukse District People\'s Defence Force',
'People\'s Defence Force…' : 'People\'s Defence Force',
'Rubber Plantation Wor…' : 'Rubber Plantation Worker'}
```


```python
#Update the column values
victims_df["category"] = victims_df["category"].map(long_spaces_str_cases_dict).fillna(victims_df["category"])
victims_df["category"] = victims_df["category"].map(incomplete_str_cases_dict).fillna(victims_df["category"])

```

#### Sector column


```python
#Convert to string type
victims_df["sector"] = victims_df["sector"].astype(str)
```


```python
#Eliminate everything what is after the first line break ""\n"
victims_df["sector"] = victims_df["sector"].str.split("\n").str[0]
```


```python
#Lets create an empty dictionary that contains all
#cases that look like this: "Teacher      Writer"
#The dictionary will help to update the column

long_spaces_str_cases_dict = {} #Create empty dictionary

for case in victims_df["sector"].unique(): #iterate through all unique vals
    if re.search(r"\s{3,}", case): #find the cases
        first_word = case.split(" ")[0] #Take only the first word
        long_spaces_str_cases_dict[case] = first_word #Create the dictionary
```


```python
#Lets retrieve all cases where the string is incomplete
#there are no cases like this
for case in victims_df["sector"].unique():
    if re.search(r"…", case):
        print("\'"+case+'\'' +  " :")
```


```python
#Update the column values
victims_df["sector"] = victims_df["sector"].map(long_spaces_str_cases_dict).fillna(victims_df["sector"])

```


```python
#There are typos to fix
fix_typos_dict = {'Member of Parliament Parties' : 'Member of Parliament',
'Civilain' : 'Civilian'}

victims_df["sector"] = victims_df["sector"].map(fix_typos_dict).fillna(victims_df["sector"])
```


```python
print(victims_df["sector"].unique())
```

    ['Civilian' 'Education' 'Activist' 'Medic' 'Parties'
     'Member of Parliament' 'Resistance Group' 'NGO' 'Artist' 'Public Service'
     'Security Force' 'Religious' 'Media']
    

The unique values of sector correspond to the data dictionary that AAPP has in their website:

Assistance Association for Political Prisoners, "Sector Data Definition", Accessed April 28, 2023, https://aappb.org/wp-content/uploads/2022/06/Sector_Data_Definition-2-Feb-2023.pdf

#### Special condition column


```python
victims_df["special_condition"] = victims_df["special_condition"].astype(str)
```


```python
#Eliminate everything what is after the first line break ""\n"
victims_df["special_condition"] = victims_df["special_condition"].str.split("\n").str[0]
```


```python
#Lets create an empty dictionary that contains all
#cases that look like this: "Teacher      Writer"
#The dictionary will help to update the column

long_spaces_str_cases_dict = {} #Create empty dictionary

for case in victims_df["special_condition"].unique(): #iterate through all unique vals
    if re.search(r"\s{3,}", case): #find the cases
        first_word = case.split(" ")[0] #Take only the first word
        long_spaces_str_cases_dict[case] = first_word #Create the dictionary
```


```python
#Lets retrieve all cases where the string is incomplete
#there are no cases like this
for case in victims_df["special_condition"].unique():
    if re.search(r"…", case):
        print("\'"+case+'\'' +  " :")
```


```python
#Update the column values
victims_df["special_condition"] = victims_df["special_condition"].map(long_spaces_str_cases_dict).fillna(victims_df["special_condition"])
```

The unique values of sector correspond to the data dictionary that AAPP has in their website:

Assistance Association for Political Prisoners, "Sector Data Definition", Accessed April 28, 2023, https://aappb.org/wp-content/uploads/2022/06/Special_Condition_Data_Definition-2-Feb-2023.pdf

### Clean the age column


```python
victims_df["age"] = victims_df["age"].astype(str)
```


```python
victims_df["age"].unique()
```




    array(['42', '29', '53', '44', '68', '45', '72', '55', 'Unkno…', '79',
           '65', '75', '26', '33', '36', '34', '2', '50', '32', '21', '18',
           '37', '16', '48', '30', '35', '39', '23', '59', '20', '40', '38',
           '19', '22', '17', '25', '47', '58', '27', '62', '31', '46', '24',
           '51', '41', '56', '43', '15', '28', '57', '54', '78', '52', '67',
           '70', '14', '66', '7', '60', '49', '13', '9', '11', '64', '63',
           '10', '90', '87', '18 mo…', '82', '1', '8', '50+', '20+', '-18',
           '76', '5', '40+', '61', '12', '80', '77', '3', '4', '93', '6',
           '84', '83', '14 mo…', '4+', '88', '86', '6+', '60+', '30+', '92',
           '6 mont…', '85', '7 mont…', '80+', '70+', '73', '2 years…', '69',
           '81', '11 mo…'], dtype=object)




```python
#lets see the count for each age value
age_counts_df = victims_df["age"].value_counts().to_frame().reset_index()
age_counts_df.columns = ['age', 'count']
age_counts_df
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
      <th>age</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Unkno…</td>
      <td>771</td>
    </tr>
    <tr>
      <th>1</th>
      <td>40</td>
      <td>131</td>
    </tr>
    <tr>
      <th>2</th>
      <td>30</td>
      <td>125</td>
    </tr>
    <tr>
      <th>3</th>
      <td>20</td>
      <td>103</td>
    </tr>
    <tr>
      <th>4</th>
      <td>50</td>
      <td>91</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>101</th>
      <td>-18</td>
      <td>1</td>
    </tr>
    <tr>
      <th>102</th>
      <td>6+</td>
      <td>1</td>
    </tr>
    <tr>
      <th>103</th>
      <td>86</td>
      <td>1</td>
    </tr>
    <tr>
      <th>104</th>
      <td>88</td>
      <td>1</td>
    </tr>
    <tr>
      <th>105</th>
      <td>11 mo…</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>106 rows × 2 columns</p>
</div>




```python
#There are some cases that have AGE+, there are very few of them 
#We can eliminate the "+" and assume they refer to the age
victims_df["age"] = victims_df["age"].str.replace("+", "", regex = True)
victims_df["age"] = victims_df["age"].str.replace("-", "", regex = True)
```


```python
#Substitute "Unknown" with nulls
victims_df["age"].replace("Unkno…", np.nan, inplace=True)
#Replace other string vals
victims_df["age"].replace("2 years…", "2", inplace=True)
#Everytime to "mo" str appears, convert to 1 year
victims_df["age"] = victims_df["age"].str.replace(r"(\d+) mo.*", "1", regex=True)
```


```python
victims_df["age"] = victims_df["age"].astype(float)
```


```python
# Create distribution histogram
victims_df["age"].hist(bins=20)

# Set plot labels
plt.xlabel("Age")
plt.ylabel("Count")
plt.title("Age Distribution")
plt.show()

```

There are not outlier values in the Age column, the column is clean

## Verify that the township names of the AAPP dataset (victims) match the township names of the township shapefile

#### Read the townships from the feature table of shapefile


```python
township_df = pd.read_excel("Townships_data.xlsx")
```


```python
shapefile_list = list(township_df["TS"].unique())
print("This is a list of all townships in the shapefile")
for town in shapefile_list:
    print(town)
```

    This is a list of all townships in the shapefile
    Hinthada
    Ingapu
    Kyangin
    Lemyethna
    Myanaung
    Zalun
    Labutta
    Mawlamyinegyun
    Danubyu
    Maubin
    Nyaungdon
    Pantanaw
    Einme
    Myaungmya
    Wakema
    Kangyidaunt
    Kyaunggon
    Kyonpyaw
    Ngapudaw
    Pathein
    Thabaung
    Yegyi
    Bogale
    Dedaye
    Kyaiklat
    Pyapon
    Bago
    Daik-U
    Kawa
    Kyauktaga
    Waingmaw
    Khaunglanhpu
    Machanbaw
    Nawngmun
    Puta-O
    Sumprabum
    Kyauktada
    Nyaunglebin
    Shwegyin
    Thanatpin
    Thandaunggyi
    Hpapun
    Kawkareik
    Waw
    Kyainseikgyi
    Myawaddy
    Gangaw
    Saw
    Latha
    Mayangone
    Htantabin
    Kyaukkyi
    Oktwin
    Phyu
    Taungoo
    Yedashe
    Padaung
    Paukkhaung
    Paungde
    Ngape
    Pwintbyu
    Salin
    Sidoktaya
    Myaing
    Pabedan
    Pyay
    Shwedaung
    Aungmyaythazan
    Thegon
    Gyobingauk
    Letpadan
    Chanayethazan
    Minhla
    Monyo
    Nattalin
    Okpho
    Thayarwady
    Kyaikmaraw
    Mawlamyine
    Mudon
    Zigon
    Falam
    Tedim
    Tonzang
    Hakha
    Thantlang
    Matupi
    Paletwa
    Kanpetlet
    Mindat
    Bhamo
    Mansi
    Momauk
    Shwegu
    Hpakant
    Mogaung
    Mohnyin
    Chipwi
    Injangyang
    Myitkyina
    Tanai
    Thanbyuzayat
    Ye
    Bilin
    Tsawlaw
    Bawlake
    Hpasawng
    Tatkon
    Zay Yar Thi Ri
    Ann
    Kyaukpyu
    Munaung
    Ramree
    Buthidaung
    Maungdaw
    Kyauktaw
    Minbya
    Mese
    Mrauk-U
    Myebon
    Pauktaw
    Ponnagyun
    Rathedaung
    Demoso
    Hpruso
    Loikaw
    Shadaw
    Hlaingbwe
    Hpa-An
    Tilin
    Chauk
    Magway
    Myothit
    Natmauk
    Taungdwingyi
    Taze
    Wetlet
    Ye-U
    Tamu
    Kani
    Pale
    Yenangyaung
    Minbu
    Pakokku
    Pauk
    Seikphyu
    Monghsat
    Mongton
    Monghpyak
    Mongyawng
    Yesagyo
    Aunglan
    Kamma
    Mindon
    Tachileik
    Hopang
    Mongmao
    Sinbaungwe
    Mongyai
    Tangyan
    Thayet
    Kyaukse
    Myittha
    Matman
    Sintgaing
    Tada-U
    Amarapura
    Chanmyathazi
    Mahaaungmyay
    Patheingyi
    Pyigyitagon
    Narphan
    Pangsang
    Mabein
    Mahlaing
    Mongmit
    Kutkai
    Muse
    Namhkan
    Manton
    Namhsan
    Meiktila
    Thazi
    Wundwin
    Palaw
    Tanintharyi
    Myingyan
    Natogyi
    Ngazun
    Taungtha
    Kyaukpadaung
    Nyaung-U
    Madaya
    Mogoke
    Pyinoolwin
    Singu
    Thabeikkyin
    Pyawbwe
    Yamethin
    Chaungzon
    Kyaikto
    Paung
    Thaton
    Det Khi Na Thi Ri
    Lewe
    Pyinmana
    Za Bu Thi Ri
    Oke Ta Ra Thi Ri
    Poke Ba Thi Ri
    Sittwe
    Gwa
    Thandwe
    Toungup
    Hkamti
    Homalin
    Kale
    Kalewa
    Mingin
    Kanbalu
    Kyunhla
    Banmauk
    Indaw
    Katha
    Tigyaing
    Kawlin
    Pinlebu
    Wuntho
    Mawlaik
    Paungbyin
    Ayadaw
    Budalin
    Botahtaung
    Dagon Myothit (East)
    Chaung-U
    Monywa
    Lahe
    Lay Shi
    Nanyun
    Myaung
    Myinmu
    Sagaing
    Khin-U
    Shwebo
    Tabayin
    Salingyi
    Yinmarbin
    Kengtung
    Mongkhet
    Mongla
    Mongping
    Mongyang
    Pangwaun
    Konkyan
    Laukkaing
    Hsipaw
    Kyaukme
    Namtu
    Nawnghkio
    Hseni
    Kunlong
    Lashio
    Pindaya
    Ywangan
    Langkho
    Mawkmai
    Mongnai
    Mongpan
    Kunhing
    Kyethi
    Laihka
    Loilen
    Monghsu
    Mongkaing
    Nansang
    Hopong
    Hsihseng
    Pinlaung
    Kalaw
    Lawksawk
    Nyaungshwe
    Pekon
    Taunggyi
    Dawei
    Launglon
    Thayetchaung
    Yebyu
    Bokpyin
    Kawthoung
    Kyunsu
    Myeik
    Dagon Myothit (North)
    Dagon Myothit (Seikkan)
    Dagon Myothit (South)
    Dawbon
    Yankin
    Mingalartaungnyunt
    North Okkalapa
    Pazundaung
    Kyauktan
    South Okkalapa
    Tamwe
    Thaketa
    Thingangyun
    Hlaingtharya (East)
    Hlaingtharya (West)
    Hlegu
    Hmawbi
    Insein
    Mingaladon
    Shwepyithar
    Taikkyi
    Cocokyun
    Dala
    Kawhmu
    Kayan
    Seikgyikanaungto
    Thanlyin
    Thongwa
    Twantay
    Kungyangon
    Ahlone
    Dagon
    Hlaing
    Kamaryut
    Bahan
    Kyeemyindaing
    Lanmadaw
    Sanchaung
    

#### The following code lines iterate through all the township names in the victims_df, aka aapp_df, and find the closest match with respect to all the township names included in the shapefile. 


```python
shapefile_township = [] #Contains all the townships of the shapefile 
aapp_townships = [] #Contains the closest match between the shapefil and the victims_df, if there is no match "" will appear
for name in victims_df["township_town"].unique():
    aapp_township = str(name)
    print("Township name of AAPP Victims list", aapp_township + "\nClosest match in shapefile is:")
    match = difflib.get_close_matches(aapp_township, shapefile_list, n = 1) #Get the closest match
    print(match)
    print("\n")
    shapefile_township.append(aapp_township)
    if len(match) == 0:
        aapp_townships.append(" ")
    else: 
        aapp_townships.append(match)
```

    Township name of AAPP Victims list Lanmadaw
    Closest match in shapefile is:
    ['Lanmadaw']
    
    
    Township name of AAPP Victims list Unknown
    Closest match in shapefile is:
    []
    
    
    Township name of AAPP Victims list Kyauktaga
    Closest match in shapefile is:
    ['Kyauktaga']
    
    
    Township name of AAPP Victims list Myingyan
    Closest match in shapefile is:
    ['Myingyan']
    
    
    Township name of AAPP Victims list Pazundaung
    Closest match in shapefile is:
    ['Pazundaung']
    
    
    Township name of AAPP Victims list Hlaingtharya
    Closest match in shapefile is:
    ['Hlaingtharya (West)']
    
    
    Township name of AAPP Victims list Bago
    Closest match in shapefile is:
    ['Bago']
    
    
    Township name of AAPP Victims list Taunggyi
    Closest match in shapefile is:
    ['Taunggyi']
    
    
    Township name of AAPP Victims list Hsihseng
    Closest match in shapefile is:
    ['Hsihseng']
    
    
    Township name of AAPP Victims list Pakokku
    Closest match in shapefile is:
    ['Pakokku']
    
    
    Township name of AAPP Victims list Thandwe
    Closest match in shapefile is:
    ['Thandwe']
    
    
    Township name of AAPP Victims list Thaketa
    Closest match in shapefile is:
    ['Thaketa']
    
    
    Township name of AAPP Victims list Hpakant
    Closest match in shapefile is:
    ['Hpakant']
    
    
    Township name of AAPP Victims list Mahaaungmyay
    Closest match in shapefile is:
    ['Mahaaungmyay']
    
    
    Township name of AAPP Victims list Zay Yar Thi Ri
    Closest match in shapefile is:
    ['Zay Yar Thi Ri']
    
    
    Township name of AAPP Victims list Myeik
    Closest match in shapefile is:
    ['Myeik']
    
    
    Township name of AAPP Victims list Shwepyithar
    Closest match in shapefile is:
    ['Shwepyithar']
    
    
    Township name of AAPP Victims list Kyaukse
    Closest match in shapefile is:
    ['Kyaukse']
    
    
    Township name of AAPP Victims list Yebyu
    Closest match in shapefile is:
    ['Yebyu']
    
    
    Township name of AAPP Victims list Dawei
    Closest match in shapefile is:
    ['Dawei']
    
    
    Township name of AAPP Victims list Dagon Myothit (South)
    Closest match in shapefile is:
    ['Dagon Myothit (South)']
    
    
    Township name of AAPP Victims list Kyaikto
    Closest match in shapefile is:
    ['Kyaikto']
    
    
    Township name of AAPP Victims list North Okkalapa
    Closest match in shapefile is:
    ['North Okkalapa']
    
    
    Township name of AAPP Victims list Taungdwingyi
    Closest match in shapefile is:
    ['Taungdwingyi']
    
    
    Township name of AAPP Victims list Pyigyitagon
    Closest match in shapefile is:
    ['Pyigyitagon']
    
    
    Township name of AAPP Victims list Monywa
    Closest match in shapefile is:
    ['Monywa']
    
    
    Township name of AAPP Victims list Dagon Myothit (North)
    Closest match in shapefile is:
    ['Dagon Myothit (North)']
    
    
    Township name of AAPP Victims list Salin
    Closest match in shapefile is:
    ['Salin']
    
    
    Township name of AAPP Victims list Mawlamyine
    Closest match in shapefile is:
    ['Mawlamyine']
    
    
    Township name of AAPP Victims list Kale
    Closest match in shapefile is:
    ['Kale']
    
    
    Township name of AAPP Victims list Chanmyathazi
    Closest match in shapefile is:
    ['Chanmyathazi']
    
    
    Township name of AAPP Victims list Pwintbyu
    Closest match in shapefile is:
    ['Pwintbyu']
    
    
    Township name of AAPP Victims list Pabedan
    Closest match in shapefile is:
    ['Pabedan']
    
    
    Township name of AAPP Victims list Tilin
    Closest match in shapefile is:
    ['Tilin']
    
    
    Township name of AAPP Victims list Myitkyina
    Closest match in shapefile is:
    ['Myitkyina']
    
    
    Township name of AAPP Victims list Pyapon
    Closest match in shapefile is:
    ['Pyapon']
    
    
    Township name of AAPP Victims list Aungmyaythazan
    Closest match in shapefile is:
    ['Aungmyaythazan']
    
    
    Township name of AAPP Victims list Myaing
    Closest match in shapefile is:
    ['Myaing']
    
    
    Township name of AAPP Victims list Mingaladon
    Closest match in shapefile is:
    ['Mingaladon']
    
    
    Township name of AAPP Victims list Hlaing
    Closest match in shapefile is:
    ['Hlaing']
    
    
    Township name of AAPP Victims list Chauk
    Closest match in shapefile is:
    ['Chauk']
    
    
    Township name of AAPP Victims list Tamwe
    Closest match in shapefile is:
    ['Tamwe']
    
    
    Township name of AAPP Victims list Twantay
    Closest match in shapefile is:
    ['Twantay']
    
    
    Township name of AAPP Victims list South Okkalapa
    Closest match in shapefile is:
    ['South Okkalapa']
    
    
    Township name of AAPP Victims list Thingangyun
    Closest match in shapefile is:
    ['Thingangyun']
    
    
    Township name of AAPP Victims list Dagon Myothit (Seikk…
    Closest match in shapefile is:
    ['Dagon Myothit (Seikkan)']
    
    
    Township name of AAPP Victims list Insein
    Closest match in shapefile is:
    ['Insein']
    
    
    Township name of AAPP Victims list Kyeemyindaing
    Closest match in shapefile is:
    ['Kyeemyindaing']
    
    
    Township name of AAPP Victims list Htantabin
    Closest match in shapefile is:
    ['Htantabin']
    
    
    Township name of AAPP Victims list Thayarwady
    Closest match in shapefile is:
    ['Thayarwady']
    
    
    Township name of AAPP Victims list Pathein
    Closest match in shapefile is:
    ['Pathein']
    
    
    Township name of AAPP Victims list Thabeikkyin
    Closest match in shapefile is:
    ['Thabeikkyin']
    
    
    Township name of AAPP Victims list Singu
    Closest match in shapefile is:
    ['Singu']
    
    
    Township name of AAPP Victims list Yinmarbin
    Closest match in shapefile is:
    ['Yinmarbin']
    
    
    Township name of AAPP Victims list Aungban
    Closest match in shapefile is:
    ['Aunglan']
    
    
    Township name of AAPP Victims list Za Bu Thi Ri
    Closest match in shapefile is:
    ['Za Bu Thi Ri']
    
    
    Township name of AAPP Victims list Bokpyin
    Closest match in shapefile is:
    ['Bokpyin']
    
    
    Township name of AAPP Victims list Kawlin
    Closest match in shapefile is:
    ['Kawlin']
    
    
    Township name of AAPP Victims list Chaung-U
    Closest match in shapefile is:
    ['Chaung-U']
    
    
    Township name of AAPP Victims list Mingalartaungnyunt
    Closest match in shapefile is:
    ['Mingalartaungnyunt']
    
    
    Township name of AAPP Victims list Mogoke
    Closest match in shapefile is:
    ['Mogoke']
    
    
    Township name of AAPP Victims list Loikaw
    Closest match in shapefile is:
    ['Loikaw']
    
    
    Township name of AAPP Victims list Chanayethazan
    Closest match in shapefile is:
    ['Chanayethazan']
    
    
    Township name of AAPP Victims list Gangaw
    Closest match in shapefile is:
    ['Gangaw']
    
    
    Township name of AAPP Victims list Kyaukpadaung
    Closest match in shapefile is:
    ['Kyaukpadaung']
    
    
    Township name of AAPP Victims list Mohnyin
    Closest match in shapefile is:
    ['Mohnyin']
    
    
    Township name of AAPP Victims list Khin-U
    Closest match in shapefile is:
    ['Khin-U']
    
    
    Township name of AAPP Victims list Tamu
    Closest match in shapefile is:
    ['Tamu']
    
    
    Township name of AAPP Victims list Thanlyin
    Closest match in shapefile is:
    ['Thanlyin']
    
    
    Township name of AAPP Victims list Sanchaung
    Closest match in shapefile is:
    ['Sanchaung']
    
    
    Township name of AAPP Victims list Mayangone
    Closest match in shapefile is:
    ['Mayangone']
    
    
    Township name of AAPP Victims list Dala
    Closest match in shapefile is:
    ['Dala']
    
    
    Township name of AAPP Victims list Nyaung-U
    Closest match in shapefile is:
    ['Nyaung-U']
    
    
    Township name of AAPP Victims list Meiktila
    Closest match in shapefile is:
    ['Meiktila']
    
    
    Township name of AAPP Victims list Wundwin
    Closest match in shapefile is:
    ['Wundwin']
    
    
    Township name of AAPP Victims list Pyinoolwin
    Closest match in shapefile is:
    ['Pyinoolwin']
    
    
    Township name of AAPP Victims list Sintgaing
    Closest match in shapefile is:
    ['Sintgaing']
    
    
    Township name of AAPP Victims list Amarapura
    Closest match in shapefile is:
    ['Amarapura']
    
    
    Township name of AAPP Victims list Kawthoung
    Closest match in shapefile is:
    ['Kawthoung']
    
    
    Township name of AAPP Victims list Shwebo
    Closest match in shapefile is:
    ['Shwebo']
    
    
    Township name of AAPP Victims list Salingyi
    Closest match in shapefile is:
    ['Salingyi']
    
    
    Township name of AAPP Victims list Daik-U
    Closest match in shapefile is:
    ['Daik-U']
    
    
    Township name of AAPP Victims list Monyo
    Closest match in shapefile is:
    ['Monyo']
    
    
    Township name of AAPP Victims list Lashio
    Closest match in shapefile is:
    ['Lashio']
    
    
    Township name of AAPP Victims list Hopin
    Closest match in shapefile is:
    ['Hopong']
    
    
    Township name of AAPP Victims list Bhamo
    Closest match in shapefile is:
    ['Bhamo']
    
    
    Township name of AAPP Victims list Madaya
    Closest match in shapefile is:
    ['Madaya']
    
    
    Township name of AAPP Victims list Kani
    Closest match in shapefile is:
    ['Kani']
    
    
    Township name of AAPP Victims list Muse
    Closest match in shapefile is:
    ['Muse']
    
    
    Township name of AAPP Victims list Bahan
    Closest match in shapefile is:
    ['Bahan']
    
    
    Township name of AAPP Victims list Pyay
    Closest match in shapefile is:
    ['Pyay']
    
    
    Township name of AAPP Victims list Pinlebu
    Closest match in shapefile is:
    ['Pinlebu']
    
    
    Township name of AAPP Victims list Indaw
    Closest match in shapefile is:
    ['Indaw']
    
    
    Township name of AAPP Victims list Nyaungshwe
    Closest match in shapefile is:
    ['Nyaungshwe']
    
    
    Township name of AAPP Victims list Taze
    Closest match in shapefile is:
    ['Taze']
    
    
    Township name of AAPP Victims list Myitnge
    Closest match in shapefile is:
    ['Myaing']
    
    
    Township name of AAPP Victims list Kyaukme
    Closest match in shapefile is:
    ['Kyaukme']
    
    
    Township name of AAPP Victims list Myaung
    Closest match in shapefile is:
    ['Myaung']
    
    
    Township name of AAPP Victims list Yesagyo
    Closest match in shapefile is:
    ['Yesagyo']
    
    
    Township name of AAPP Victims list Pyinmana
    Closest match in shapefile is:
    ['Pyinmana']
    
    
    Township name of AAPP Victims list Tedim
    Closest match in shapefile is:
    ['Tedim']
    
    
    Township name of AAPP Victims list Launglon
    Closest match in shapefile is:
    ['Launglon']
    
    
    Township name of AAPP Victims list Hsipaw
    Closest match in shapefile is:
    ['Hsipaw']
    
    
    Township name of AAPP Victims list Nawnghkio
    Closest match in shapefile is:
    ['Nawnghkio']
    
    
    Township name of AAPP Victims list Wetlet
    Closest match in shapefile is:
    ['Wetlet']
    
    
    Township name of AAPP Victims list Pauk
    Closest match in shapefile is:
    ['Pauk']
    
    
    Township name of AAPP Victims list Hakha
    Closest match in shapefile is:
    ['Hakha']
    
    
    Township name of AAPP Victims list Padaung
    Closest match in shapefile is:
    ['Padaung']
    
    
    Township name of AAPP Victims list Demoso
    Closest match in shapefile is:
    ['Demoso']
    
    
    Township name of AAPP Victims list Myothit
    Closest match in shapefile is:
    ['Myothit']
    
    
    Township name of AAPP Victims list Pekon
    Closest match in shapefile is:
    ['Pekon']
    
    
    Township name of AAPP Victims list Kyonpyaw
    Closest match in shapefile is:
    ['Kyonpyaw']
    
    
    Township name of AAPP Victims list Yenangyaung
    Closest match in shapefile is:
    ['Yenangyaung']
    
    
    Township name of AAPP Victims list Taungtha
    Closest match in shapefile is:
    ['Taungtha']
    
    
    Township name of AAPP Victims list Kengtung
    Closest match in shapefile is:
    ['Kengtung']
    
    
    Township name of AAPP Victims list Chaungzon
    Closest match in shapefile is:
    ['Chaungzon']
    
    
    Township name of AAPP Victims list Shwedaung
    Closest match in shapefile is:
    ['Shwedaung']
    
    
    Township name of AAPP Victims list Waingmaw
    Closest match in shapefile is:
    ['Waingmaw']
    
    
    Township name of AAPP Victims list Patheingyi
    Closest match in shapefile is:
    ['Patheingyi']
    
    
    Township name of AAPP Victims list Thongwa
    Closest match in shapefile is:
    ['Thongwa']
    
    
    Township name of AAPP Victims list Mingin
    Closest match in shapefile is:
    ['Mingin']
    
    
    Township name of AAPP Victims list Seikphyu
    Closest match in shapefile is:
    ['Seikphyu']
    
    
    Township name of AAPP Victims list Momauk
    Closest match in shapefile is:
    ['Momauk']
    
    
    Township name of AAPP Victims list Sinbaungwe
    Closest match in shapefile is:
    ['Sinbaungwe']
    
    
    Township name of AAPP Victims list Thaton
    Closest match in shapefile is:
    ['Thaton']
    
    
    Township name of AAPP Victims list Mogaung
    Closest match in shapefile is:
    ['Mogaung']
    
    
    Township name of AAPP Victims list Myanaung
    Closest match in shapefile is:
    ['Myanaung']
    
    
    Township name of AAPP Victims list Falam
    Closest match in shapefile is:
    ['Falam']
    
    
    Township name of AAPP Victims list Botahtaung
    Closest match in shapefile is:
    ['Botahtaung']
    
    
    Township name of AAPP Victims list Kayan
    Closest match in shapefile is:
    ['Kayan']
    
    
    Township name of AAPP Victims list Monekoe
    Closest match in shapefile is:
    ['Monyo']
    
    
    Township name of AAPP Victims list Paung
    Closest match in shapefile is:
    ['Paung']
    
    
    Township name of AAPP Victims list Pale
    Closest match in shapefile is:
    ['Pale']
    
    
    Township name of AAPP Victims list Ye-U
    Closest match in shapefile is:
    ['Ye-U']
    
    
    Township name of AAPP Victims list Waw
    Closest match in shapefile is:
    ['Waw']
    
    
    Township name of AAPP Victims list Natogyi
    Closest match in shapefile is:
    ['Natogyi']
    
    
    Township name of AAPP Victims list Kanpetlet
    Closest match in shapefile is:
    ['Kanpetlet']
    
    
    Township name of AAPP Victims list Ye
    Closest match in shapefile is:
    ['Ye']
    
    
    Township name of AAPP Victims list Shwegu
    Closest match in shapefile is:
    ['Shwegu']
    
    
    Township name of AAPP Victims list Hlegu
    Closest match in shapefile is:
    ['Hlegu']
    
    
    Township name of AAPP Victims list Tigyaing
    Closest match in shapefile is:
    ['Tigyaing']
    
    
    Township name of AAPP Victims list Ingapu
    Closest match in shapefile is:
    ['Ingapu']
    
    
    Township name of AAPP Victims list Tabayin
    Closest match in shapefile is:
    ['Tabayin']
    
    
    Township name of AAPP Victims list Poke Ba Thi Ri
    Closest match in shapefile is:
    ['Poke Ba Thi Ri']
    
    
    Township name of AAPP Victims list Kanbalu
    Closest match in shapefile is:
    ['Kanbalu']
    
    
    Township name of AAPP Victims list Aunglan
    Closest match in shapefile is:
    ['Aunglan']
    
    
    Township name of AAPP Victims list Ayadaw
    Closest match in shapefile is:
    ['Ayadaw']
    
    
    Township name of AAPP Victims list Budalin
    Closest match in shapefile is:
    ['Budalin']
    
    
    Township name of AAPP Victims list Tanintharyi
    Closest match in shapefile is:
    ['Tanintharyi']
    
    
    Township name of AAPP Victims list Paungde
    Closest match in shapefile is:
    ['Paungde']
    
    
    Township name of AAPP Victims list Yegyi
    Closest match in shapefile is:
    ['Yegyi']
    
    
    Township name of AAPP Victims list Kyauktada
    Closest match in shapefile is:
    ['Kyauktada']
    
    
    Township name of AAPP Victims list Kyunhla
    Closest match in shapefile is:
    ['Kyunhla']
    
    
    Township name of AAPP Victims list Katha
    Closest match in shapefile is:
    ['Katha']
    
    
    Township name of AAPP Victims list Kungyangon
    Closest match in shapefile is:
    ['Kungyangon']
    
    
    Township name of AAPP Victims list Natmauk
    Closest match in shapefile is:
    ['Natmauk']
    
    
    Township name of AAPP Victims list Palaw
    Closest match in shapefile is:
    ['Palaw']
    
    
    Township name of AAPP Victims list Labutta
    Closest match in shapefile is:
    ['Labutta']
    
    
    Township name of AAPP Victims list Ywangan
    Closest match in shapefile is:
    ['Ywangan']
    
    
    Township name of AAPP Victims list Thanbyuzayat
    Closest match in shapefile is:
    ['Thanbyuzayat']
    
    
    Township name of AAPP Victims list Sidoktaya
    Closest match in shapefile is:
    ['Sidoktaya']
    
    
    Township name of AAPP Victims list Mindat
    Closest match in shapefile is:
    ['Mindat']
    
    
    Township name of AAPP Victims list Lewe
    Closest match in shapefile is:
    ['Lewe']
    
    
    Township name of AAPP Victims list Letpadan
    Closest match in shapefile is:
    ['Letpadan']
    
    
    Township name of AAPP Victims list Bilin
    Closest match in shapefile is:
    ['Bilin']
    
    
    Township name of AAPP Victims list Taungoo
    Closest match in shapefile is:
    ['Taungoo']
    
    
    Township name of AAPP Victims list Thayetchaung
    Closest match in shapefile is:
    ['Thayetchaung']
    
    
    Township name of AAPP Victims list Matupi
    Closest match in shapefile is:
    ['Matupi']
    
    
    Township name of AAPP Victims list Sagaing
    Closest match in shapefile is:
    ['Sagaing']
    
    
    Township name of AAPP Victims list Myinmu
    Closest match in shapefile is:
    ['Myinmu']
    
    
    Township name of AAPP Victims list Gyobingauk
    Closest match in shapefile is:
    ['Gyobingauk']
    
    
    Township name of AAPP Victims list Puta-O
    Closest match in shapefile is:
    ['Puta-O']
    
    
    Township name of AAPP Victims list Kawkareik
    Closest match in shapefile is:
    ['Kawkareik']
    
    
    Township name of AAPP Victims list Hpapun
    Closest match in shapefile is:
    ['Hpapun']
    
    
    Township name of AAPP Victims list Nattalin
    Closest match in shapefile is:
    ['Nattalin']
    
    
    Township name of AAPP Victims list Kawhmu
    Closest match in shapefile is:
    ['Kawhmu']
    
    
    Township name of AAPP Victims list Myawaddy
    Closest match in shapefile is:
    ['Myawaddy']
    
    
    Township name of AAPP Victims list Thandaunggyi
    Closest match in shapefile is:
    ['Thandaunggyi']
    
    
    Township name of AAPP Victims list Tonzang
    Closest match in shapefile is:
    ['Tonzang']
    
    
    Township name of AAPP Victims list Hpa-An
    Closest match in shapefile is:
    ['Hpa-An']
    
    
    Township name of AAPP Victims list Kyainseikgyi
    Closest match in shapefile is:
    ['Kyainseikgyi']
    
    
    Township name of AAPP Victims list Saw
    Closest match in shapefile is:
    ['Saw']
    
    
    Township name of AAPP Victims list Kyaukkyi
    Closest match in shapefile is:
    ['Kyaukkyi']
    
    
    Township name of AAPP Victims list Tangyan
    Closest match in shapefile is:
    ['Tangyan']
    
    
    Township name of AAPP Victims list Paletwa
    Closest match in shapefile is:
    ['Paletwa']
    
    
    Township name of AAPP Victims list Kutkai
    Closest match in shapefile is:
    ['Kutkai']
    
    
    Township name of AAPP Victims list Mrauk-U
    Closest match in shapefile is:
    ['Mrauk-U']
    
    
    Township name of AAPP Victims list Shwegyin
    Closest match in shapefile is:
    ['Shwegyin']
    
    
    Township name of AAPP Victims list Oke Ta Ra Thi Ri
    Closest match in shapefile is:
    ['Oke Ta Ra Thi Ri']
    
    
    Township name of AAPP Victims list Inntakaw
    Closest match in shapefile is:
    ['Pantanaw']
    
    
    Township name of AAPP Victims list Kyauktaw
    Closest match in shapefile is:
    ['Kyauktaw']
    
    
    Township name of AAPP Victims list Hseni
    Closest match in shapefile is:
    ['Hseni']
    
    
    Township name of AAPP Victims list Kawa
    Closest match in shapefile is:
    ['Kawa']
    
    
    Township name of AAPP Victims list Minbya
    Closest match in shapefile is:
    ['Minbya']
    
    
    Township name of AAPP Victims list Buthidaung
    Closest match in shapefile is:
    ['Buthidaung']
    
    
    Township name of AAPP Victims list Rathedaung
    Closest match in shapefile is:
    ['Rathedaung']
    
    
    Township name of AAPP Victims list Banmauk
    Closest match in shapefile is:
    ['Banmauk']
    
    
    Township name of AAPP Victims list Namhkan
    Closest match in shapefile is:
    ['Namhkan']
    
    
    Township name of AAPP Victims list Ponnagyun
    Closest match in shapefile is:
    ['Ponnagyun']
    
    
    Township name of AAPP Victims list Maungdaw
    Closest match in shapefile is:
    ['Maungdaw']
    
    
    Township name of AAPP Victims list Kyaikmaraw
    Closest match in shapefile is:
    ['Kyaikmaraw']
    
    
    Township name of AAPP Victims list Wuntho
    Closest match in shapefile is:
    ['Wuntho']
    
    
    Township name of AAPP Victims list Namhsan
    Closest match in shapefile is:
    ['Namhsan']
    
    
    Township name of AAPP Victims list Zigon
    Closest match in shapefile is:
    ['Zigon']
    
    
    Township name of AAPP Victims list Mawlaik
    Closest match in shapefile is:
    ['Mawlaik']
    
    
    Township name of AAPP Victims list Kyaukhtu
    Closest match in shapefile is:
    ['Kyauktaw']
    
    
    Township name of AAPP Victims list Ngazun
    Closest match in shapefile is:
    ['Ngazun']
    
    
    Township name of AAPP Victims list Thantlang
    Closest match in shapefile is:
    ['Thantlang']
    
    
    Township name of AAPP Victims list Myittha
    Closest match in shapefile is:
    ['Myittha']
    
    
    Township name of AAPP Victims list Hpayarthonesu
    Closest match in shapefile is:
    []
    
    
    Township name of AAPP Victims list Kalaw
    Closest match in shapefile is:
    ['Kalaw']
    
    
    Township name of AAPP Victims list Dawbon
    Closest match in shapefile is:
    ['Dawbon']
    
    
    Township name of AAPP Victims list Homalin
    Closest match in shapefile is:
    ['Homalin']
    
    
    Township name of AAPP Victims list Ngathayauk
    Closest match in shapefile is:
    ['Natmauk']
    
    
    Township name of AAPP Victims list Kyondoe
    Closest match in shapefile is:
    []
    
    
    Township name of AAPP Victims list Nam Mar
    Closest match in shapefile is:
    []
    
    
    Township name of AAPP Victims list Yedashe
    Closest match in shapefile is:
    ['Yedashe']
    
    
    Township name of AAPP Victims list Pinlaung
    Closest match in shapefile is:
    ['Pinlaung']
    
    
    Township name of AAPP Victims list Okpho
    Closest match in shapefile is:
    ['Okpho']
    
    
    Township name of AAPP Victims list Laungon
    Closest match in shapefile is:
    ['Launglon']
    
    
    Township name of AAPP Victims list Mogaung Town
    Closest match in shapefile is:
    ['Mogaung']
    
    
    


```python
print("It seems that all townships match!")
```

    It seems that all townships match!
    

### Perform a join and check the number of strings that did not find any match


```python
#Create two dataframes that will be used for merging
township_shapefile = township_df["TS"].to_frame() #This contains the original township column of the township shapefile data table
township_aapp = victims_df["township_town"].to_frame() #This contains the original township column of victims_df
```


```python
#Create a dataframe that merges the two lists (dataframes) from the townships of the victims dataframe and the townships of the shapefile
#Using the indicator parameter, verify if there was match or not
townships_merged_df = pd.merge(township_shapefile, township_aapp, left_on='TS', right_on='township_town', how='outer', indicator='_merged')


#Both indicates that the township is found in the victims_df and the shapefile dataset
townships_merged_df['_merged'] = townships_merged_df['_merged'].replace({'both': 'Both'})

#Left indicates that the township is only found in the township_shapefile
townships_merged_df['_merged'] = townships_merged_df['_merged'].replace({'left_only': 'Only in shapefile'})

#Right indicates that the township is only found in the township_aapp 
townships_merged_df['_merged'] = townships_merged_df['_merged'].replace({'right_only': 'Only in victims_df'})


```

#### Inspect the townships that are the victims dataframe but not found in the shapefile


```python
only_victims_df = townships_merged_df[townships_merged_df['_merged'] == 'Only in victims_df']
print("\nThese are the townships in the victims dataframe that are not found in the shapefile")
for town in only_victims_df["township_town"].unique():
    print(town)
```

    
    These are the townships in the victims dataframe that are not found in the shapefile
    Unknown
    Hlaingtharya
    Dagon Myothit (Seikk…
    Aungban
    Hopin
    Myitnge
    Monekoe
    Inntakaw
    Kyaukhtu
    Hpayarthonesu
    Ngathayauk
    Kyondoe
    Nam Mar
    Laungon
    Mogaung Town
    


```python
shapefile_township = []
victims_townships = []
for name in only_victims_df["township_town"].unique():
    victims_township = str(name)
    print("Township name of victims list", victims_township + "\nClosest match in shapefile is:")
    match = difflib.get_close_matches(victims_township, shapefile_list, n = 2)
    print(match)
    print("\n")
    shapefile_township.append(victims_township)
    if len(match) == 0:
        victims_townships.append(" ")
    else: 
        victims_townships.append(match)
```

    Township name of victims list Unknown
    Closest match in shapefile is:
    []
    
    
    Township name of victims list Hlaingtharya
    Closest match in shapefile is:
    ['Hlaingtharya (West)', 'Hlaingtharya (East)']
    
    
    Township name of victims list Dagon Myothit (Seikk…
    Closest match in shapefile is:
    ['Dagon Myothit (Seikkan)', 'Dagon Myothit (South)']
    
    
    Township name of victims list Aungban
    Closest match in shapefile is:
    ['Aunglan', 'Paungbyin']
    
    
    Township name of victims list Hopin
    Closest match in shapefile is:
    ['Hopong', 'Hopang']
    
    
    Township name of victims list Myitnge
    Closest match in shapefile is:
    ['Myaing', 'Myingyan']
    
    
    Township name of victims list Monekoe
    Closest match in shapefile is:
    ['Monyo', 'Mongkhet']
    
    
    Township name of victims list Inntakaw
    Closest match in shapefile is:
    ['Pantanaw', 'Indaw']
    
    
    Township name of victims list Kyaukhtu
    Closest match in shapefile is:
    ['Kyauktaw', 'Kyauktan']
    
    
    Township name of victims list Hpayarthonesu
    Closest match in shapefile is:
    []
    
    
    Township name of victims list Ngathayauk
    Closest match in shapefile is:
    ['Natmauk']
    
    
    Township name of victims list Kyondoe
    Closest match in shapefile is:
    []
    
    
    Township name of victims list Nam Mar
    Closest match in shapefile is:
    []
    
    
    Township name of victims list Laungon
    Closest match in shapefile is:
    ['Launglon', 'Nyaungdon']
    
    
    Township name of victims list Mogaung Town
    Closest match in shapefile is:
    ['Mogaung', 'Mongton']
    
    
    


```python
#Read the excel file that contains the updated values for the townships and the reasons on how I decided to update the values
townships_to_be_updated_df = pd.read_excel("Townships_updated_values_for_join.xlsx")
```


```python
townships_updates_dict = dict(zip(townships_to_be_updated_df['Victims_township'], townships_to_be_updated_df['Match to shapefile']))
```


```python
townships_updates_dict
```




    {'Aungban': 'Kalaw', 'Hopin': 'Mohnyin', 'Hlaingtharya': 'Hlaingtharya (West)', 'Dagon Myothit (Seikk…': 'Dagon Myothit (Seikkan)', 'Myitnge': 'Amarapura', 'Monekoe': 'Muse', 'Inntakaw': 'Bago', 'Kyaukhtu': 'Saw', 'Hpayarthonesu': 'Kyainseikgyi', 'Ngathayauk': 'Nyaung-U', 'Kyondoe': 'Kawkareik', 'Nam Mar': 'Mohnyin', 'Laungon': 'Launglon', 'Mogaung Town': 'Mogaung'}



#### Update the township names in the victims_df so that they can match with the shapefile


```python
victims_df["township_town"] = victims_df["township_town"].map(townships_updates_dict).fillna(victims_df["township_town"])
```

### In the case of state_region of the victims_df ensure that they also match with the state_region column of the shapefile


```python
shapefile_list = list(township_df["ST"].unique())
shapefile_state = [] #Contains all the states of the shapefile 
aapp_states = [] #Contains the closest match between the shapefil and the victims_df, if there is no match "" will appear
for name in victims_df["state_region"].unique():
    aapp_state = str(name)
    print("state name of AAPP Victims list", aapp_state + "\nClosest match in shapefile is:")
    match = difflib.get_close_matches(aapp_state, shapefile_list, n = 1) #Get the closest match
    print(match)
    print("\n")
    shapefile_state.append(aapp_state)
    if len(match) == 0:
        aapp_states.append(" ")
    else: 
        aapp_states.append(match)
```

    state name of AAPP Victims list Yangon
    Closest match in shapefile is:
    ['Yangon']
    
    
    state name of AAPP Victims list Mon
    Closest match in shapefile is:
    ['Mon']
    
    
    state name of AAPP Victims list Magway
    Closest match in shapefile is:
    ['Magway']
    
    
    state name of AAPP Victims list Ayeyarwady
    Closest match in shapefile is:
    ['Ayeyarwady']
    
    
    state name of AAPP Victims list Mandalay
    Closest match in shapefile is:
    ['Mandalay']
    
    
    state name of AAPP Victims list Bago
    Closest match in shapefile is:
    ['Yangon']
    
    
    state name of AAPP Victims list Shan
    Closest match in shapefile is:
    []
    
    
    state name of AAPP Victims list Rakhine
    Closest match in shapefile is:
    ['Rakhine']
    
    
    state name of AAPP Victims list Sagaing
    Closest match in shapefile is:
    ['Sagaing']
    
    
    state name of AAPP Victims list Kachin
    Closest match in shapefile is:
    ['Kachin']
    
    
    state name of AAPP Victims list Nay Pyi Taw
    Closest match in shapefile is:
    ['Nay Pyi Taw']
    
    
    state name of AAPP Victims list Tanintharyi
    Closest match in shapefile is:
    ['Tanintharyi']
    
    
    state name of AAPP Victims list Kayah
    Closest match in shapefile is:
    ['Kayah']
    
    
    state name of AAPP Victims list Chin
    Closest match in shapefile is:
    ['Chin']
    
    
    state name of AAPP Victims list Kayin
    Closest match in shapefile is:
    ['Kayin']
    
    
    state name of AAPP Victims list Unknown
    Closest match in shapefile is:
    []
    
    
    


```python
#Create two dataframes that will be used for merging
state_shapefile = township_df["ST"].to_frame() #This contains the original state column of the township shapefile data table
state_victims = victims_df["state_region"].to_frame() #This contains the original state column of victims_df
```


```python
#Create a dataframe that merges the two lists (dataframes) from the states of the victims dataframe and the states of the shapefile
#Using the indicator parameter, verify if there was match or not
states_merged_df = pd.merge(state_shapefile, state_victims, left_on='ST', right_on='state_region', how='outer', indicator='_merged')

#Both indicates that the state is found in the victims_df and the shapefile dataset
states_merged_df['_merged'] = states_merged_df['_merged'].replace({'both': 'Both'})

#Left indicates that the state is only found in the township_shapefile
states_merged_df['_merged'] = states_merged_df['_merged'].replace({'left_only': 'Only in shapefile'})

#Right indicates that the state is only found in the township_aapp 
states_merged_df['_merged'] = states_merged_df['_merged'].replace({'right_only': 'Only in victims_df'})


```


```python
only_victims_df = states_merged_df[states_merged_df['_merged'] == 'Only in victims_df']
print("\nThese are the states in the victims dataframe that are not found in the shapefile")
for state in only_victims_df["state_region"].unique():
    print(state)
```

    
    These are the states in the victims dataframe that are not found in the shapefile
    Bago
    Shan
    Unknown
    


```python
township_df["ST"].unique()
```




    array(['Ayeyarwady', 'Bago (East)', 'Kachin', 'Yangon', 'Kayin', 'Magway',
           'Bago (West)', 'Mandalay', 'Mon', 'Chin', 'Kayah', 'Nay Pyi Taw',
           'Rakhine', 'Sagaing', 'Shan (East)', 'Shan (North)', 'Tanintharyi',
           'Shan (South)'], dtype=object)



The reason why these states do not match is because the victims_df do not distinguish between Bago (East) and Bago (West), same with Shan state <br>
To deal with this, all the townships inside each state will be retrieved, then in the victims_df, the state_region will be updated according to the lists


```python
Bago_west_townships = list(township_df.loc[township_df["ST"] == "Bago (West)", "TS"])
victims_df.loc[victims_df["township_town"].isin(Bago_west_townships), "state_region"] = "Bago (West)"
```


```python
Bago_east_townships = list(township_df.loc[township_df["ST"] == "Bago (East)", "TS"])
victims_df.loc[victims_df["township_town"].isin(Bago_east_townships), "state_region"] = "Bago (East)"
```


```python
Shan_east_townships = list(township_df.loc[township_df["ST"] == "Shan (East)", "TS"])
victims_df.loc[victims_df["township_town"].isin(Shan_east_townships), "state_region"] = "Shan (East)"
```


```python
Shan_north_townships = list(township_df.loc[township_df["ST"] == "Shan (North)", "TS"])
victims_df.loc[victims_df["township_town"].isin(Shan_north_townships), "state_region"] = "Shan (North)"
```


```python
Shan_south_townships = list(township_df.loc[township_df["ST"] == "Shan (South)", "TS"])
victims_df.loc[victims_df["township_town"].isin(Shan_south_townships), "state_region"] = "Shan (South)"
```


```python
#In the case where state_region is still Bago or Shan, it indicates, that the specific region is Unknown, values will be updated accordingly
unknown_regions_dict = {"Bago" : "Unknown", "Shan" : "Unknown"}
victims_df["state_region"] = victims_df["state_region"].map(unknown_regions_dict).fillna(victims_df["state_region"])
```

### Eliminate the rows that have missing values for state and township, i.e. unknown location



```python
victims_df = victims_df.drop(victims_df[(victims_df["state_region"] == "Unknown") & (victims_df["township_town"] == "Unknown")].index)
```

### The victims_df is ready to be match with the township shapefile based on townships!


```python
#victims_df.to_excel("victims_township_level_vf.xlsx")
```

## It is necessary to display the count of victims at township level and include the count according to gender and sector


```python
#Use pivot table to count the number of victims that belong to each level of the categorical vars gender, sector at each township
township_victims_per_sector_df = victims_df.pivot_table(index='township_town', columns='sector', values='id_number', aggfunc='count', dropna=False)
```


```python
township_victims_per_gender_df = victims_df.pivot_table(index='township_town', columns='gender', values='id_number', aggfunc='count', dropna=False)
```


```python
### Create a new column that contains the group age 

def get_age_group(age):
    if age >= 1 and age <= 12:
        return "Children (0 - 12 yrs)"
    elif age >= 13 and age <= 17:
        return "Adolescents (13 - 17 yrs)"
    elif age >= 18 and age <= 64:
        return "Adults (18 - 65 yrs)"
    elif age >= 65:
        return "Older adults (> 65 yrs)"
    else:
        return "Unknown age"
    
victims_df['age_group'] = victims_df['age'].apply(get_age_group)

```


```python
township_victims_per_age_df = victims_df.pivot_table(index='township_town', columns='age_group', values='id_number', aggfunc='count', dropna=False)
```


```python
# Merge the three dataframes based on township_town column
victims_township_df = township_victims_per_sector_df.merge(township_victims_per_gender_df, on='township_town').merge(township_victims_per_age_df, on='township_town')
```


```python
victims_township_df.to_excel("victims_aggregate_counts_township_vf.xlsx")
```

## Load the socioeconomic data collected 


```python
ApproxVulPop_2016_df = pd.read_excel("MIMU_data.xlsx", sheet_name = "ApproxVulPop_2016")
Wealth_Ranking_Index_df = pd.read_excel("MIMU_data.xlsx", sheet_name = "Wealth_Ranking_Index")
DisabledPop_2014_df = pd.read_excel("MIMU_data.xlsx", sheet_name = "DisabledPop_2014")
FemPop_2014_df = pd.read_excel("MIMU_data.xlsx", sheet_name = "FemPop_2014")
MalePop_2014_df = pd.read_excel("MIMU_data.xlsx", sheet_name = "MalePop_2014")
Population_2014_df = pd.read_excel("MIMU_data.xlsx", sheet_name = "Population_2014")
PerAdultFemLiteracyRate_2014_df = pd.read_excel("MIMU_data.xlsx", sheet_name = "%AdultFemLiteracyRate_2014")
PerAdultMaleLiteracyRate_2014_df = pd.read_excel("MIMU_data.xlsx", sheet_name = "%AdultMaleLiteracyRate_2014")
PerAdultLiteracyRate_2014_df = pd.read_excel("MIMU_data.xlsx", sheet_name = "%AdultLiteracyRate_2014")
EmployeeGovTotal_2014_df = pd.read_excel("MIMU_data.xlsx", sheet_name = "EmployeeGovTotal_2014")
FemEmployeeGov_2014_df = pd.read_excel("MIMU_data.xlsx", sheet_name = "FemEmployeeGov_2014")
MaleEmployeeGov_2014_df = pd.read_excel("MIMU_data.xlsx", sheet_name = "MaleEmployeeGov_2014")
StudentTotal_2014_df = pd.read_excel("MIMU_data.xlsx", sheet_name = "StudentTotal_2014")
FemStudentTotal_2014_df = pd.read_excel("MIMU_data.xlsx", sheet_name = "FemStudentTotal_2014")
MaleStudentTotal_2014_df = pd.read_excel("MIMU_data.xlsx", sheet_name = "MaleStudentTotal_2014")
```

### Keep only columns that indicate the state, township, township_pcode and the value of the variable


```python
ApproxVulPop_2016_df = ApproxVulPop_2016_df[["ApproxVulPop_2016", "Township_Name", "Township_Pcode", "State_Region"]]
Wealth_Ranking_Index_df = Wealth_Ranking_Index_df[["WealthRankingIdx_2015", "Township_Name", "Township_Pcode", "State_Region"]]
DisabledPop_2014_df = DisabledPop_2014_df[["DisabledPop_2014", "Township_Name", "Township_Pcode", "State_Region"]]
FemPop_2014_df = FemPop_2014_df[["FemPop_2014", "Township_Name", "Township_Pcode", "State_Region"]]
MalePop_2014_df = MalePop_2014_df[["MalePop_2014", "Township_Name", "Township_Pcode", "State_Region"]]
Population_2014_df = Population_2014_df[["Population_2014", "Township_Name", "Township_Pcode", "State_Region"]]
PerAdultFemLiteracyRate_2014_df = PerAdultFemLiteracyRate_2014_df[["%AdultFemLiteracyRate_2014", "Township_Name", "Township_Pcode", "State_Region"]]
PerAdultMaleLiteracyRate_2014_df = PerAdultMaleLiteracyRate_2014_df[["%AdultMaleLiteracyRate_2014", "Township_Name", "Township_Pcode", "State_Region"]]
PerAdultLiteracyRate_2014_df = PerAdultLiteracyRate_2014_df[["%AdultLiteracyRate_2014", "Township_Name", "Township_Pcode", "State_Region"]]
EmployeeGovTotal_2014_df = EmployeeGovTotal_2014_df[["EmployeeGovTotal_2014", "Township_Name", "Township_Pcode", "State_Region"]]
FemEmployeeGov_2014_df = FemEmployeeGov_2014_df[["FemEmployeeGov_2014", "Township_Name", "Township_Pcode", "State_Region"]]
MaleEmployeeGov_2014_df = MaleEmployeeGov_2014_df[["MaleEmployeeGov_2014", "Township_Name", "Township_Pcode", "State_Region"]]
StudentTotal_2014_df = StudentTotal_2014_df[["StudentTotal_2014", "Township_Name", "Township_Pcode", "State_Region"]]
FemStudentTotal_2014_df = FemStudentTotal_2014_df[["FemStudentTotal_2014", "Township_Name", "Township_Pcode", "State_Region"]]
MaleStudentTotal_2014_df = MaleStudentTotal_2014_df[["MaleStudentTotal_2014", "Township_Name", "Township_Pcode", "State_Region"]]
```

### Merge all the dataframes, rows that have the same string characters of Township_Name, Township_Pcode, State_Region will be merged


```python
# Create a list of dataframes to merge
dfs_to_merge = [ApproxVulPop_2016_df, Wealth_Ranking_Index_df, DisabledPop_2014_df, FemPop_2014_df, MalePop_2014_df, Population_2014_df, PerAdultFemLiteracyRate_2014_df, PerAdultMaleLiteracyRate_2014_df, PerAdultLiteracyRate_2014_df, EmployeeGovTotal_2014_df, FemEmployeeGov_2014_df, MaleEmployeeGov_2014_df, StudentTotal_2014_df, FemStudentTotal_2014_df, MaleStudentTotal_2014_df]

# Merge the dataframes in the list using a for loop
socioeconomc_characs_df = ApproxVulPop_2016_df #Initialize a dataframe, that contains the first dataframe, i.e. ApproxVulPop
for i in range(1, len(dfs_to_merge)): #Iterate through all the indices of the dfs_to_merge list
    socioeconomc_characs_df = pd.merge(merged_df, dfs_to_merge[i], on=["Township_Name", "Township_Pcode", "State_Region"], how="left") #Merge the selected dataframe
```


```python
socioeconomc_characs_df
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
      <th>ApproxVulPop_2016</th>
      <th>Township_Name</th>
      <th>Township_Pcode</th>
      <th>State_Region</th>
      <th>WealthRankingIdx_2015</th>
      <th>DisabledPop_2014</th>
      <th>FemPop_2014</th>
      <th>MalePop_2014</th>
      <th>Population_2014</th>
      <th>%AdultFemLiteracyRate_2014</th>
      <th>%AdultMaleLiteracyRate_2014</th>
      <th>%AdultLiteracyRate_2014</th>
      <th>EmployeeGovTotal_2014</th>
      <th>FemEmployeeGov_2014</th>
      <th>MaleEmployeeGov_2014</th>
      <th>StudentTotal_2014</th>
      <th>FemStudentTotal_2014</th>
      <th>MaleStudentTotal_2014_x</th>
      <th>MaleStudentTotal_2014_y</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>214355</td>
      <td>Bogale</td>
      <td>MMR017024</td>
      <td>Ayeyarwady</td>
      <td>-2.767283</td>
      <td>2689.0</td>
      <td>163369.0</td>
      <td>159296.0</td>
      <td>322665.0</td>
      <td>90.4</td>
      <td>95.8</td>
      <td>93.0</td>
      <td>3817.0</td>
      <td>2247.0</td>
      <td>1570.0</td>
      <td>36390.0</td>
      <td>18273.0</td>
      <td>18117.0</td>
      <td>18117.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>96973</td>
      <td>Danubyu</td>
      <td>MMR017022</td>
      <td>Ayeyarwady</td>
      <td>-1.366612</td>
      <td>1528.0</td>
      <td>93578.0</td>
      <td>85775.0</td>
      <td>179353.0</td>
      <td>94.7</td>
      <td>97.5</td>
      <td>96.0</td>
      <td>2751.0</td>
      <td>1502.0</td>
      <td>1249.0</td>
      <td>17461.0</td>
      <td>8840.0</td>
      <td>8621.0</td>
      <td>8621.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>129947</td>
      <td>Dedaye</td>
      <td>MMR017026</td>
      <td>Ayeyarwady</td>
      <td>-2.416806</td>
      <td>1548.0</td>
      <td>103312.0</td>
      <td>99614.0</td>
      <td>202926.0</td>
      <td>93.9</td>
      <td>96.7</td>
      <td>95.2</td>
      <td>2484.0</td>
      <td>1576.0</td>
      <td>908.0</td>
      <td>23046.0</td>
      <td>11724.0</td>
      <td>11322.0</td>
      <td>11322.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>119848</td>
      <td>Einme</td>
      <td>MMR017015</td>
      <td>Ayeyarwady</td>
      <td>-1.959879</td>
      <td>1418.0</td>
      <td>99472.0</td>
      <td>94629.0</td>
      <td>194101.0</td>
      <td>87.9</td>
      <td>92.0</td>
      <td>89.9</td>
      <td>2449.0</td>
      <td>1367.0</td>
      <td>1082.0</td>
      <td>20501.0</td>
      <td>10334.0</td>
      <td>10167.0</td>
      <td>10167.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>176125</td>
      <td>Hinthada</td>
      <td>MMR017008</td>
      <td>Ayeyarwady</td>
      <td>-1.303521</td>
      <td>2891.0</td>
      <td>178741.0</td>
      <td>159694.0</td>
      <td>338435.0</td>
      <td>94.1</td>
      <td>97.7</td>
      <td>95.7</td>
      <td>7533.0</td>
      <td>4305.0</td>
      <td>3228.0</td>
      <td>35797.0</td>
      <td>18056.0</td>
      <td>17741.0</td>
      <td>17741.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>325</th>
      <td>122645</td>
      <td>Thanlyin</td>
      <td>MMR013023</td>
      <td>Yangon</td>
      <td>2.545048</td>
      <td>2081.0</td>
      <td>137526.0</td>
      <td>130537.0</td>
      <td>268063.0</td>
      <td>96.6</td>
      <td>98.3</td>
      <td>97.4</td>
      <td>12562.0</td>
      <td>3855.0</td>
      <td>8707.0</td>
      <td>29944.0</td>
      <td>14774.0</td>
      <td>15170.0</td>
      <td>15170.0</td>
    </tr>
    <tr>
      <th>326</th>
      <td>49218</td>
      <td>Thingangyun</td>
      <td>MMR013009</td>
      <td>Yangon</td>
      <td>9.391928</td>
      <td>1181.0</td>
      <td>110788.0</td>
      <td>98698.0</td>
      <td>209486.0</td>
      <td>95.8</td>
      <td>98.3</td>
      <td>96.9</td>
      <td>7384.0</td>
      <td>4365.0</td>
      <td>3019.0</td>
      <td>27106.0</td>
      <td>13423.0</td>
      <td>13683.0</td>
      <td>13683.0</td>
    </tr>
    <tr>
      <th>327</th>
      <td>96240</td>
      <td>Thongwa</td>
      <td>MMR013025</td>
      <td>Yangon</td>
      <td>-0.715277</td>
      <td>1054.0</td>
      <td>82384.0</td>
      <td>75492.0</td>
      <td>157876.0</td>
      <td>91.5</td>
      <td>95.4</td>
      <td>93.3</td>
      <td>1904.0</td>
      <td>1173.0</td>
      <td>731.0</td>
      <td>15717.0</td>
      <td>7596.0</td>
      <td>8121.0</td>
      <td>8121.0</td>
    </tr>
    <tr>
      <th>328</th>
      <td>134457</td>
      <td>Twantay</td>
      <td>MMR013027</td>
      <td>Yangon</td>
      <td>-1.089770</td>
      <td>1757.0</td>
      <td>115585.0</td>
      <td>111251.0</td>
      <td>226836.0</td>
      <td>93.0</td>
      <td>96.2</td>
      <td>94.5</td>
      <td>3398.0</td>
      <td>1673.0</td>
      <td>1725.0</td>
      <td>28156.0</td>
      <td>13821.0</td>
      <td>14335.0</td>
      <td>14335.0</td>
    </tr>
    <tr>
      <th>329</th>
      <td>15322</td>
      <td>Yankin</td>
      <td>MMR013010</td>
      <td>Yangon</td>
      <td>10.851770</td>
      <td>436.0</td>
      <td>38222.0</td>
      <td>32724.0</td>
      <td>70946.0</td>
      <td>96.7</td>
      <td>98.7</td>
      <td>97.6</td>
      <td>5027.0</td>
      <td>2638.0</td>
      <td>2389.0</td>
      <td>8957.0</td>
      <td>4545.0</td>
      <td>4412.0</td>
      <td>4412.0</td>
    </tr>
  </tbody>
</table>
<p>330 rows × 19 columns</p>
</div>



### Perform a join and check the number of strings that did not find any match


```python
#Create two dataframes that will be used for merging
township_shapefile = township_df["TS"].to_frame() #This contains the original township column of the township shapefile data table
township_socioeconomic_df = socioeconomc_characs_df["Township_Name"].to_frame() #This contains the original township column of victims_df
```


```python
#Create a dataframe that merges the two lists (dataframes) from the townships of the victims dataframe and the townships of the shapefile
#Using the indicator parameter, verify if there was match or not
townships_merged_df = pd.merge(township_shapefile, township_socioeconomic_df, left_on='TS', right_on='Township_Name', how='outer', indicator='_merged')


#Both indicates that the township is found in the victims_df and the shapefile dataset
townships_merged_df['_merged'] = townships_merged_df['_merged'].replace({'both': 'Both'})

#Left indicates that the township is only found in the township_shapefile
townships_merged_df['_merged'] = townships_merged_df['_merged'].replace({'left_only': 'Only in shapefile'})

#Right indicates that the township is only found in the township_aapp 
townships_merged_df['_merged'] = townships_merged_df['_merged'].replace({'right_only': 'Only in victims_df'})

```


```python
only_victims_df = townships_merged_df[townships_merged_df['_merged'] == 'Only in victims_df']
print("\nThese are the townships in the victims dataframe that are not found in the shapefile")
for town in only_victims_df["Township_Name"].unique():
    print(town)
```

    
    These are the townships in the victims dataframe that are not found in the shapefile
    Minhla (B)
    Bawlakhe
    Yinmabin
    Hseni/Theinni
    Kukai
    Namkhan
    Keshi
    Hlaingtharya
    Htantabin (Y)
    Seikkan
    


```python
#Read the excel file that contains the updated values for the townships and the reasons on how I decided to update the values
townships_to_be_updated_df = pd.read_excel("Townships_updated_values_for_join_socioeconomic_vars_and_shapefile.xlsx")
```


```python
townships_updates_dict = dict(zip(townships_to_be_updated_df['Socioeconomic_df_Township'], townships_to_be_updated_df['Match to shapefil']))
```

#### Update the township names in the socioeconomic_df so that they can match with the shapefile


```python
socioeconomc_characs_df["Township_Name"] = socioeconomc_characs_df["Township_Name"].map(townships_updates_dict).fillna(socioeconomc_characs_df["Township_Name"])
```

#### Drop the rows that contain names of townships not found in the shapefile


```python
socioeconomc_characs_df = socioeconomc_characs_df[~socioeconomc_characs_df["Township_Name"].isin(["Minhla (B)", "Keshi", "Hlaingtharya", "Htantabin (Y)"])]
```

### The socioeconomic_characs_df is ready to be match with the township shapefile based on townships!


```python
socioeconomc_characs_df.to_excel("socioeconomic_vars_township_level_vf.xlsx")
```


