This competition predicted customers' 45-day payments based on their first seven days of game play. The dataset has approximate 2.3 million samples and 107 features.

Data Link: https://www.dropbox.com/sh/7jc7cxojntdd95c/AAByxEvaURHdfAVcaqUji5b-a?dl=0

Competition Link: https://js.dclab.run/v2/cmptDetail.html?id=226
## Background

The player in *Age of Savagery* is given a piece of land and is prompted by the game's storyline to build a complex of buildings somewhere in the land to form their 'inner city' to satisfy the needs of the game's storyline. *Age of Savagery* entails fighting for land, establishing cities and fortresses.

In the flow of such games, 

- In the inner city (your own small city), players must **gather resources**, **amass manpower**, **consume** low-level resources **to produce** high-level resources, **build up** and **defend** the city, as well as **attract/train** a variety of human resources, **upgrade** skills for manpower, **upgrade** skills for buildings, etc. depending on the environment. You must employ tools that can speed up time if you don't want to wait because doing these things, such as building structures and improving your character's talents, requires both resources and time.
- On the periphery (the large map), players can **loot the cities**, **resources**, and **land of other players** and keep expanding their own territory.

The data contains a wide range of features, including accessible resources (which can also be used), levels, props, skills, etc.

## The Goal

This competition predicted customers' 45-day payments based on their first seven days of game play.

### The Data

![data_overview](https://github.com/Jahn1998/Data_Mining/blob/main/Imgs/data_overview.png)


All **resource acquisition/consumption**, **manpower recruitment/loss**/... , **accelerated quantity**, **research**, **building**, **PVP, PVE, online time, amount paid, number of payments**, etc. are all characteristics of the users’ behaviours in the first 7 days of service.

### The Difficulties

1. **Size of the Data Set**
    1. 2,000,000 users
    2. 107 features
    
    **Solution**: a model with simple mathematics, such as logistic regression
    
2. **Label Imbalance**
    
    ![label_imbalance](https://github.com/Jahn1998/Data_Mining/blob/main/Imgs/label_imbalance.png)
    
    **Conclusion:** a significant issue with game conversion exists, and over 99% of customers do not pay.
    **Solution**: splitting the process into two steps, first classification and then regression. 
    
3. **Features Imbalance**
    
    ![features_imbalance](https://github.com/Jahn1998/Data_Mining/blob/main/Imgs/features_imbalance.png)
    
    **Conclusion:** The feature distribution shows that most features are heavily skewed to the left, with 75% of users not using many resources and a lot of players leaving the game.
    **Solution**: caps
    

### Baseline

The official's linear regression model used RMSE to measure the final result of the regression.

The RMSE baseline was `62.00116183660681`

### Procedure

### Label Exploration

---

- If a new player who has not paid in the first seven days has a 99.8% chance of not paying in the following days.
- If a new player who has paid in the first seven days has a 70% chance of not paying in the following days.
- Players who continue to pay after 7 days pay more in the first 7 days
- When the payment amount within 7 days larger than 6 CNY, players are more likely to continue paying than not to stop paying.

![The Probability to Continue to Pay](https://github.com/Jahn1998/Data_Mining/blob/main/Imgs/The%20Probability%20to%20Continue%20to%20Pay.png)

### **Data Analysis**

---

**Time spent online**

- The game time of paying-players is significantly more than that of non-paying users
- Those who spend *above 800 minutes* each week online are of low value (at the payment level)
- Those who spend *less than 15 minutes* each week online are of low value (at the payment level)

![time_spent](https://github.com/Jahn1998/Data_Mining/blob/main/Imgs/time_spent.png)

**Distribution and Skewness**

"low retention rate" is the cause and "Resources Distribution Anomaly" is the result
![data distribution](https://github.com/Jahn1998/Data_Mining/blob/main/Imgs/data%20distribution.png)

**Outliers**

- Definition：
    
    When the value of any instance exceeds [QL-1.5*IQR, QU+1.5*IQR], the value of this instance is considered to be an outlier.
    
    QL: First Quartile，QU：Third Quartile ，IQR：QU-QL
    
- There are only 40,000 users who paid within 7 days, and it was covered by the outliers.
- There are 100,000 abnormal users, of which only 4,000 pay more, that is, many users who are highly similar to paying users do not pay, and it is difficult for the algorithm to judge these users

### **Feature Engineering**

Anomaly identification based on the box plot, with 0 override for players who haven't made a payment within 7 days in the outliers and no treatment for players who have, accentuating the distinction between the characteristics of paying and non-paying players.

### **Model Construction**

1. Stage 1: Classifying
    1. When used to distinguish between paying and non-paying users, logistic regression models have an accuracy rate of`0.9968302114394293` on average.
    2. When threshold  for logistic regression is set to 0.5，Recall `0.8421310404875925`，Precision `1.0`，the main source of RMSE is the amount paid by paying users who aren't included in logistic regression plus the difference between the amount paid by logistic regression and the amount predicted by linear regression；In order to reduce RMSE，it is imperative to improve Recall.
        
        The test set output the probability of each sample, which enabled to choose a suitable threshold based on the probability.
        
    3. Changing the threshold for logistic regression: the 0.02 criterion is selected by setting the threshold candidates to minimize the RMSE.
2. Stage 2: Regression
    1. Choose among Linear Regression Model, Random Forest Regression Model, and Gradient Boosting Regression Model. In the end, Random Forest Regression Model is selected.
    2. Increasing the number of base learners to increase the ability to fit; 
    3. Adjusting the height of the tree to prevent overfitting; 
    4. Adjusting the min_samples_split (if too large, the tree cannot be very deep);
    5. Regulating the maximum number of features  (too small tends to make the model unstable).

### Conclusion

The final RMSE was `53.925094868763`, using the GBR model.

### Future Improvement

- Bin numerical features：facilitate tree model regression
- XGB
- Neural Networks
