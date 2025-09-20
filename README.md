# nlp-sentiment-analysis-online-review

Table of Contents

- [Stakeholder Value Propositions](#stakeholder-value-propositions)
- [Data Description](#data-description)
- [Data Preprocessing and EDA](#data-preprocessing-and-eda)
- [Geospatial Analysis](#geospatial-analysis)
- [Network Analysis](#network-analysis)

## TL;DR
Integrated data using mysql
Leveraged natural language processing (sentiment analysis), network analysis, and geospatial analytics on an online review platform to detect and interpret user behaviors and preferences.

In this project, I used a collection of data related to businesses, reviews, users, and other interactions on the Yelp platform. The dataset includes information from 8 metropolitan areas in the USA and Canada, covering a variety of business categories and user demographics, which provides a valuable resource for understanding the factors that influence customer satisfaction, business success, and platform growth. Thus, the algorithms and models tested and developed from this data and project can be applied to other online platforms and domains.

## Stakeholder Value Propositions
To analyze the performance of online sales campaign and provide recommendations for improving future sales, I performed sentiment, network, and geospatial analysis to examine the data from following three stakeholders' perspectives:
### 1. Users' Perspectives
   1.1 Since account creation, how many reviews do regular users and elite users post on average each month? How often are reviews rated as useful, funny, or cool (count and proportion)?
   1.2 Do reviews with a high number of “useful” votes tend to be positive or negative? What is the length of those reviews with high number of "useful" vote? How about the reviews with high number of “funny” and “cool” votes?
   1.3 What is the relationship between the number of reviews and the number of fans? What is the relationship between the number of fans and whe elite user status?
   1.4 Which cities receive the most user reviews?
#### 2. Businesses/restaurants' Perspectives
   2.1 How many restaurants fall into each star-rating category? What is the relationship between restaurant ratings, the number of reviews, the average review score, and the ratio of positive to negative reviews?
   2.2 Analyze which states have the largest number of restaurants, the highest proportion of five-star establishments, and the highest positive review rates, and examine how these patterns relate to geographic location.
   2.3 What is the relationship between restaurant ratings and days in operation?
#### 3. Platform's Perspectives
   3.1 How many users registered and how many reviews were posted each year and each month? 
   3.2 What are the number of and proportions of elite users?
   3.3 What are the annual retention rates for all users, and specifically for elite users?

## Data Description

The following 6 datasets are integrated to perform further analysis:

1. yelp_business.csv: This file contains information about businesses on the Yelp platform, including their ID, name, neighborhood, address, city, state, postal_code, latitude and longitude, star rating, review count, and categories.
2. yelp_business_hours.csv: This file contains information about the hours of operation of businesses, including the business ID and opening and closing times for each day of the week.
3. yelp_checkin.csv: This file contains businesses' registration information, including the time and date when businesses registered on Yelp.
4. yelp_review.csv: This file contains customers' reviews, including the review ID, user ID, business ID, rating stars, reviews in text, number of useful/funny/cool and the date that reviewed was posted.
5. yelp_tip.csv: This file contains data related to tips, such as text about tips, the date that reviewed was posted, business ID, and user ID.
6. yelp_user.csv: This file contains Yelp users information, including their ID, name, registration time, review count, the number of times their reviews received useful/funny/cool, average stars of their ratings, and other information about their past reviews.

## Data Preprocessing and EDA
### Data Cleaning
### 1. Users' Perspectives
   1.1 Since account creation, how many reviews do regular users and elite users post on average each month? How often are reviews rated as useful, funny, or cool (count and proportion)?
   Firstly, 

   1.2 Do reviews with a high number of “useful” votes tend to be positive or negative? What is the length of those reviews with high number of "useful" vote? How about the reviews with high number of “funny” and “cool” votes?
   1.3 What is the relationship between the number of reviews and the number of fans? What is the relationship between the number of fans and whe elite user status?
   1.4 Which cities receive the most user reviews?
### EDA
Following data exploration is performed using mySQL.


## Geospatial Analysis

<img width="481" height="504" alt="image" src="https://github.com/user-attachments/assets/330464c8-90df-4812-b8de-0ec1e09ec83b" /><img width="639" height="504" alt="image" src="https://github.com/user-attachments/assets/fe5a4768-0455-42a5-a249-7b04b8ab47b4" />



## Sentiment Analysis

## Network Analysis
