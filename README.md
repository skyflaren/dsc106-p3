We choose to look at a dataset from the World Bank that measures various Governance Indicators, that measure the quality of various countries’ governments. The information that we deemed most important in the data set was the 6 governance indicators (Control of Corruption, Voice and Accountability, Government Effectiveness, Political Instability, Regulatory Quality, and Rule of Law) for each year for each country. We decided that for this project since we were able to include an interactive aspect we would be able to display all of the governance indicators using color shading in the form of a choropleth. Similarly, we also chose to use the years as an interactive component by allowing users to view data from each year by selecting the year using a slider. Finally, we decided to encode the country data geographically through our choropleth where we could show data for distinct countries on the map itself. We came to these decisions by discussing ideas and then looking at some global visualizations that we thought were effective, including Our World In Data’s Global Energy visualizations, where we got the idea of setting the years as an interactive slider. Before coming to this idea, we considered averaging the data over the years to include information from each year on the map.
We split up the work for the development of the visualization by breaking apart the components that we needed. Justin worked on styling the visualization, optimizing the code, and adding the data source. Angela worked on the data preparation, choropleth, and tooltip. Colin worked on the legend and slider for different years. We started with the Svelte template and then worked on cleaning the dataset from the World Bank with the pandas library. Since there were multiple separate datasets for the different governance indicators, we had to take steps to melt and merge the datasets. Then, we added the cleaned dataset to the GitHub repository and used the data for the choropleth. To display the map for the choropleth, we used the topojson library and world map json file that we found online. Since each of the governance indicators values are between -2.5 and 2.5, we displayed the range through a blue gradient scale for the choropleth and added a legend for the scale, with blue in color theory representing authority and trust. We also needed to separate the data into the different indicators and years using a dropdown menu selector for the indicators and slider component for the years. Lastly, we added the tooltip to allow users to see the exact values for the indicators. For most of these elements, we used a combination of d3.js and HTML. We also styled the visualization with custom font and colors with CSS. 
Overall, the development of the project took around 15 hours. The components that took the most time was the choropleth visualization and the legend since it was difficult to find up-to-date documentation for the d3 library. 
