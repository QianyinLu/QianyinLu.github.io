Title: How to Create a Static Site on Github
Category: Pelican
Date: 2020-08-24 10:01
Modified: 2020-08-24 12:30
Tags: Static, Pelican
Slug: static-page
Authors: Presnie Lu
Summary: Static Site With Pelican


**This first blog will be used to describe how to generate a static page using pelican and upload it to github.**

## Build a New Repository
The first step is to create a new repository in github. I simply renamed the new repository using my own user name. I followed the instructions provided by [GitHub](https://pages.github.com/) and initialize the repository with a README file and a MIT license. After that, I cloned this repository to the local machine and locate the system to work under this cloned folder. I also created a new branch called source to store and change my source code so that the master branch only works for serving files. 

## Create a Static Site with Pelican
The next step is to use the static site generater [Pelican](https://blog.getpelican.com/) to create my own personal website. I listed several steps used to achieve such goal:  

1. Install the pelican and ghp-import packages: the ghp-import package is used to publish the website to github.  
```pip install pelican ghp-import```  
2. Run pelican-quickstart to create the site: Most questions can follow the default setting or our own preferences. Note: The following two questions are answered with Y to enable the automation process.  

    * Do you want to generate a Fabfile/Makefile to automate generation and publishing? Y  
    * Do you want to upload your website using GitHub Pages? Y  
    
3. Create blogs/Articles under the content folder with text editor.(I used Markdown)  
4. Generate the static files and Start the server using command:   
```make html && make serve ```  
  
5. Preview the website by typing localhost:8000 in web browser's address bar.  

## Update the Website to Github
The left step is to send the entire code to Github. We add, commit and push everying to the github. We will also generate the final output of the pages under master branch on github using the make function. 
  
``` make github ```  

## Optional: Change the Theme
There are many [themes](http://www.pelicanthemes.com/) available for pelican users. Once we decided which one to use, we can then navigate to the [Pelican Themes GitHub repository](https://github.com/getpelican/pelican-themes). Following the instruction from the README file, I first clone the repository to local machine and then add the following line to pelicanconf file from my website's folder. I used the [attila](https://github.com/arulrajnet/attila/tree/02dcad911ba1eb2d797a79ec008a810d89a2fde1) theme and many customized setting can be done similarly by modifying folders or the pelicanconf file.  
  
```THEME = Location of the Selected Theme Folder```


