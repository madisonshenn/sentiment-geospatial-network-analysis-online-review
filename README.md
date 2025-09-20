# sentiment-geospatial-network-analysis-online-review

# Table of Contents:

Please jump to the specific section of your interest if needed:

- [1 Stakeholder Value Propositions](#1-stakeholder-value-propositions)  
  a. users' perspectives  
  b. businesses' perspectives  
  c. platform's perspectives  
- [2 Data Description](#2-data-description)
- [3 Data Cleaning and Preprocessing with MySQL](#3-data-cleaning-and-preprocessing-with-mysql)
- [4 Exploratory Data Analysis](#4-exploratory-data-analysis)
- [5 Geospatial Analysis On Reviews](#5-geospatial-analysis-on-reviews)  
  a. global level visualization and analysis  
  b. city level visualization and analysis  
  c. tracking high value users (HVUs)  
- [6 Sentiment Analysis](#6-sentiment-analysis)  
  a. general sentiment analysis across platform  
  b. in depth sentiment analysis on the reviews of a popular spot  
- [7 Network and Community Detection](#7-network-and-community-detection)
- [8 Strategic Takeaways](#8-strategic-takeaways)

---------------------------------------------------------------------------------------------
# TL;DR  
I used MySQL for data integration, cleaning, and preprocessing, and Python for exploratory and advanced analytics. The analysis combined natural language processing (sentiment analysis), network analysis, and geospatial methods to uncover user behaviors and preferences on the Yelp platform.  
The dataset covers businesses, reviews, users, and interactions across eight metropolitan areas, spanning diverse categories and demographics. This provides a rich resource for understanding the factors driving customer satisfaction, business success, and platform growth. The algorithms and models developed in this project are broadly transferable and can be applied to other online platforms and domains.  

---------------------------------------------------------------------------------------------
# 1 Stakeholder Value Propositions
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
   
---------------------------------------------------------------------------------------------
# 2 Data Description
The following 6 datasets are integrated to perform further analysis:  
1. yelp_business.csv: This file contains information about businesses on the Yelp platform, including their ID, name, neighborhood, address, city, state, postal_code, latitude and longitude, star rating, review count, and categories.
2. yelp_business_hours.csv: This file contains information about the hours of operation of businesses, including the business ID and opening and closing times for each day of the week.
3. yelp_checkin.csv: This file contains businesses' registration information, including the time and date when businesses registered on Yelp.
4. yelp_review.csv: This file contains customers' reviews, including the review ID, user ID, business ID, rating stars, reviews in text, number of useful/funny/cool and the date that reviewed was posted.
5. yelp_tip.csv: This file contains data related to tips, such as text about tips, the date that reviewed was posted, business ID, and user ID.
6. yelp_user.csv: This file contains Yelp users information, including their ID, name, registration time, review count, the number of times their reviews received useful/funny/cool, average stars of their ratings, and other information about their past reviews.

---------------------------------------------------------------------------------------------
# 3 Data Cleaning and Preprocessing with MySQL
## 3.1 Data Cleaning
Before analysis, I performed systematic data cleaning and preprocessing using MySQL and key steps are summarized here with selected SQL queries used.

### Key Steps
1. Detecting anomalies and invalid values:  
* Checked for missing or out-of-range values in business names, states, and star ratings.
* Verified reviews, tips, and check-ins had valid IDs and timestamps.  
<pre> ```sql 
-- Find businesses with invalid star ratings
SELECT business_id, name, stars
FROM yelp_business
WHERE stars < 0 OR stars > 5;

-- Check for reviews with missing user or business IDs
SELECT *
FROM yelp_review
WHERE user_id IS NULL OR business_id IS NULL;
    ``` </pre>

2. Filtering and restructuring data:  
* Separated businesses into open vs. closed.
* Removed businesses without valid operating hours.
* Dropped user records missing essential information.  
<pre> ```sql 
-- Split businesses into open and closed groups
SELECT *
FROM yelp_business
WHERE is_open = 1;  -- open businesses

SELECT *
FROM yelp_business
WHERE is_open = 0;  -- closed businesses

-- Filter out businesses with no valid hours
DELETE FROM yelp_business_hours
WHERE monday = 'None'
  AND tuesday = 'None'
  AND wednesday = 'None'
  AND thursday = 'None'
  AND friday = 'None'
  AND saturday = 'None'
  AND sunday = 'None';

-- Remove users missing key attributes
DELETE FROM yelp_user
WHERE name IS NULL
   OR review_count IS NULL
   OR yelping_since IS NULL;
    ``` </pre>

3. Validation checks:  
* confirmed no invalid rows remain in reviews, tips, and check-ins.  
* Ensured user metrics stayed within logical ranges.  
<pre> ```sql 
-- Check user metrics are valid
SELECT user_id, average_stars, fans
FROM yelp_user
WHERE average_stars < 0 OR average_stars > 5
   OR fans < 0;

-- Confirm no invalid rows in review table
SELECT COUNT(*)
FROM yelp_review
WHERE stars NOT BETWEEN 1 AND 5;
    ``` </pre>     

---------------------------------------------------------------------------------------------
## 3.2 Data Preprocessing
After cleaning, the dataset was preprocessed to ensure usability and consistency for analysis.
1. Split businesses into open and closed categories using the is_open flag.  

<pre> ```sql 
  -- Select open businesses
SELECT *
FROM yelp_business
WHERE is_open = 1;

-- Select closed businesses
SELECT *
FROM yelp_business
WHERE is_open = 0;
    ``` </pre>  

2. User Table (yelp_user)  
* Validated user activity metrics:  
    average_stars must be between 0 and 5. 
    useful, funny, cool, and fans must be non-negative.  
* Dropped users with invalid metrics.  

<pre> ```sql 
-- Check for invalid user metrics
SELECT user_id, average_stars, fans, useful, funny, cool
FROM yelp_user
WHERE average_stars < 0 OR average_stars > 5
   OR fans < 0
   OR useful < 0
   OR funny < 0
   OR cool < 0;
    ``` </pre>
    
3. Review, Tip, and Check-in Tables  
* Verified that cleaned data contained no invalid references to users or businesses.
* Ensured consistency across foreign key relationships.

<pre> ```sql 
-- Confirm all review records reference valid businesses
SELECT r.review_id
FROM yelp_review r
LEFT JOIN yelp_business b ON r.business_id = b.business_id
WHERE b.business_id IS NULL;

-- Confirm all review records reference valid users
SELECT r.review_id
FROM yelp_review r
LEFT JOIN yelp_user u ON r.user_id = u.user_id
WHERE u.user_id IS NULL;
    ``` </pre>
 
---------------------------------------------------------------------------------------------
# 4 Exploratory Data Analysis

4.1 
We want to understand the relationship between the number of “useful” marks and the highest star ratings.

The following regression plots show the trend of how the highest rating (stars) changes as the number of “useful” marks on reviews increases. The x-axis represents thresholds of “useful” marks, ranging from 100 to 1000 in increments of 20. The y-axis represents the highest rating corresponding to each threshold.
<img width="1610" height="451" alt="image" src="https://github.com/user-attachments/assets/847992ed-51da-4cc2-9abf-2cdf6c476f23" />
<img width="1597" height="451" alt="image" src="https://github.com/user-attachments/assets/98cb36b4-a8d0-4e6d-9ea9-8de2c167fea9" /> 

4.2 
<img width="1027" height="474" alt="image" src="https://github.com/user-attachments/assets/0f273ebd-08c0-4e8f-9aed-41d805cd512d" />


<img width="1337" height="578" alt="image" src="https://github.com/user-attachments/assets/0ff32923-bd8d-4cad-978a-35de6b40be06" />

<img width="3076" height="1763" alt="image" src="https://github.com/user-attachments/assets/0003be40-624e-46c4-89a4-10a2e9bb85f2" />


---------------------------------------------------------------------------------------------
# 5 Geospatial Analysis On Reviews
In this section, I will show how I analyzed the geographic locations of businesses/restaurants. First, I looked at a global view of Yelp businesses. Next, I zoomed in on the two most concentrated regions (North America and Europe), and explored the cities with the highest number of reviews within those two regions respectively.

---------------------------------------------------------------------------------------------
## 5.1 Global Visualization and Analysis
<table>
  <tr>
    <td width="60%">
      Note: I used Matplotlib and Basemap libraries to generate the world map. Then, I generated a globe-style version of the map using orthographic projection, and filled the continents and oceans with the specified color codes. After that, I drew country boundaries and used a scatter plot to mark the latitude and longitude of business locations.
    </td>
    <td width="40%">
      <img src="https://github.com/user-attachments/assets/330464c8-90df-4812-b8de-0ec1e09ec83b" width="100%" />
    </td>
  </tr>
</table>

Zoom In to North America and Europe
<p align="center">
  <img src="https://github.com/user-attachments/assets/fe5a4768-0455-42a5-a249-7b04b8ab47b4" width="45%" />
  &nbsp;&nbsp;
  <img src="https://github.com/user-attachments/assets/30c8d1ec-4f29-44f8-b95f-636775036374" width="30%" />
</p>

---------------------------------------------------------------------------------------------
## 5.2 City level visualiztaion and analysis

I used Matplotlib to create the following four scatter plots. Firstly, I selected businesses located within small geographic bounding boxes around Las Vegas, Pheonix, Stuttgart, and Edinburgh, by filtering latitude and longitude ranges from the dataset rating_data. For each city, I ploted the businesses’ coordinates as scatter points on a black background. 

Now, we can compare the spatial distribution of businesses in two cities in U.S. and two cities in Europe.

An interesting observation is that U.S. cities often have orderly blocks or grid structures, whereas other cities may display a more fluid and irregular design. By visualizing business location data on maps, we can gain deeper insights into the distribution of economic activity and business clusters across different urban areas. This approach reveals spatial patterns and trends in a direct and engaging way, offering valuable perspectives for urban planning, policy-making, and business decision-making. Data visualization allows us to visually explore and understand the complex spatial relationships within urban areas, helping us better grasp how cities develop and change.

<img width="1251" height="624" alt="image" src="https://github.com/user-attachments/assets/898ff8ad-cf25-4b7e-a141-b34badf7daa7" />
<img width="1251" height="624" alt="image" src="https://github.com/user-attachments/assets/6018ef00-c426-42de-9496-748c246285fc" />

#### How people rate different businesses in Las Vegas:
Below is an interactive animation created using the impressive Folium package to generate striking Leaflet map visualizations. 

In this animation, businesses are highlighted according to their star ratings. The goal is to see whether certain hotspots or concentrated areas have particularly great restaurants. 

It turns out that both well-rated and poorly-rated businesses are fairly evenly distributed across the city.
Following are the screenshots showing the time-dynamic of reviews (rating stars) in Las Vegas:
<p align="center">
   <img src="https://github.com/user-attachments/assets/023fe3df-ec26-470d-bba7-8e1979628484" width="32%" />
   <img src="https://github.com/user-attachments/assets/1b107b2c-20cf-4ec5-bcce-79735be753f0" width="32%" />
   <img src="https://github.com/user-attachments/assets/c0e2f187-f07d-4d0c-81f3-4c90fb53c06c" width="32%" />
</p>

<p align="center">
   <img src="https://github.com/user-attachments/assets/d1663ea3-1868-4aa5-9e30-b99a1fb603b6" width="32%" />
   <img src="https://github.com/user-attachments/assets/6a473e3a-f909-441c-a8ae-b3fd41f38307" width="32%" />
   <img src="https://github.com/user-attachments/assets/008bc5d2-9db5-4254-b230-a1542524e986" width="32%" />
</p>

---------------------------------------------------------------------------------------------
## 5.3 Tracking High-Value Users (HVUs)
It's useful to focus on the top-ranking users from the review dataset. Reviews from HVUs highlight popular restaurants and emerging trends in specific regions. Platform like Yelp can monetize these insights by guiding business advertisers to understand where and how to invest marketing spend. Besides, HVUs often write detailed, useful, and high-signal reviews (frequently marked as useful/funny/cool). Thus, by monitoring their activity, Yelp can maintain high-quality content, which directly drives consumer trust and platform stickiness. From tracking HVUs activities, platform can also identify high-value business categories and regions that generate the most engagement.

By aggregation function, I found the top users with review['user_id'] == 'CxDOIDnH8gp9KXzpBHJYXw'. To track this high value user, I created a map showing all the locations he/she visited in chronological order, and then generated a heatmap animation from it.

Below is screenshots showing a few timepoints of the locations of the restaurants this top users posted reviews about:
<p align="center">
   <img src="https://github.com/user-attachments/assets/9641b602-3bd9-43cd-b66a-816f5bdb3619" width="32%" />
   <img src="https://github.com/user-attachments/assets/180a2ca9-0644-4489-a66e-a7d078aeea8c" width="32%" />
   <img src="https://github.com/user-attachments/assets/3f79ff4c-726e-435e-b5d6-de4eb345e1d0" width="32%" />
</p>

---------------------------------------------------------------------------------------------
# 6 Sentiment Analysis
This section includes a general analysis across the platform and a more in depth sentiment analysis on reviews of a specific restaurant.
Quick Links to jump to the second part for more in depth analysis on the reviews of a popular spot: - [Sentiment Analysis On Gen Korean BBQ House Reviews](#sentiment-analysis-on-gen-korean-bbq-house-reviews)

## 6.1 Sentiment Analysis Platform Wide
How I did the sentiment analysis in this section: I created a TextBlob object blob to represent the text to be processed. Then, by calling methods of the TextBlob object such as words, tags, noun_phrases, and sentiment, it performs text processing and sentiment analysis. For example, if the input text is 'I love this restaurant! It’s amazing.'. The TextBlob object blob executes tokenization, part-of-speech tagging, noun phrase extraction, and sentiment analysis, and outputs the corresponding results.

Note: Since the data contains more than 1,000,000 users so I sampled 10000 users to perform the following analysis and visualization.

By performing sentiment polarity scoring on the reviews of elite users and regular users separately, we can see the overall polarity distribution. Elite users are more concentrated around 0.25, leaning toward the positive side.
<img width="2578" height="1638" alt="image" src="https://github.com/user-attachments/assets/394c0bfe-d214-408b-85fa-91319d730160" />

Next, I looked at the keyword distribution for reviews with polarity greater than 0.25 or less than –0.25, and construct a word cloud.
<img width="4709" height="1311" alt="image" src="https://github.com/user-attachments/assets/a1de2c78-e37a-4bbd-8ce7-988f3d623e84" />

Cap max reviews to 30 for better visuals. ~80% of the users write only about 2 reviews.
<img width="3010" height="1500" alt="image" src="https://github.com/user-attachments/assets/a9c4078a-fff4-4b18-8d6d-9a6350103c02" />

---------------------------------------------------------------------------------------------
## 6.2 Sentiment Analysis On Gen Korean BBQ House Reviews
Why do we want to apply more in-depth sentiment analysis using tools such as VADER and AFINN, specifically on popular restaurants?
* A 5-star rating doesn’t explain why a restaurant is popular or unpopular, but sentiment analysis of review text captures nuances of customer experience (e.g., “great food but slow service”). This lets the platform identify drivers of positive vs. negative reviews.
* VADER and AFINN can quantify sentiment polarity and intensity across thousands of reviews. Platforms can provide restaurants with analytics dashboards on, for instance, what aspects are most praised (food quality, atmosphere) and what aspects need improvement (service speed, pricing). This strengthens the platform’s value proposition to business owners, encouraging them to engage (or advertise).
* Deeper sentiment signals can be used to enhance search and ranking algorithms (e.g., show “family-friendly” or “great service” restaurants). Thus, enhancing user experience.
* We can also use sentiment analysis to track shifts over time. This is valuable in situation like flagging early warning signals is an ever popular restaurant’s sentiment score starts dropping due to service or quality declining.

### 6.2.1 EDA and Top Common Words for this restaurants

Distributions of review types:
<img width="2531" height="1652" alt="image" src="https://github.com/user-attachments/assets/8ef8175d-87ad-43f5-97ba-0bab38bf415d" />
<img width="5370" height="1770" alt="image" src="https://github.com/user-attachments/assets/df0a2e2f-2b76-4c40-b3f7-41742bfde308" />

Word cloud & top ten most common words in the reviews of this restaurant:
<p align="center">
  <img src="https://github.com/user-attachments/assets/c2ea0fdb-9f21-404a-a12f-b3ac754a9d9d"  width="45%" />
  &nbsp;
  <img src="https://github.com/user-attachments/assets/73a5527d-087e-41c3-afea-4a9905d6f756"  width="45%" />
</p>

### 6.2.2 Sentiment analysis: positive and not so postive words
In this section, I primarily used the VADER (Valence Aware Dictionary and Sentiment Reasoner) sentiment analysis tool from Python’s NLTK library which is specifically designed for sentiment analysis of social media text.

Implementation:
1. Data Preparation: Filter out the reviews for this specific restaurant.
2. Tokenization: Split reviews into individual words.
3. Stopword Removal: Remove stopwords that carry no real meaning.
4. Sentiment Analysis: Use VADER to assign sentiment scores to each word.
5. Lexicon Classification: Classify words into positive and non-positive categories based on their sentiment scores.

#### Word Clouds
Left: word clouds for positive words  
Right: word clouds for non-positive words
<p align="center">
  <img src="https://github.com/user-attachments/assets/28552f2f-1e6f-4a04-945a-1bb350ebb219"
 width="45%" />
  &nbsp;
  <img src="https://github.com/user-attachments/assets/13f1901a-f41f-4255-9b87-6708af4ed0e0"  width="45%" />
</p>

#### Visualizing the top 20 positive and negative sentimental words based on the scores
<img src="https://github.com/user-attachments/assets/26ec4303-9ca2-4a7d-a7c7-8d4ecada4512" width="80%" />

### 6.2.3 Calculate sentiment for the reviews using AFINN
Implementation:
1. Load the AFINN Sentiment Lexicon: Load the AFINN dictionary; this lexicon contains words and their corresponding sentiment scores.
2. Tokenization and Score Calculation: Tokenize each review, then use the dictionary to assign scores for each word, summing them to obtain the overall sentiment score of the review.
3. Ranking and Display: Sort all reviews by sentiment score and display the top six reviews with their sentiment scores.

Visualization showing bigrams that appeared more that 30 times
  <img src="https://github.com/user-attachments/assets/7b5dd17b-763c-4091-9aea-01fcb4811b50" width="100%" />

Visualizations showing bigrams that appear more than 30 times and contain 'pork', 'bbq', or 'service'.
<p align="center">
  <img src="https://github.com/user-attachments/assets/1216aced-eb86-4faf-b02a-37e336354543" width="32%" />
  <img src="https://github.com/user-attachments/assets/a5bfdb5e-9696-44be-a004-9a293711f248" width="32%" />
  <img src="https://github.com/user-attachments/assets/f06762d5-cbfe-49b2-a44b-2eb0ebd1ea48" width="32%" />
</p>

---------------------------------------------------------------------------------------------
# 7 Network and Community Detection

## 7.1 Building the User Friendship Network
To examine the social dimension of Yelp, we constructed a friendship graph using the friends field in yelp_user.csv. This allows us to investigate clustering patterns, overall connectivity, and the presence of potential influencers.  

Each user_id was paired with its corresponding friend list (comma-separated) to generate edge relationships. Using NetworkX, we created an undirected graph. A global sample of roughly 6,000 users was drawn, and the structure was visualized with the spring layout algorithm.  

<img width="414" height="216" alt="Screenshot 2025-09-20 015237" src="https://github.com/user-attachments/assets/8a1a5341-d1a4-4840-a057-5f5de7a9bc39" />

Key Findings:  
* The global network is highly sparse: many users are isolated or connected to only one other person.
* A small number of central nodes act as network hubs, likely corresponding to Elite or particularly social users.
* The Yelp friendship graph is loosely connected overall, implying that user activity is driven more by review behavior than by friendships.  

Implication  
While Yelp provides a friend system, it does not appear to be the primary driver of engagement or discovery. This insight is relevant when considering social-based recommendation methods—collaborative filtering may be limited in effectiveness without stronger graph density. To uncover meaningful community structures, a regional zoom-in is explored in the next section.

## 7.2 Stuttgart Subgraph and Community Detection

To capture more tangible community behavior, we focused on the Stuttgart, Germany region. We filtered users who had reviewed Stuttgart businesses and then constructed their friend network.  

Using NetworkX, we built the Stuttgart-specific subgraph and applied degree centrality to highlight influential users. Then we ran the Louvain algorithm to detect communities.  

Visualizing the results with multiple layouts, including spring, circular, and Kamada–Kawai:
<img width="454" height="443" alt="Screenshot 2025-09-20 015249" src="https://github.com/user-attachments/assets/476c9e8f-0d3c-4013-ae93-8428ed96bbe9" />

Key Findings:   
* Unlike the global graph, the Stuttgart network is denser and exhibits clear community clusters.
* The Louvain method identified five primary communities, which likely represent real social circles or groups with common interests.
* Influential users, revealed through centrality measures, could serve as amplifiers for local promotions or targeted content.  

Implication:  
Regional analysis reveals micro-community structures that are not visible at the global scale. For Yelp, this indicates that a localized approach to social features could be more effective—for instance, boosting elite content within communities or applying community-aware ranking in recommendations and marketing campaigns.

---------------------------------------------------------------------------------------------
# 8 Insights by Stakeholder Perspective
### 1. User Perspective
For users, a key motivation is often how to efficiently earn the Elite User badge, which brings access to special promotions and increased visibility on the platform. Analysis shows that the most important factor is the volume of reviews and photos posted, particularly reviews. Long-form reviews with higher information content are more impactful than short or stylistically polished ones. Reviews that highlight overlooked issues at otherwise highly-rated restaurants are especially likely to attract attention. Geographically, users located in states such as Nevada and Arizona have a higher likelihood of being awarded Elite status, due to the density of high-traffic businesses in these regions.  

For general users who are less concerned with status, the primary goal remains finding restaurants that match their preferences. Overall, restaurant star ratings and user review scores are broadly aligned, providing a reliable guide to quality. However, five-star ratings may sometimes be inflated, making it important to cross-check against detailed user feedback.

### 2. Business Perspective

From the perspective of restaurant owners, increasing visibility and customer traffic can be achieved by opening locations in states with larger markets such as Nevada, Arizona, North Carolina, Ohio, and Pennsylvania. For those aiming to achieve or maintain five-star status, Nevada and Arizona are particularly promising markets. In addition to fundamentals like foot traffic and operating days, maintaining a low rate of negative reviews is critical.  

Restaurants less concerned about location can still improve ratings by maximizing weekly operating days and ensuring consistently high customer satisfaction. Simply put, operational consistency and strong review management are more influential than geography when aiming for higher average ratings.

### 3. Platform Perspective

From the platform’s standpoint, existing user engagement is strong, but new user registrations have declined year over year. This suggests both market saturation and the need for new growth strategies. Potential measures include seasonal promotions (e.g., partnering with restaurants during summer, when dining out activity peaks) and offering targeted discounts to attract new users. Expanding into new states or international markets also presents a viable path for sustaining growth.  

For existing users—especially active non-Elite users—the platform could introduce cyclical engagement campaigns that reward activity milestones (e.g., posting reviews, check-ins, or photos). “Come-back” benefits could re-engage inactive users, while further enhancing the perks of Elite membership would incentivize broader participation and raise overall community quality.

# 9 Strategic Takeaways

