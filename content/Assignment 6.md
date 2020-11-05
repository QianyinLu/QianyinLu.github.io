Title: Create an interactive dashboard using Streamlit
Category: Dashboard
Date: 2020-11-04 12:01
Modified: 2020-11-05 16:30
Tags: Dashboard, Visualization, Streamlit
Slug: streamlit
Authors: Presnie Lu
Summary: Create a dashboard for PhDs awarded in the US.

**This blog is used to show some basic analysis on the dataset [PhDs awarded in the US](https://ncses.nsf.gov/pubs/nsf19301/data). We will also create an interactive dashboard using [Streamlit](https://www.streamlit.io/). The main goal of the dashboard is to provide some interesting perspectives of the recipients and institutions as well as offering people the ability to search for or focus on the specific location, sex, field of study and universities they are interested in. Feel free to play around in this [Dashboard](https://streamlit-pre.herokuapp.com/). **

# Dataset
In this project, I chose to focus on three datasets. The first dataset is intended to give people an overview of how recipients and institution number change over time. Therefore, I chose Table 2 as my first data source. The second table to give people a detailed view of recipients. I decided to take a look at how recipients from each state different from each other in terms of Sex and Field of Study. My goal was to enable people to compare whichever states they want. Therefore, I chose Table6 as my second data source. Lastly, I'm curious about if there's difference between Doctorate-granting institutions, that is Universities, in terms of field of studies. Therefore, I chose Table4 as my last data source. 

# Streamlit 
To build a dashboard, I chose to use streamlit. After installing streamlit in your computer, all you have to do is to follow the hint from the [documentation](https://docs.streamlit.io/en/stable/api.html) and build customized widgets using python by your own. I won't go into details about it, but you can take a look at my [GitHub](https://github.com/QianyinLu/streamlit/blob/main/app.py) to see how my dashboard is built. For the sake of convenience, I did not use jupyter notebook in this project. Please refer to the python file in github (app.py) and I included all data cleaning as well as manipulations within that file. 


# Dashboard and Analysis 
## Overview
Table2 is used to generate the **Overview** Session. Since the base number of recipients and institutions are hugely different, I decide to instead calculate their %change from previous years. To do this, I used **shift** to generate a lagged value and calculated the percentage change base on this. 

    :::python
    #caculate percentage change
    df = pd.read_excel("overview.xlsx", skiprows = 4)
    df = df.iloc[:,:3]
    df.columns = ["Year", "Doctorate-granting institutions", "Doctorate recipients"]
    df["Institutions - Percentage change"] = (df.iloc[:,1] - df.iloc[:,1].shift(1) )/df.iloc[:,1].shift(1) * 100
    df["Recipients - Percentage change"] = (df.iloc[:,2] - df.iloc[:,2].shift(1) )/df.iloc[:,2].shift(1) * 100

Then, to see whether the number of Recipients will change with the number granting institutions, a line plot was used with x as year and y as percentage change. I used an altair line plot and add some layers on it so that when people put mouse on the line, they can also know the specific value of that percentage change. In addition, the overview page also allows readers to choose whichever feature(recipient/insititution/both) or time range they want to see. 
![Overview](/img/overview.png "Overview")   
From the plot, we can see that the change of Recipients does not follow the exact trend of that of institutions.In fact, the change percentage for both features does not show obvious (either increase or decrease) trend from year 1974 to 2017. However, compare to the change rate of Institutions, Recipients' change fluctuates much more. We can observe that from 1990 to 1999, there is a significant decrease in percentage change of the recipients number while the change of institution number is fairly stable. Therefore, we can conclude that there is no direct relationship between the percentage change of the total number of recipients and that of institutions.

## Recipients
In this session, I want to see whether there's significant difference between the number of Male and Female recipients and more specifically, I want to see whether such difference exists in various field of study and accross states. This is to see whether there's the well-known Simpson's paradox exists in terms of recipients' sex.   
I used Table6 in this session. In this data set, there are some missing values due to the confidential information. Originally, they were recorded as "D", I changed them into NaN now for better understanding. It is easy to notice that I melted the wide table to a long table for the convenience of plotting. 

    :::python
    #melt the dataset
    re = pd.read_excel("re.xlsx", skiprows = 4)
    re.iloc[0,0] = "United States" 
    re.columns = ["State", "Total Male", "Total Female", "Life_sciences Male", "Life_sciences Female",
                 "Physical/earth_sciences Male", "Physical/earth_sciences Female",
                 "Math/CS Male", "Math/CS Female", "Psychology/social_sciences Male",
                 "Psychology/social_sciences Female", "Engineering Male", "Engineering Female",
                 "Education Male", "Education Female", "Humanities/arts Male", "Humanities/arts Female",
                 "Others Male", "Others Female"]
    re.iloc[:,1:] = re.iloc[:,1:].apply(pd.to_numeric, errors = "coerce")  
    melted = re.melt(id_vars = "State", var_name='category', value_name='Number of People')
    cat_split = melted.category.str.split().apply(pd.Series)
    melted["Field of Study"] = cat_split.iloc[:,0]
    melted["Sex"] = cat_split.iloc[:,1]
    state = melted[["State", "Field of Study", "Sex", "Number of People"]]

For this session, I want to give people the ability of choose which state they want to focus on. (Note that if they only care about the entire US, there's an option to just look at the totals by choosing "United States" in the select box.) In addition, people can add or remove any field of study base on their interest. After people decide which state to focus on and what field(s) of study to look at, they can also pick which other state to compare with. With all these options, the dashboard can provide visualizations of number of recipients by Sex and field of Study as well as a view of the dataset of the specified state.  
   
I will use United States as an example.   
First, let's look at the total number of male and female in all fields of study for the US.(This can be seen from the "Show me the data" button):
![US Total](/img/re_total.png "US Total")   
From this we can see the total numbers of Males and Females do not vary a lot. The number of Male is a little bit higher the number of females. However, this is not the case for each field of study.  
![US Total By Fields of Study](/img/re_us.png "US Total By Fields of Study")   
Firstly, The difference between males and females within each field of study is much more obvious comparing to what we saw from the total population aggregated on all fields of studies. We can also observe from the plot that there are relatively huge differences between number of males and females for each fields of study. For Engineering, Math/Computer Science and Physical/health Sciences, the number of Males is much larger than the number of females. While for Education, Psychology/social science, the number of females is larger than the number males. This actually fits our common sense that males are usually better at hard science and females are better at soft science.  
Then, I wonder whether the difference would be different accross states. I will use North Carolina as an example. (To get this plot, choose North Carolina in "Compare with another state" from sidebar and click the checkbox "Compare with another state")  
![US vs NC](/img/re_compare.png "US VS NC")   
It is very interesting to see from the plot that the distribution of males and females as well as fields of study for North Carolina actually resembles the situation of entire United States. Firstly, we see Life Science is the field with most recipients for both the entire country and NC. What's more, although the number of recipients aren't exactly the same, the relative ranks of fields by number of recipients are exactly the same. In addition, the proportions of males and females for each fields of study are also the same for the entire US and NC. With such information, in the future, when we try to explore distribution or characteristics of recipients of the entire country, we can simply draw samples from North Carolina because it fits the general situation of the US. 

## Institutions
This session is intended to compare the institutions for each field of study. The data used is from table4 and therefore, all information is constraint to the Top20/21 universities. Notice that some Universities does not have a value for certain Study Area, this does not mean that they do not have recipients in that area. Instead, it means they fail to be in top 20/21 universities for that specific field of study. In addition, we should pay attention to the variable **Rank** from the dataset skips positions after equal ranking. Therefore, universities with the same amount of recipients have a same ranking. That is why we have either 20 or 21 universities.  
Notice that the orignal data table was a nested table with fields of study listed ahead of top institutions. Therefore, I did some manipulations to clean up the table:

    :::python
    #cleanup the data
    ins = pd.read_excel("institution.xlsx", skiprows = 3)
    subjects_all = list(set(ins[ins.Rank == "-"]["Field and institution"].values))
    subjects = [i for i in subjects_all if "From" not in i]
    ins.loc[:,"Study Area"] = 0
    for i in subjects:
        a1 = ins[ins["Field and institution"] == i]
        idx = int(a1.index.values)
        if "20" in ins.loc[idx+1,"Field and institution"]:
            ins.loc[idx+2:idx+21,"Study Area"] = i
        else:
            ins.loc[idx+2:idx+22,"Study Area"] = i
    ins_total = ins[ins["Study Area"] == 0]
    ins_sep = ins[ins["Study Area"] != 0]

In general, the session is used for two main purposes. The first purpose is to search for top universities for each field. Readers can input any field they like and restrict the number of output. The "Information of the Field and Top Universities" will then render the general information of that field (total number of recipients) and the restraint number of top universities in that field. The second goal is to compare as many universities as you like. The scatter and bar plot basically shows you not only the performance of that university in the field but also comparison in an aggregate level. The scatter plot also has interactive tooltips so that readers can know the detailed information of each point by put the mouse on it.  
   
I will use Cornell, Yale and Harvard University as an example.  
![University Comparison](/img/top_u.png "University Comparison")   
From this plot we can see that Harvard has the most recipients in total among the three. It is consistent with the situation for each seperated field of study. For instance, Harvard has a much higher number of people in Humanities and arts and Life Sciences compare to both the other two schools. It is also in the Top 20 schools of a total 4 fields. Cornell is the second place and this is mostly because it is in the Top 20 universities of a total 2 fields, which is more than Yale university. Afterall, we can conclude that the universities with good overall (total) performance usually perform also well in fields. 

# Deploy the App
To let other people also view the app without running the python code, I used [Heroku](https://dashboard.heroku.com/apps) to deploy the app so that everyone with the [link](https://streamlit-pre.herokuapp.com/) has the access to it. Before the deployment, there are a few steps need to be prepared. First, we should create a github repository to store the app. Secondly, we need to create a requirements file to tell the server what packages need to be installed. Thirdly, we will need a setup file (already provided by heroku) to make sure the server environment is correct and customized some preference if you have. Lastly, the Profile serves like a make file that connect the server with file so that our app can be successfully launched. After all these steps, we can then deploy the entire repository with Heroku.


