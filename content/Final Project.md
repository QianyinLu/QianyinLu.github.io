Title: Create an interactive dashboard for COVID19
Category: Dashboard
Date: 2020-11-15 11:00
Modified: 2020-11-18 14:30
Tags: Dashboard, Visualization, Streamlit, Maching Learning
Slug: finalproject
Authors: Presnie Lu
Summary: Build machine learning models for daily total cases growth rate and create a dashboard 
og_image: https://image.freepik.com/free-photo/white-medical-mask-sky-blue-pink-background-face-mask-protection-against-pollution-virus-flu-coronavirus-health-care-concept-selective-foocus-copy-space_215497-39.jpg

**The goals of this project are mainly two:**   
**1. Generate machine learning models to predict daily total cases growth rate and explore which factors influence this the most.**   
**2. Create an interactive dashboard that help readers visualize the general situation of Covid19 from the entire US, state and individual level.**   
**3. Analyze how policy influence the case growth rate.**   
**In this article, I will mainly focus on what I did in this project and what I learned from it. In addition, I will offer a brief walkthrough for our final data product, which is the dashboard. **

To begin with, please take a look at our [dashboard](https://final-project-823.herokuapp.com/) and [github](https://github.com/YinJishen/BIOSTAT823-Final-Project).   
In summary, our dashboard is consisted of four parts. The first part provides readers real time update of COVID19 new/total and death cases for the entire United states and individual states. For the next part, we compared the effect of policies between state. The method we used is to calculate the difference of increase rate of total cases within a 7/15 day ranges. In this part, we also implement comparison of individual state's increase rate to the average of entire U.S. The third part used random forest models for predicting daily case growth rate for aggregate U.S. level and also for each individual state based on hospitalization, policy and demographic data. The last part implemented a stratified sampling methodology based on age group to get partial sampling from our huge dataset. Using such data, we analyzed what factors, such as race and sex, influence the individual level infectious and death rate. This part also helps us explore further the demographic composition effect for state level from part three.   
    
In general, the target audience for our project is the group of people who are non-statisticians. Therefore, our dashboard uses mostly interpretable plot to show the model results. In addition, our dashboard also allows users to choose which area they are interested in with a lot of freedom by using dropdown menu or selection box. Therefore, even if the audience does not come with a goal to analyze data, the dashboard can still serve as an informative tool and search engine for people to learn any related information aboout COVID19.    
        

# PartIII: Predict Covid19 Case Daily Growth Rate
One big focus of this project is to generate model(s) for predicting the daily growth rate of COVID19 cases. I was mainly charged for doing this part during the process. By cooperating with one other team member, we finally decided to generate models from two levels: the entire US and state level. The specific session that represents our results is **Part III: Predict Covid19 Case Growth Rate** from our [dashboard](https://final-project-823.herokuapp.com/).

## Data Source
In terms of Covid19 cases data, we mainly got the data from the [Centers for Disease Control and Prevention](https://data.cdc.gov/). We also decided to use make predictions from these three perspectives: [Hospitalization](https://www.cdc.gov/nhsn/covid19/report-patient-impact.html#anchor_1594393649), [Demographic](https://www.census.gov/data/developers/data-sets/acs-1year.html) and [Policy](https://github.com/USCOVIDpolicy/COVID-19-US-State-Policy-Database). All these databases are available online and readers can easily pull the data through our code from the data folder in Github. Among these three, I was reponsible for get the statewise demographic and policy data.   

## Model Building 
There are two parts of models we decide to build. The first part of the model is using all three perspectives of the variables as I mentioned and predict the overall case growth rate for all states together. For this model, the input of our data is all states' related information from 2020 April to July. The second part is using only hospitalization and policy data as independent variables to predict the daily case growth rate for each state. Therefore, in this part, we actually built different models for different states. The data range is the same (2020 April to July) for all states. The reason for removing the demographic information is that we assume for individual state, the demographic information, such as age group, race and population density will not change too much within the available date range. Therefore, such information will be less useful to predict the change.   

### Data Preprocessing
Apart from mapping different sources of data into consistent state and date levels, we also did some extra work before building the model. One biggest change we made was to the policy data. The raw data only records the specified date for policies to start or end. Therefore, in order to convert such information into a binary indicator (for instance, wether a state in day 10 is under the policy where it requires all businesses to close), I wrote a helper function to convert all relevant policies into dummy variables. This also explained why we did not remove this variable from the second part because for individual state, the policy variable actually varies across different days. We also realized that there are too many policies availble. To avoid overfitting problems, we decided to pre-select some features base on 1. our personal interest and 2. how did the policy variables change across state or overtime. (For instance, we did not include declaration of state of emergency because it is mostly executed for all states before April.)

In addition, notice that for both parts, the data are time-relevant. Thus, in order to de-trend and make the data more stationary, we decided to add lag variables of case growth into the model. Similarly, we added some helper function to help us generate such variable more efficiently.

### Model Building
For both parts, we decided to use random forest as our model. There are two reasons: 1. Random forest tends to generate more accurate prediction results. 2. Random forest is able to offer feature importances conveniently. After choosing the model, we decided to split the data into training and test set. As we realized that we have time series data, it will be more wise to split the data base on time order. Then, we chose the all data before "2020-06-01" as our training set and the rest of data as our test set. We also chose to log scale some of the discrete variables from the hospitalization perspective and scale all our data using standard scaler so that the scale of data will not influence the final outcome. Finally, we decided to use RMSE as our accuracy matrix. 

## Dashboard Product 
The dashboard was divided into two parts base on the model. The first part is called overview and the second part is a state level view.

### Overview
This part of the dashboard is used to show readers our prediction result for model1 - the entire US level. The goal of this part is to show which features are important to case daily growth rate for all US as a whole. The top level presents the total prediction error (RMSE) for training and test data. The lower level is a feature importance plot. The reader can adjust manually on the left sidebar on how many features they want to see.  
![US_feature](\img\US_feature.png "US Feature Importance")  
From this plot we can see, hospitalization is the most important feature that determines the change of case increase rate. In addition, state's race composition is also relatively importance as we can see that the ratio of African American and white were the top 10 important features among all. Similarly, the age group for each state is also somewhat important.   

### State Level
This part of the dashboard is used to see each state's case growth rate prediction result as well as state specific demographic information. In general ,this part is divided into three perspectives. The first part shows the prediction error for training and test set for the selected state. The second part, as the following, shows the actual value of the actual case growth and our predictions using the model. Readers can choose which period to look at by clicking on the left sidebar. In addition, if the reader wants to see the specified values of prediction and actuals, he can just move the mouse on it and a bubble contains the tooltip information will automatically appears. The following is an example of Virginia's prediction.   
![State_level](\img\state_prediction.png "State Level Predictions")  
From the plot we can see that our predictions mostly align quite close to the actual line. Our predicted values showed consistent trend with the actual growth rate. The only difference is that our model's result seem to be a little more stable. Overall, we see the growth rate as a whole is getting smoother and more stable as time goes and the scale was much smaller. This means COVID19 case increase in Virginia is much slower in June compared to April and May.   
Similar to Overview, we put a feature importance plot for state level model. Readers can also choose how many features to look at using the left sidebar. Different from overview, we exclude the state demographic information in the plot due to the reason I mentioned in the modeling part. What's more, we removed some variables that have really minimal feature importance score. An example of Minnesota's feature importance plot is shown below.   
![MN1](\img\MN.png "MN feature importance")  
From this importance figure, we got a similar conclusion from the US level plot. Hospitalization is the most important aspect among all. On the other hand, policy's effect is actually quite small, even minimal. This can be caused by the fact that policy tend to be in effect after a periods of time. Therefore, even at the start date our dummy variable indicates 1, the policy hasn't been in effect on the case growth rate yet. Such confusing information will make the model think such variable is not important. Therefore, to further explore the effect of policy across states, we developed our Part II, which uses data within 7 days that the policy starts.   

Considering that the reader may curious about the demographic information about this specified state, I added a session called State information to show visually what is the demographic composition of the state. This allows the reader to also relate this state level information with the overview part. The following is an example of Minnesota's demographic information.
![MN2](\img\sex.png "MN sex ratio")   
![MN3](\img\race.png "MN race ratio")  
From the plot we can see the sex ratio of Minnesota is quite balanced. In addition, from the race ratio pie chart, white is the majority of the state. We can then relate this information and infer that the COVID19 growth rate in MN is different from other states due to the difference of white population percentage in the state.   
        
When readers clicked the checkbox "Show me more data", a complete table of this state's demographic information will be shown as well as a bar plot showing the age groups ("Above 65" and "Under 18"), which are the two groups we believe will be affected by COVID19 the most. 
![MN4](\img\More.png "More data")    


**Then, I will introduce the rest parts of our dashboard. I was also engaged in a lot of data collection and plotting process, but I won't go into too many details about it. **

# Part I: Overview

This part is intended to offer an overview of COVID19 from a state and country level. This page is also always updated to the most recent available information from the CDC because it pulls information directly using API from the CDC. Therefore, the reader can feel free to choose any date by change the data input from the left sidebar.   
    
For the first part, readers can choose to focus on a state. The dashboard is able to provide the new cases and new death of this state. I will use North Carolina as an example here.   
![NC](\img\NC.png "North Carolina COVID19")    
From this plot we can see that the number of new cases has been increasing largely from April to July. This is showing that for North Carolina's overall COVID19 situation is getting worse. While luckily, the death case remained at a relatively stable level. This plot possibly indicates that North Carolina is doing good in medical care for COVID19 patients while the government and people did not follow strictly the policy that prevented the spread of virus.   
    
For the second part, the dashboard offers an overview of the most updated COVID19 situation for the entire United States as a map. The map is also interactive that readers can put cursor on the map and they can see the specified number and state name on the map. The readers can choose which data type to focus on from the left sidebar. I attached the plots of new cases and death per million people map as the following:   
![US_NEW_CASE](\img\map_new_cases.png "US New Cases")   
![Death per_million_people](\img\death.png "Us Death per million")   
From the plot, it is easy to observe that for the last day CDC updates the information, California, Texas and Illinois are the most severe states with the most new cases increase. In addition, in terms of death rate, New Jersey and Massachusettes are the worst states with highest death per million people. This indicates that these states' hospitalization system might not be as good as others or the demographic composition of these states, such as over 65 population leads to high death rate. 

# Part II: State Level Comparison

As I mentioned in part III, because the policy variable in the random forest can hardly capture the real effect of policy, we instead built a separate session to evaluation the effectness of policy across states. In this session, readers can choose from the left sidebar which policy, day range, how many states and which date to look at.    
    
The first part is using the within certain day ranges (7) to see the short term and day ranges(15) as the long term effect of certain policy. By default, readers should see the top 5 and last 5 states evaluated by policy's performance. To be more specific, the performance is calculated by difference of case increase rate. (e.g. Average of the case growth rate 7 days after the policy started - Average of the case growth rate 7 days before the policy started) An example of stay at home order range by 7 days is shown below:  
![Stay](\img\stay.png "Stay at home order")   
According to the chart, the higher decrease of the growth rate, the better the policy is. On the other hand, the worst state actually has still increasing growth rate. This means that such policy in certain state has a negative effect. This can happen for two reasons: firstly, the government does not implement the policy with certain punishment for violation, thus, most people chose to ignore it; secondly, the policy took place too late: there are too many cases exists and the policy itself can no longer handle such scenario, therefore, the effect of policy is unable to offset the rapid increase of the new cases.   

The second part is actually less relevant to the policy itself. This map gave people an overview of how each state's case increase rate compared to the entire United States. The reader can choose any available data and the map will reflect the specified number of increase rate of each state (by moving the cursor) and that day's overall US increase rate. Because the color of the map is determined by the State's increase rate - US's average increase rate, the darker the state's color, the worse the COVID19 in that state. I will show an example of the increase rate in 2020-05-28:  
![Inc_rate](\img\relative_inc.png "Relative Increase Rate")    
From this map, Minnesota was doing the worst among the entire US in 2020-05-28 because it's case increase rate is much higher than the average of the US. Actually, we can then easily relate this information with our Part III, which I gave an example of Minnesota as well for their demographic information. By looking at the demographic composition as well as the feature importance plot for Minnesota and US, apart from the crucial hospitalization information, we also suspect that the whether this state has higher proportion of white population can lead to the especially high daily case growth rate. Therefore, in our last Part IV, we will do an analysis from the individual level base on race and sex and see how death rate and infectious rate is related to that. 

# Part IV: 
Due to the lare amount of the dataset, we initially process the data to the SQL database. For convenience and efficiency, we do not want reader to wait for too long to load the visualization. Therefore, we decided to use only partial data. The sampling method we used is stratified sampling base on age groups. The basic idea is that after the sampling process, the proportion of each age group should still be the samge compare to our original data and our final tree map looks like this:  
![Treemap](\img\treemap.png "Tree Map")    
From this we can see how many people are in each age group and the area of each square represents what proportion each age group is compared to the entire sample. You can see the comparison of before and after sampling by clicking the introduction part from the left sidebar. 
        
After this sampling method, we did some analysis using bar plots for infection and fatality cases base on age, sex and race. The following is a example of analyzing infection and death cases using sex and race:   
![Sex_race](\img\sex_race_inf.png "Sex and Race, infection count")    
![Sex_race2](\img\sex_race_death.png "Sex and Race, death count")
From the perspective of total cases, White/Non-Hispanic as well as Black/Non-Hispanic have the largest infection count. While from the perspective of infection rate, American Indian is the highest. We believe the reason behind this is that Native Americans' culture were to gathered together and live as a group. Such activities made them to get infected easily. On the other hand, when looking at death cases, there is obvious difference between the number of cases of female and male, that male has more death cases for almost all races. Among all, Asian has the lowest death rate; In addition, Hawaiian has a higher rate of death compared to infection rate, this might be due to the limitation of data size.    
    
Then, this is another example of analyzing age groups and sex:   
![age_inf](\img\age_inf.png "Sex and Age, infection count")    
![age_death](\img\age_death.png "Sex and Age, death count") 
In general, infection rate is quite low for young people. Actually, it increases as population's age increases until 70, indicating younger population’s immune system do perform stronger. For people older than 70, infection rates decrease, and this might be caused by the limitation of their ability of outdoor activities, and this prevents them from many dangerous cases.Compared to infection cases, the death cases has a more obvious and straight information: people over 70 have a much larger death rate, they have a very high possibility of death once they got COVID19.



# Key Insights from the project

## Findings and Conclusions
The detailed insights and analyzation were provided as examples before in my explanations for separate parts. The following are some key insights:

1. According to the most updated information, now, in terms of total/new cases, states like California, Texas and Minnesota are the most severe states; In terms of death per million people, Massachussets and New Jersey are the most severe ones.
2. Policy effect usually takes more than a week to be truly in effect. In addition, some policies might not be effective or even have opposite effect on the new case growth rate. 
3. For individual states level and the entire U.S. level, hospitalization factors are the most important factors to predict daily new case growth rate. With the combination of part II's observation, policy usually have lagged effect, therefore, our model did not capture the obvious effect of policies. We assume two reasons behind this: State's new case increase rate was too high and the spread of COVID19 was out of control that the policy can hardly stop this; State government does not implement the policy with punishment, therefore, the policy was not strict enough to be truly in effect because people did not take it seriously(Compare to Chinese government.)
4. Infection rate has large reason to do with each race's culture, for instance, Native Americans and Hispanics(recorded as others in partIII) because they tend to gather together. On the other hand, older people tend to get higher chance of being both infected and death because of their immune system. 

Overall, our data product provided some interesting views and perspectives on analyzing the policy effect statewise and country-wise. Although the process might not be robost enough, according to our analysis, state government and the U.S. government should definitely try harder to improve the effectness of COVID19 policies. The way they should approach to better policy needs to vary area by area according to each states' demographic and economic condition.   
    

## Skills
Thanks to my teammates, we learned a lot of things from each other:

1. Practice using APIs from a variety of websites     
2. Model improving by scaling and transforming variables   
3. Deeper understanding of Time series models   
4. Practice of interactive plots in our dashboard   
5. Learn to find interesting insights from "not-so-important" factors (policy) and more practical understanding of the model


**Finally, thanks to all of my teammate for their efforts and amazing ideas. Also, it was a great experience studying BIOS823 this semester, I learned so much practical machine learning and deep learning skills.**

# Full Reference

1. COVID-19 cases in US from January till now:   
  https://data.cdc.gov/Case-Surveillance/United-States-COVID-19-Cases-and-Deaths-by-State-o/9mfq-cb36   

2. COVID-19 patient-level data:  
  https://data.cdc.gov/Case-Surveillance/COVID-19-Case-Surveillance-Public-Use-Data/vbim-akqf   

3. hospital resources (Hospital Capacity by State):       
https://www.cdc.gov/nhsn/covid19/report-patient-impact.html#anchor_1594393649  

4. COVID-19 US State Policy:      
https://github.com/USCOVIDpolicy/COVID-19-US-State-Policy-Database  

5. State demographic information:      
https://www.census.gov/data/developers/data-sets/acs-1year.html

