# Assignment 2: Create a custom map

## **Background information:**<br> 
World Resources Institute (WRI) is a global research non-profit organization focused on sustainability. WRI works on seven areas: food, forests, water, ocean, cities, energy, and climate. WRI works under a threefold approach:
•	Count it
•	Change it
•	Scale it 
The organization emphasizes the importance of conducting unbiased research and using data to understand sustainability issues. 
One of the WRI Energy Program’s main tools is the Energy Access Explorer (EAE). The EAE is an online, open-source, interactive platform that allows decision-makers in Sub-Saharan countries to visualize data related to energy demand and supply. When using the EAE, decision-makers can quickly construct heatmaps to identify underserved areas or areas with high potential for renewable energy generation. 


## **Objective:** <br>
Currently, the EAE Platform is available in Ethiopia, Nigeria, Kenya, Uganda, Tanzania, Zambia, and India. However, the EAE is willing to increase its outreach to countries with a large population without energy access. The purpose of this assignment is to build a custom google map for South Sudan, the least-electrified country in the world in 2020. 


## **Design:**<br>
The color palette created for this map is built upon the colors used by the EAE and by WRI. The first image contains the methodology used by the EAE to create its geographic analysis, and the second image displays the home page of WRI. 

![image](https://user-images.githubusercontent.com/52460741/227830846-270ad644-697f-4b7a-8616-cef4325afac0.png)
![image](https://user-images.githubusercontent.com/52460741/227830940-982989ac-f5ef-47c3-8d76-4b1342feabdc.png)

Finally, this is the color palette: 

![image](https://user-images.githubusercontent.com/52460741/227831095-5c6dceec-8b84-40f5-adf8-3935e2d18e8c.png)


### **Zoom at country level**
WRI works in different countries of Africa. 

![image](https://user-images.githubusercontent.com/52460741/227831439-7e044303-aa9d-4a41-9ccf-ae16feed2447.png)


### **Zoom at specific country level (South Sudan)**
For the EAE it is important to know the location of villages (yellow). Moreover, electricity access is associated to the presence of physical infrastructure, e.g. roads, hence the highways are highlighted in black, whereas roads are highlighted in grey. Some villages may not have access to highways, and this would be a hint to examine the energy access level in this village. 

![image](https://user-images.githubusercontent.com/52460741/227831588-fb3afc26-4fe6-4e47-8413-1fa1f3d0356b.png)


### **Zoom at city/village level**
The EAE emphasizes the importance of locating social infrastructure buildings, e.g., schools, hospitals, government buildings. This is because the presence of these buildings is associated to an area with high demand of energy. The image displays the capital Jurba, however, there are villages in where hospitals and schools but still have unreliant access to energy. 

![image](https://user-images.githubusercontent.com/52460741/227832964-9089d255-8144-4501-906a-584ed42ba476.png)

Moreover, there may exist underdeveloped areas in urban settings that lack access to energy. Hence, the EAE would need to know to ubicate those areas. In this regard, knowing the street names is relevant. However, for South Sudan not all the names of streets are available. This image shows the main street "Nyggilo". 

![image](https://user-images.githubusercontent.com/52460741/227833632-ec11e9b0-e089-4fb6-8940-9a74bba4a3bd.png)

### **Lookup table**<br>
The map was created using [Google Map Styling Wizard](https://mapstyle.withgoogle.com/). The base theme was "Dark".  The following table contains the map features and their styles. 



## **Downloadable JSON file**<br>

[Click here](./Code.json)
