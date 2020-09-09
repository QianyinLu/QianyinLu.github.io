Title: Solution Explanations to the Euler Project
Category: Algorithms
Date: 2020-08-25 10:01
Modified: 2020-09-09 11:30
Tags: Euler Project, data structure
Slug: euler-project-problem
Authors: Presnie Lu
Summary: Solve three problems of the Euler Project

**This blog is used to illustrate how to solve three problems from Project Euler. Eeach problem will have the solution code attached as well as an explanation to that step by step.**

## Problem 1 - Largest product in a series
**from Project Euler's [Problem 8](https://projecteuler.net/problem=8).**   
  
To solve this problem, I first copied the string that includes the 1000-digit number to python. The major modification I did for the string was to remove the line switch so that the string will only include integers. Then, I used list comprehension to convert all numbers from string to integers and splitted each integer by ",". The next step is to regroup these integers into the required numbers of adjacent digits, in this case, each sublist in combo includes 13 adjacent digits.  After that, I generated a list called products to include products of each combination of 13 adjacent digits. Then, it is easy to find the largest number and its index in the product list and find what combination leads to such outcome by using the same index. 

#### code:  

    :::python
    def consecutive_num(raw, n):
        number=raw.replace("\n","")
        integer_set = [int(x) for x in number]
        combo = [integer_set[i:i+n] for i in range(len(integer_set))]
        products = [np.prod(num) for num in combo]
        idx = np.argmax(products)
        return (combo[idx],products[idx])
    
    consecutive_num(raw,13)

Using such function, the largest product is 23514624000 and the 13 adjacent digits should be 5576689664895

## Problem 2

## Problem 3