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

| Feature type | Element type | Stylers |
| ------------ | ------------ | -------------------------- |
| All | Geometry | Color: Dark Green #008D6A |
| All | Labels / Text fill | Color: Yellow #F3B229  |
| All | Labels / text outline | Color: Black #000000, Weight 2  |
| Country | Geometry / Stroke | Color: Light Green #96BC33  |
| Province | Geometry / Stroke | Color: Light Green #96BC33  |
| Land parcel | Labels / Text fill | Color: White #ffffff  |
| Landscape / Human-made | Geometry / Stroke | Color: Orange #F7941D  |
| Landscape / Human-made | Geometry | Color: Dark Green #008D6A  |
| Landscape / Natural | Geometry | Color: Dark Green #008D6A  |
| Points of interest | Labels / Text fill | Color: #ffffff  |
| Points of interest | Labels / Text fill outline | Color: Grey #94948b Weight 1  |
| POI / Park | Labels / Text fill | Color: White #ffffff  |
| POI / Park | Labels / Text outline | Color: Grey #94948b Weight 1  |
| POI / Park | Geometry / Fill | Color: Opaque blue #023E58  |
| Road  | Geometry | Color: Grey #94948b |
| Road  | Labels / Text fill | Color: White #ffffff |
| Road  | Labels / Text outline| Color: Grey #94948b Weight 1 |
| Road / Highway  | Geometry / Fill | Color: Black #000000 |
| Road / Highway  | Labels / Text fill | Color: White #ffffff |
| Road / Highway  | Labels / Text outline | Color: Grey #94948b Weight 1 |
| Transit  | Labels / Text fill | Color: White #ffffff |
| Transit  | Labels / Text outline | Color: Grey #94948b Weight 1 |
| Transit/Line  | Geometry / Fill | Color: Grey #94948b Weight 1 |
| Transit/Station  | Geometry | Color: Grey #94948b |
| Water  | Geometry | Color: Light Blue #0291C2 |
| Water  | Text fill | Color: White #ffffff |


## **Downloadable JSON file**<br>

[Click here](./Code.json)

### **Instructions for using the JSON file**<br>
The user will need to download the file containing the json code, then will need to open google wizard maps and create a new project. When creating a project, the option "Import JSON should be selected". Then the code should be pasted. Hence, the style will be imported and the user can add more features to the map. 


### **Reflection**<br>
This task allowed me to understand the importance of choosing a right color palette that reflects the organization's mission. Regarding time allocation, I expected a long time for creating the color palette, as this would require reading about the organization's work and getting familiarized to their colors. However, a time consuming task was creating the HTML on github, as it required me to get more training on how to use github, especially how to embed links. Overall, I expected to use 10 hours on creating the map, nonetheless the final time required was 11.5 hours. This is because I needed to learn more about Google Map Styling Wizard and Github. 

The following table contain the estimated hours for each requirement of this assignment. 
![image](https://user-images.githubusercontent.com/52460741/227836923-01937b07-18d5-4769-b32f-f4de4828e30c.png)

The following table contains the actual hours invested for each requirement of this assignment. 
![image](https://user-images.githubusercontent.com/52460741/227836955-c7d8a39b-c460-46ed-9630-82b0cd4d8b05.png)


