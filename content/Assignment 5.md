Title: Use SWAPI to query Star Wars Universe characters
Category: Database
Date: 2020-10-20 10:01
Modified: 2020-10-23 11:30
Tags: Database, API, requests
Slug: SWAPI
Authors: Presnie Lu
Summary: Download all people from Start Wars Universe using SWAPI and find the oldest character

**This blog is used to apply the requests library, download all the people in the Star Wars universe using the [Star Wars API](https://swapi.dev/documentation). We will also show the name of the oldest person (or robot or alien) and list the titles of all the films they appeared in.**

# Explore SWAPI

One important tip before opening up python and using the requests library is to read through the documentation of APIs. In most cases, there are certain patterns of the urls which we pulled data from and it is very important to understand how these urls are structured. For SWAPI, we use this [documentation](https://swapi.dev/documentation) to figure out how to pull data for all star war universe characters. We notice that the basic url( which is the root URL for all of the API) is **https://swapi.dev/api/** and we will append our customized url address base on this from now on. By reading through the document, we will see a section which tells you how to pull data for all people:  
![People documentation](/img/people_doc.png "Documentation of Start War People")    
We see that we can use https://swapi.dev/api/people/ to get all the people resources. We can have a simple try by entering this url address from our browser. A screenshot of result looks like following:  
![People list](/img/people_list.png "Screenshot of people list")   
From the response **HTTP 200 OK**, we see this is a successful request. We also notice there is a count listed above, which tells us there are 82 records available for these characters. However, when you scroll down, you will find out that the listed records from the results are definitely much fewer than this number. Therefore, we can instead use a for loop in python to go through all records of each of the character. To illustrate what individual record looks like from the url, we simply enter this address into the browser: https://swapi.dev/api/people/1/. A screenshot is presented below:  
![People Instance](/img/people_instance.png "Screenshot of people instance")   
We see the structure of the record is very organized and we can easily turn this dictionary into a data frame in python. 


# Download Data Using API

The next step is to use requests library from python to pull these data. As I mentioned before, we need to list all combination of base url with 82 people to get a full list of star war universe characters. Note that later from our exploration of dataset, we found out that people with ID 17 is actually unavailable. Therefore, in order to keep all 82 records, we will need to pull data from https://swapi.dev/api/people/1/, https://swapi.dev/api/people/2/, .... ,https://swapi.dev/api/people/83/. Therefore, a for loop is used and we append each record as a form of the list every time we finish a request.  

    :::python
    #pull all data
    people = []
    for i in range(1, 84):
        p = requests.get('https://swapi.dev/api/people/{}/'.format(i)).json()
        people.append(p)

Then, to make the data more presentable, we convert the list into a panda data frame. 
    
    :::python
    #Convert list to dataframe
    people_df = pd.DataFrame(people)

Then I explore whether there is empty records exists. That is if we have any records with name as NA. The code is shown below as well as a screenshot of our result. It confirms our assumption that record 17 is indeed empty:

    :::python
    #drop empty record
    people_df[people_df["name"].isna()]
    people_df = people_df.drop(16,axis = 0)

![Empty Record](/img/nas.png "Empty Record") 
Now, we will have 82 records, each row will be the relevant information of each character. Each column will be the features of characters, including: name, height, mass, hair_color, skin_color, eye_color, birth_year, gender, homeworld, films, species, vehicles, starships, created, edited, url, detail. I won't go into details about what each of these features means. For now, we should care about **name, birth_year and films** the most. The following table shows the record of first five people from the database with features that we care about.  
![Downloaded Data](/img/people_df.png "Downloaded Data")  

# Find the Oldest Character

Now we got a dataframe with all people's information in it. In order to find out who is the oldest character, we can use sort to order these characters from the oldest to youngest. The feature that we should look at is **birth_year**. However, we notice that the birth_year is not documented as what we usually did. All birth years are combinations of certain number and string "BBY" or "ABY". Therefore, it is necessary for us to go back to the [documentation](https://swapi.dev/documentation) and find out what this feature really means. Under **People** session, it lists a bunch of attributes as well as their meanings. These attributes are the same as features as I mentioned in last part:  
![Attributes](/img/attributes.png "Attributes")  
We see that the string in birth_year means Before the Battle of Yavin or After the Battle of Yavin. Because we need to find out the oldest character, I decide to first filter out people that were born before the Battle, which is equivalent to those people with "BBY" in birth_year. 

    :::python
    #filter out people born before the battle
    BBY = people_df[people_df["birth_year"].str.contains("BBY")]
    
Then, after applying this filter, the data with people born before the battle looks like this (the first five people with selected feature):   
![People Born Before Battle](/img/BBY.png "People Born Before Battle")  
The next step would be finding out what is the oldest people among these. The basic idea is to find the largest number located before the string. For instance, 9BBY is smaller than 100BBY before the later one was born 100 years before the battle. Therefore, I used split to extract only the numeric part from birth_year. To be more specific, I first split on "B" so that the string will be divided into number, empty string and string Y (100BBY will be divided into 100, "", "Y"). We then apply pd.series to make each divided part into a column. Note that because the cell was original string, in order to sort in a numeric convention, we need to convert the string to float. Since we only need the numeric part, I added this "year" column into the original data and sort in descending order base this column. 

    :::python
    #sort by numeric term
    year = BBY["birth_year"].str.split("B").apply(pd.Series)[0]
    BBY["year"] = year.astype("float")
    BBY_sorted = BBY.sort_values("year", ascending = False)

The sorted table with five oldest characters look like this:  
![People Sorted with Age](/img/BBY_sort.png "People Sorted with Age")  
From this we can see the oldest character is Yoda.
![Yoda](/img/yoda.png "Yoda")  

# List the titles of all the films Yoda appeared in

The remaining step is quite easy. Now we know that Yoda is the oldest character in this database, we can look into the attribute **films**. We notice that this attribute contains a list of different urls link to the film database from SWAPI, we can then apply a similar step of using requests and for loop to extract the relevant information of the films.  

    :::python
    #list out films 
    yoda = BBY_sorted.iloc[0,:]
    urls = yoda["films"]
    films = [requests.get(url).json() for url in urls]
    films_df = pd.DataFrame(films)
    
Finally, the following table shows a list of film titles and episode_id that Yoda appears in:  
![Film title and episode_id](/img/film2.png "film")  

# Addition: Unnest the data frame
During the process of exploring the dataset, we realize that there are certain columns in the dataframe have nested items. Therefore, we will use "explode" in python to unnest these items. This way, we will have a few rows for each person.
    
    :::python
    # additional step for unnested df
    people_df = (
        people_df.
        explode("films").
        explode("species").
        explode("vehicles").
        explode("starships")
    )

The resulting table looks like this:  
![Unnested Table](/img/unnest.png "Unnested Table")  



