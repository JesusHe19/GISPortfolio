
#!/usr/bin/env python
# coding: utf-8

# ### Import libraries

# In[430]:


import arcgis
import pandas as pd 
import os
import arcpy
import re
import numpy as np
import matplotlib.pyplot as plt
import difflib


# ## Clean and prepare the data from AAPP, i.e. the dataset that contains information about victims

# ### Load table data

# In[431]:


#Remember that the table data has to be located in the same folder
#as your arcgis project
current_path = os.getcwd()
print("The current working directory (where file is located):\n", current_path)


# In[432]:


#Read the excel file as a pandas dataframe
victims_df = pd.read_excel("Victims_data.xlsx")

#Get a quick view of the dataframe, print the first 10 rows
victims_df.head(10)


# In[433]:


for col in victims_df.columns:
    print(col)


# ### Inspect the columns and create a data dictionary

# When cleaning datasets, it is important to understand what each column refers to<br>
# 
# #': Indicates the row number <br>
# 
# link_name: It is not clear what this variable indicates, it contains the name of the victim and a number. The AAPP website do not provide any information about the meaning of this variable. This variable will not be used in the analysis <br>
# 
# id_number: It is the unique identifier of the victim <br>
# 
# name: Name of the victim<br>
# 
# name in burmese: Name of the victim in Burmese language <br>
# 
# gender: Whether the victim's gender is Male (M); Female (F); Unknown; or LGBT <br>
# 
# age: Age of the victim <br>
# 
# parent's_name: Name of victim's parent. In Myanmar people do not have last name, therefore the parents name is used for identifying people  <br>
# 
# sector: It indicates to the working sector a victim belongs to<br>
# 
# category: It indicates the occupation of the victim<br>
# 
# township_town: Myanmar is divided into regions, states, districts, townships and villages <br>
# 
# state_region: Myanmar is divided into regions, states, districts, townships and villages <br>
# 
# deceased_date_view: Date the victim was killed <br>
# 
# #.1: No special meaning
# 
# special_condition_L: Relevant information about the victim <br>
# 
# event_detail_killed_L: Description on how the victim was murdered<br>
# 
# killed_event_type_L: Indicates type of victim, i.e. deceased <br>
# 
# verification: Indicates whether the event was verified, i.e. the event did occur <br>
# 
# Sources:<br>
# Scott, James G. Law and Custom in Burma and the Burmese Family. Oxford University Press, 1963. <br>
# 
# Assistance Association for Political Prisoners, "Daily Briefing Since Coup", Accessed April 28, 2023, https://aappb.org/?cat=109<br>
# 

# ### Drop columns not necessary for the analysis

# In[434]:


#Drop the column that indicates name in Burmese, the column #.1, and the link_name
victims_df = victims_df.drop(['#', "#.1", "link_name", 'name_in_burmese'], axis = 1)


# In[435]:


#AAPP classifies the victims of the Myanmar coup into: 
#deceased, prisoners, and detainees
#This dataset is only for deceased, therefore the column killed_event_type_L
#can be eliminated

victims_df = victims_df.drop(['killed_event_type_L'], axis = 1)


# ### Modify column names to make them more understandable

# In[436]:


victims_df = victims_df.rename(columns={'event_detail_killed_L': 'event_details', 
                                        'special_condition_L' : 'special_condition',
                                         'deceased_date_view' : 'event_date'})


# ### Inspect each categorical column and verify the information is correctly encoded
# #### It will be necessary to count the number of times each unique value of a column appear, and also count the missing values

# In[437]:


def inspect(col):
    print("Column:", col, "\n")
    print("Count of unique values")
    print(victims_df[col].value_counts())
    print("\nThe number of missing values is:", str(victims_df["gender"].isnull().sum()))


# In[438]:


categorical_vars = ["gender", "sector", "category", "special_condition", "verification"]
for var in categorical_vars:
    inspect(var)
    print("\n")


# From the categorical variables, it can be seen that none of them have missing values, there is no need to handle missing data for these variables. <br> 
# However, the sector, category, and special_condition columns need to be cleaned. There is an issue on how the information is encoded. <br>
# 
# Unfortunately, the string characters contained in the columns sector, category, and special condition cannot be fully downloaded from the AAPP open database. The data from AAPP cannot be downloaded as Excel file, it can only downloaded as a PDF. The predetermined width of the column cannot be modified, hence when the database is downloaded as a PDF file, the column widht is too narrow to fully contain all the string characters inside each row. <br>
# 
# The following example will explain the issue: <br>
# * 'National League for De…' The string character for the category "National League for Democracy"
# *  'Regional Hluttaw\nNational League for De…' The string character contains more than one category, categories are separated by an empty space, furthermore the string character is too long.
# 
# The solution for this issue: <br>
# * Contact AAPP and request the dataset as an Excel file, this has already been done but no response has been received since 2 weeks ago. 
# * For each string value, take only the first category, e.g., in this case 'Regional Hluttaw\nNational League for De…', the value will be modified to contain only "Regional Hluttaw". If after taking the first category, the category is still incomplete, the website of AAPP will be manually inspected to identify the correct category, e.g., after inspecting manuall the website it is conclude that 'Internally Displaced People…'  refers to "Internally Displaced People"
# 

# ### Solve the issue in the sector, category, and special condition columns

# #### Category column

# In[439]:


#convert to string type
victims_df["category"] = victims_df["category"].astype(str)


# In[440]:


#Eliminate everything what is after the first line break ""\n"
victims_df["category"] = victims_df["category"].str.split("\n").str[0]


# In[441]:


#Lets create an empty dictionary that contains all
#cases that look like this: "Teacher      Writer"
#The dictionary will help to update the column

long_spaces_str_cases_dict = {} #Create empty dictionary

for case in victims_df["category"].unique(): #iterate through all unique vals
    if re.search(r"\s{3,}", case): #find the cases
        first_word = case.split(" ")[0] #Take only the first word
        long_spaces_str_cases_dict[case] = first_word #Create the dictionary


# In[442]:


#Lets retrieve all cases where the string is incomplete
for case in victims_df["category"].unique():
    if re.search(r"…", case):
        print("\'"+case+'\'' +  " :")


# In[443]:


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


# In[444]:


#Update the column values
victims_df["category"] = victims_df["category"].map(long_spaces_str_cases_dict).fillna(victims_df["category"])
victims_df["category"] = victims_df["category"].map(incomplete_str_cases_dict).fillna(victims_df["category"])


# #### Sector column

# In[445]:


#Convert to string type
victims_df["sector"] = victims_df["sector"].astype(str)


# In[446]:


#Eliminate everything what is after the first line break ""\n"
victims_df["sector"] = victims_df["sector"].str.split("\n").str[0]


# In[447]:


#Lets create an empty dictionary that contains all
#cases that look like this: "Teacher      Writer"
#The dictionary will help to update the column

long_spaces_str_cases_dict = {} #Create empty dictionary

for case in victims_df["sector"].unique(): #iterate through all unique vals
    if re.search(r"\s{3,}", case): #find the cases
        first_word = case.split(" ")[0] #Take only the first word
        long_spaces_str_cases_dict[case] = first_word #Create the dictionary


# In[448]:


#Lets retrieve all cases where the string is incomplete
#there are no cases like this
for case in victims_df["sector"].unique():
    if re.search(r"…", case):
        print("\'"+case+'\'' +  " :")


# In[449]:


#Update the column values
victims_df["sector"] = victims_df["sector"].map(long_spaces_str_cases_dict).fillna(victims_df["sector"])


# In[450]:


#There are typos to fix
fix_typos_dict = {'Member of Parliament Parties' : 'Member of Parliament',
'Civilain' : 'Civilian'}

victims_df["sector"] = victims_df["sector"].map(fix_typos_dict).fillna(victims_df["sector"])


# In[451]:


print(victims_df["sector"].unique())


# The unique values of sector correspond to the data dictionary that AAPP has in their website:
# 
# Assistance Association for Political Prisoners, "Sector Data Definition", Accessed April 28, 2023, https://aappb.org/wp-content/uploads/2022/06/Sector_Data_Definition-2-Feb-2023.pdf

# #### Special condition column

# In[452]:


victims_df["special_condition"] = victims_df["special_condition"].astype(str)


# In[453]:


#Eliminate everything what is after the first line break ""\n"
victims_df["special_condition"] = victims_df["special_condition"].str.split("\n").str[0]


# In[454]:


#Lets create an empty dictionary that contains all
#cases that look like this: "Teacher      Writer"
#The dictionary will help to update the column

long_spaces_str_cases_dict = {} #Create empty dictionary

for case in victims_df["special_condition"].unique(): #iterate through all unique vals
    if re.search(r"\s{3,}", case): #find the cases
        first_word = case.split(" ")[0] #Take only the first word
        long_spaces_str_cases_dict[case] = first_word #Create the dictionary


# In[455]:


#Lets retrieve all cases where the string is incomplete
#there are no cases like this
for case in victims_df["special_condition"].unique():
    if re.search(r"…", case):
        print("\'"+case+'\'' +  " :")


# In[456]:


#Update the column values
victims_df["special_condition"] = victims_df["special_condition"].map(long_spaces_str_cases_dict).fillna(victims_df["special_condition"])


# The unique values of sector correspond to the data dictionary that AAPP has in their website:
# 
# Assistance Association for Political Prisoners, "Sector Data Definition", Accessed April 28, 2023, https://aappb.org/wp-content/uploads/2022/06/Special_Condition_Data_Definition-2-Feb-2023.pdf

# ### Clean the age column

# In[457]:


victims_df["age"] = victims_df["age"].astype(str)


# In[458]:


victims_df["age"].unique()


# In[459]:


#lets see the count for each age value
age_counts_df = victims_df["age"].value_counts().to_frame().reset_index()
age_counts_df.columns = ['age', 'count']
age_counts_df


# In[460]:


#There are some cases that have AGE+, there are very few of them 
#We can eliminate the "+" and assume they refer to the age
victims_df["age"] = victims_df["age"].str.replace("+", "", regex = True)
victims_df["age"] = victims_df["age"].str.replace("-", "", regex = True)


# In[461]:


#Substitute "Unknown" with nulls
victims_df["age"].replace("Unkno…", np.nan, inplace=True)
#Replace other string vals
victims_df["age"].replace("2 years…", "2", inplace=True)
#Everytime to "mo" str appears, convert to 1 year
victims_df["age"] = victims_df["age"].str.replace(r"(\d+) mo.*", "1", regex=True)


# In[462]:


victims_df["age"] = victims_df["age"].astype(float)


# In[463]:


# Create distribution histogram
victims_df["age"].hist(bins=20)

# Set plot labels
plt.xlabel("Age")
plt.ylabel("Count")
plt.title("Age Distribution")
plt.show()


# There are not outlier values in the Age column, the column is clean

# ## Verify that the township names of the AAPP dataset (victims) match the township names of the township shapefile

# #### Read the townships from the feature table of shapefile

# In[464]:


township_df = pd.read_excel("Townships_data.xlsx")


# In[465]:


shapefile_list = list(township_df["TS"].unique())
print("This is a list of all townships in the shapefile")
for town in shapefile_list:
    print(town)


# #### The following code lines iterate through all the township names in the victims_df, aka aapp_df, and find the closest match with respect to all the township names included in the shapefile. 

# In[466]:


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


# In[467]:


print("It seems that all townships match!")


# ### Perform a join and check the number of strings that did not find any match

# In[468]:


#Create two dataframes that will be used for merging
township_shapefile = township_df["TS"].to_frame() #This contains the original township column of the township shapefile data table
township_aapp = victims_df["township_town"].to_frame() #This contains the original township column of victims_df


# In[469]:


#Create a dataframe that merges the two lists (dataframes) from the townships of the victims dataframe and the townships of the shapefile
#Using the indicator parameter, verify if there was match or not
townships_merged_df = pd.merge(township_shapefile, township_aapp, left_on='TS', right_on='township_town', how='outer', indicator='_merged')


#Both indicates that the township is found in the victims_df and the shapefile dataset
townships_merged_df['_merged'] = townships_merged_df['_merged'].replace({'both': 'Both'})

#Left indicates that the township is only found in the township_shapefile
townships_merged_df['_merged'] = townships_merged_df['_merged'].replace({'left_only': 'Only in shapefile'})

#Right indicates that the township is only found in the township_aapp 
townships_merged_df['_merged'] = townships_merged_df['_merged'].replace({'right_only': 'Only in victims_df'})


# #### Inspect the townships that are the victims dataframe but not found in the shapefile

# In[470]:


only_victims_df = townships_merged_df[townships_merged_df['_merged'] == 'Only in victims_df']
print("\nThese are the townships in the victims dataframe that are not found in the shapefile")
for town in only_victims_df["township_town"].unique():
    print(town)


# In[471]:


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


# In[472]:


#Read the excel file that contains the updated values for the townships and the reasons on how I decided to update the values
townships_to_be_updated_df = pd.read_excel("Townships_updated_values_for_join.xlsx")


# In[473]:


townships_updates_dict = dict(zip(townships_to_be_updated_df['Victims_township'], townships_to_be_updated_df['Match to shapefile']))


# In[474]:


townships_updates_dict


# #### Update the township names in the victims_df so that they can match with the shapefile

# In[475]:


victims_df["township_town"] = victims_df["township_town"].map(townships_updates_dict).fillna(victims_df["township_town"])


# ### In the case of state_region of the victims_df ensure that they also match with the state_region column of the shapefile

# In[476]:


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


# In[477]:


#Create two dataframes that will be used for merging
state_shapefile = township_df["ST"].to_frame() #This contains the original state column of the township shapefile data table
state_victims = victims_df["state_region"].to_frame() #This contains the original state column of victims_df


# In[478]:


#Create a dataframe that merges the two lists (dataframes) from the states of the victims dataframe and the states of the shapefile
#Using the indicator parameter, verify if there was match or not
states_merged_df = pd.merge(state_shapefile, state_victims, left_on='ST', right_on='state_region', how='outer', indicator='_merged')

#Both indicates that the state is found in the victims_df and the shapefile dataset
states_merged_df['_merged'] = states_merged_df['_merged'].replace({'both': 'Both'})

#Left indicates that the state is only found in the township_shapefile
states_merged_df['_merged'] = states_merged_df['_merged'].replace({'left_only': 'Only in shapefile'})

#Right indicates that the state is only found in the township_aapp 
states_merged_df['_merged'] = states_merged_df['_merged'].replace({'right_only': 'Only in victims_df'})


# In[479]:


only_victims_df = states_merged_df[states_merged_df['_merged'] == 'Only in victims_df']
print("\nThese are the states in the victims dataframe that are not found in the shapefile")
for state in only_victims_df["state_region"].unique():
    print(state)


# In[480]:


township_df["ST"].unique()


# The reason why these states do not match is because the victims_df do not distinguish between Bago (East) and Bago (West), same with Shan state <br>
# To deal with this, all the townships inside each state will be retrieved, then in the victims_df, the state_region will be updated according to the lists

# In[481]:


Bago_west_townships = list(township_df.loc[township_df["ST"] == "Bago (West)", "TS"])
victims_df.loc[victims_df["township_town"].isin(Bago_west_townships), "state_region"] = "Bago (West)"


# In[482]:


Bago_east_townships = list(township_df.loc[township_df["ST"] == "Bago (East)", "TS"])
victims_df.loc[victims_df["township_town"].isin(Bago_east_townships), "state_region"] = "Bago (East)"


# In[483]:


Shan_east_townships = list(township_df.loc[township_df["ST"] == "Shan (East)", "TS"])
victims_df.loc[victims_df["township_town"].isin(Shan_east_townships), "state_region"] = "Shan (East)"


# In[484]:


Shan_north_townships = list(township_df.loc[township_df["ST"] == "Shan (North)", "TS"])
victims_df.loc[victims_df["township_town"].isin(Shan_north_townships), "state_region"] = "Shan (North)"


# In[485]:


Shan_south_townships = list(township_df.loc[township_df["ST"] == "Shan (South)", "TS"])
victims_df.loc[victims_df["township_town"].isin(Shan_south_townships), "state_region"] = "Shan (South)"


# In[486]:


#In the case where state_region is still Bago or Shan, it indicates, that the specific region is Unknown, values will be updated accordingly
unknown_regions_dict = {"Bago" : "Unknown", "Shan" : "Unknown"}
victims_df["state_region"] = victims_df["state_region"].map(unknown_regions_dict).fillna(victims_df["state_region"])


# ### Eliminate the rows that have missing values for state and township, i.e. unknown location
# 

# In[487]:


victims_df = victims_df.drop(victims_df[(victims_df["state_region"] == "Unknown") & (victims_df["township_town"] == "Unknown")].index)


# ### The victims_df is ready to be match with the township shapefile based on townships!

# In[488]:


#victims_df.to_excel("victims_township_level_vf.xlsx")


# ## It is necessary to display the count of victims at township level and include the count according to gender and sector

# In[489]:


#Use pivot table to count the number of victims that belong to each level of the categorical vars gender, sector at each township
township_victims_per_sector_df = victims_df.pivot_table(index='township_town', columns='sector', values='id_number', aggfunc='count', dropna=False)


# In[490]:


township_victims_per_gender_df = victims_df.pivot_table(index='township_town', columns='gender', values='id_number', aggfunc='count', dropna=False)


# In[491]:


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


# In[492]:


township_victims_per_age_df = victims_df.pivot_table(index='township_town', columns='age_group', values='id_number', aggfunc='count', dropna=False)


# In[493]:


# Merge the three dataframes based on township_town column
victims_township_df = township_victims_per_sector_df.merge(township_victims_per_gender_df, on='township_town').merge(township_victims_per_age_df, on='township_town')


# In[495]:


victims_township_df.to_excel("victims_aggregate_counts_township_vf.xlsx")


# ## Load the socioeconomic data collected 

# In[201]:


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


# ### Keep only columns that indicate the state, township, township_pcode and the value of the variable

# In[202]:


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


# ### Merge all the dataframes, rows that have the same string characters of Township_Name, Township_Pcode, State_Region will be merged

# In[203]:


# Create a list of dataframes to merge
dfs_to_merge = [ApproxVulPop_2016_df, Wealth_Ranking_Index_df, DisabledPop_2014_df, FemPop_2014_df, MalePop_2014_df, Population_2014_df, PerAdultFemLiteracyRate_2014_df, PerAdultMaleLiteracyRate_2014_df, PerAdultLiteracyRate_2014_df, EmployeeGovTotal_2014_df, FemEmployeeGov_2014_df, MaleEmployeeGov_2014_df, StudentTotal_2014_df, FemStudentTotal_2014_df, MaleStudentTotal_2014_df]

# Merge the dataframes in the list using a for loop
socioeconomc_characs_df = ApproxVulPop_2016_df #Initialize a dataframe, that contains the first dataframe, i.e. ApproxVulPop
for i in range(1, len(dfs_to_merge)): #Iterate through all the indices of the dfs_to_merge list
    socioeconomc_characs_df = pd.merge(merged_df, dfs_to_merge[i], on=["Township_Name", "Township_Pcode", "State_Region"], how="left") #Merge the selected dataframe


# In[204]:


socioeconomc_characs_df


# ### Perform a join and check the number of strings that did not find any match

# In[205]:


#Create two dataframes that will be used for merging
township_shapefile = township_df["TS"].to_frame() #This contains the original township column of the township shapefile data table
township_socioeconomic_df = socioeconomc_characs_df["Township_Name"].to_frame() #This contains the original township column of victims_df


# In[206]:


#Create a dataframe that merges the two lists (dataframes) from the townships of the victims dataframe and the townships of the shapefile
#Using the indicator parameter, verify if there was match or not
townships_merged_df = pd.merge(township_shapefile, township_socioeconomic_df, left_on='TS', right_on='Township_Name', how='outer', indicator='_merged')


#Both indicates that the township is found in the victims_df and the shapefile dataset
townships_merged_df['_merged'] = townships_merged_df['_merged'].replace({'both': 'Both'})

#Left indicates that the township is only found in the township_shapefile
townships_merged_df['_merged'] = townships_merged_df['_merged'].replace({'left_only': 'Only in shapefile'})

#Right indicates that the township is only found in the township_aapp 
townships_merged_df['_merged'] = townships_merged_df['_merged'].replace({'right_only': 'Only in victims_df'})


# In[207]:


only_victims_df = townships_merged_df[townships_merged_df['_merged'] == 'Only in victims_df']
print("\nThese are the townships in the victims dataframe that are not found in the shapefile")
for town in only_victims_df["Township_Name"].unique():
    print(town)


# In[208]:


#Read the excel file that contains the updated values for the townships and the reasons on how I decided to update the values
townships_to_be_updated_df = pd.read_excel("Townships_updated_values_for_join_socioeconomic_vars_and_shapefile.xlsx")


# In[209]:


townships_updates_dict = dict(zip(townships_to_be_updated_df['Socioeconomic_df_Township'], townships_to_be_updated_df['Match to shapefil']))


# #### Update the township names in the socioeconomic_df so that they can match with the shapefile

# In[210]:


socioeconomc_characs_df["Township_Name"] = socioeconomc_characs_df["Township_Name"].map(townships_updates_dict).fillna(socioeconomc_characs_df["Township_Name"])


# #### Drop the rows that contain names of townships not found in the shapefile

# In[211]:


socioeconomc_characs_df = socioeconomc_characs_df[~socioeconomc_characs_df["Township_Name"].isin(["Minhla (B)", "Keshi", "Hlaingtharya", "Htantabin (Y)"])]


# ### The socioeconomic_characs_df is ready to be match with the township shapefile based on townships!

# In[212]:


socioeconomc_characs_df.to_excel("socioeconomic_vars_township_level_vf.xlsx")


# 
