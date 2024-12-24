# Upliance.-AI

### Objective

This project aims to analyze and visualize user behaviors, cooking sessions, and order patterns. The analysis identifies relationships between cooking session participation and order trends, explores demographic factors influencing user activity, and highlights popular dishes and meal types. The insights provide actionable recommendations for improving business operations and customer satisfaction.

### Datasets:

####  Order Details Table:
- Columns: Order ID, User ID, Order Date, Meal Type, Dish Name, Order Status, Amount (USD), Time of Day, Rating, Session ID
- Provides information about user orders, meal preferences, and order status.

#### Cooking Sessions Table:

- Columns: Session ID, User ID, Dish Name, Meal Type, Session Start, Session End, Duration (mins), Session Rating
- Details the cooking sessions, including duration, user participation, and session ratings.

#### User Details Table:
- Columns: User ID, User Name, Age, Location, Registration Date, Phone, Email, Favorite Meal, Total Orders
Contains demographic information about users

### Data Cleaning and Preparation

#### 1. Handling Missing Values

- Rating: Replaced missing (N/A) values in the "Rating" column with Blanks for calculations and analysis.
- Session Rating: Checked for consistency and ensured missing ratings were treated as neutral or zero.
  
#### 2. Data Type Adjustments
- Converted dates to the appropriate datetime format for accurate sorting and time-based analysis.
- Changed numeric fields (e.g., Amount, Session Rating) to numeric data types for aggregation.

#### 3. Duplicate Removal
- Checked for duplicate entries in all datasets and removed redundant rows to ensure data consistency.

#### 4. Data Validation
- Cross-verified the relationships between tables using primary and foreign keys:
- Matched User ID across all three tables.
- Matched Session ID in the Order Details and Cooking Sessions tables.
  
### New Measures and Calculated Columns

#### Calculated Columns

```
Session-Order Match = 
IF(
    ISBLANK(OrderDetails[Session ID]),
    "False",
    "True"
)
```

```
Session-Order Time Gap = 
DATEDIFF(
    RELATED(CookingSessions[Session End]), 
    OrderDetails[Order Date], 
    MINUTE
)
```

```
Dish Popularity Rank = 
RANKX(
    ALL(OrderDetails[Dish Name]), 
    CALCULATE(COUNT(OrderDetails[Order ID])), 
    , 
    DESC
)
```

```
Age_Group = 
SWITCH(
    TRUE(),
    UserDetails[Age] >= 18 && UserDetails[Age] <= 25, "18-25",
    UserDetails[Age] >= 26 && UserDetails[Age] <= 35, "26-35",
    UserDetails[Age] >= 36 && UserDetails[Age] <= 45, "36-45",
    UserDetails[Age] > 45, "46+",
    "Unknown"
)
```

#### Measures:

```
Conversion Rate = 
DIVIDE(
    COUNTROWS(FILTER('OrderDetails', 'OrderDetails'[Order Status] = "Completed")),
    COUNTROWS('CookingSessions'),
    0
) * 100
```

Average Session Rating for Completed Orders:

```
Avg Session Rating = 
AVERAGEX(
    FILTER('OrderDetails', 'OrderDetails'[Order Status] = "Completed"), 
    RELATED('CookingSessions'[Session Rating])
)
```

Dish Frequency (Sessions):

```
Dish Frequency (Sessions) = 
COUNTROWS(CookingSessions)
```

Dish Frequency (Orders):

```
Dish Frequency (Orders) = 
COUNTROWS(OrderDetails)
```

Revenue per Dish:

```
Revenue per Dish = 
SUMX(
    VALUES(OrderDetails[Dish Name]), 
    SUM(OrderDetails[Amount (USD)])
)
```

Order Count by Demographic Group (e.g., by Age Group):

```
Orders by Age_Group = 
COUNTROWS(
    FILTER(
        OrderDetails, 
        RELATED(UserDetails[Age_Group])  = "26-35" -- Replace with specific Age Group
    )
)
```

Lifetime Value (LTV):

```
LTV = 
SUMX(
    RELATEDTABLE(OrderDetails), 
    OrderDetails[Amount (USD)]
)
```

Average Session Rating by Demographic:

```
Avg Session Rating by Demographic = 
AVERAGEX(
    RELATEDTABLE(CookingSessions), 
    CookingSessions[Session Rating]
)
```

Revenue per Meal Type:

```
Revenue per Meal Type = 
SUMX(
    VALUES(OrderDetails[Meal Type]), 
    SUM(OrderDetails[Amount (USD)])
)
```

Meal Type Distribution:

```
Meal Type Frequency = 
COUNTROWS(OrderDetails)
```
Total Orders

```
Total Orders = COUNT(OrderDetails[Order ID])
```

Total Revenue

```
Total Revenue = Sum(OrderDetails[Amount (USD)])
```

### Visualizations

#### 1. Overview Dashboard

#### Visuals:

Key Metrics (Card Visuals):
Conversion Rate
Total Revenue (SUM of OrderDetails[Amount (USD)])
Total Orders (COUNT of OrderDetails[Order ID])
Average Session Rating

#### Purpose:
To provide a quick summary of the business performance.

#### 2. Popular Dishes and Meal Types

#### Visuals:
Dish Popularity (Bar Chart):

Axis: Dish Name
Values: Count of Dish Name (Frequency in Orders)
Revenue by Dish (Column Chart):

Axis: Dish Name
Values: Revenue per Dish
Meal Type Distribution (Pie or Donut Chart):

Values: Meal Type Frequency
Legend: Meal Type

#### Purpose:
Identify which dishes and meal types are most popular and generate the highest revenue.

#### 3. Demographics

#### Visuals:

Orders by Age Group (Stacked Column Chart):

Axis: Age Group
Values: Order Count by Demographic Group
Legend: Meal Type (Optional)
Revenue by Location (Map Visual):

Location: UserDetails[Location]
Size: Total Revenue

#### Purpose:
Understand how demographics influence orders and revenue.

#### 4. Cooking Sessions

#### Visuals:
Session Ratings by Dish (Scatter Chart):

X-Axis: Dish Name
Y-Axis: Average Session Rating
Size: Session Frequency
Average Session Duration by Meal Type (Bar Chart):

Axis: Meal Type
Values: Average(Session Duration)

#### Purpose:

Analyze cooking session performance and identify trends in session ratings and durations.

#### 5. Conversion Analysis

#### Visuals:
Conversion Rate by Meal Type (Clustered Bar Chart):

Axis: Meal Type
Values: Conversion Rate
Session-Order Time Gap (Line Chart):

X-Axis: Order Date
Values: Average(Session-Order Time Gap)

#### Purpose:
Evaluate how effectively cooking sessions lead to orders and assess time-related factors.

#### 6. Revenue Insights

#### Visuals:
Revenue by Age Group (Tree Map):

Group: Age Group
Values: Revenue
Daily Revenue Trend (Line Chart):

X-Axis: Order Date
Values: Total Revenue

#### Purpose:
Track revenue performance and identify high-performing customer groups.

#### 7. Drill-Down Analysis

#### Visuals:
Dish Performance by Time of Day (Matrix or Table):

Rows: Dish Name
Columns: Time of Day
Values: Count of Orders, Average Rating
Detailed User Behavior (Table Visual):

Columns: User Name, Location, Total Orders, Favorite Meal, Total Revenue

#### Purpose:
Provide granular details about dish performance and individual user behavior.

#### 8. Slicers:

- By Age Group
- By Location
- By Meal Type

#### 9. User Engagement Analysis

#### Visual: Clustered Bar and Line Chart (Combo Chart)
X-Axis: User Name
Line Values: Average Session Rating
Bar Values: Total Orders

#### Purpose:

- To compare user engagement in terms of the number of orders placed against their average cooking session rating.
- Identify highly engaged users with high ratings or potential users to target for improved engagement.

### Findings

#### 1. User Engagement
- Users with higher average session ratings tend to place more orders:
- Alice Johnson, Bob Smith, and Charlie Lee placed 3 orders each with average session ratings above 4.5.
- Users like Grace King and Frank Green, despite high session ratings (5 and 4.8), placed fewer orders.
- Average session rating strongly correlates with user activity, but external factors (e.g., user availability) may impact order frequency.

#### 2. Revenue by Location
- New York generated the highest revenue ($35), followed closely by Chicago ($32) and Los Angeles ($31).
- Smaller markets like Miami ($11) and Austin ($13) contributed significantly less revenue, indicating potential growth opportunities in these areas.

#### 3. Session-Order Time Gap
- Negative values indicate shorter gaps between cooking sessions and orders, highlighting efficiency:
- December 8th saw the highest time gap reduction (-2040), suggesting effective coordination.

#### 4. Session Rating by Dish
- Grilled Chicken and Spaghetti had the highest average session ratings (4.77 and 4.63 respectively) and were prepared in multiple sessions.
- Breakfast dishes (e.g., Oatmeal) had lower session ratings (4.1) but were limited in frequency.

#### 5. Revenue by Dish
- Spaghetti and Grilled Chicken dominate revenue generation with $55.5 and $51, respectively.
- Lunch options like Caesar Salad ($28) and Veggie Burger ($22) had moderate revenue, indicating potential for targeted lunch promotions.

#### 6. Revenue and Orders by Age Group
- The 26-35 age group contributed the highest revenue ($121), followed by 36-45 ($46) and 18-25 ($13).
- Dinner is the preferred meal type across all age groups, with 5 orders from the 26-35 age bracket, highlighting it as the primary target audience.

#### 7. Meal Type Insights
- Dinner is the most frequent meal type with 8 orders, followed by Lunch (5) and Breakfast (3).
- Conversion rates:
Breakfast: 100% (all breakfast sessions converted to orders).
Dinner: 87.5%.
Lunch: 80%.

#### 8. Average Cooking Session Duration
- Dinner has the longest average duration (38.75 minutes), indicating its complexity and popularity.
- Lunch (21 minutes) and Breakfast (23.33 minutes) are relatively faster to prepare.

#### 9. User Behavior
- Users are consistent with their favorite meals:
- Alice Johnson (Dinner) generated $35.
- Charlie Lee (Breakfast) contributed $32, indicating balanced meal-type preferences across top users.

#### 10. Dish Popularity
- Grilled Chicken and Spaghetti are tied for the top spot with 4 orders each.
- Caesar Salad is the most popular lunch dish with 3 orders.
- Breakfast dishes (e.g., Pancakes and Oatmeal) are less frequently ordered but highly rated.

#### 11. Daily Revenue Trend
- Revenue peaks on December 1st and 8th with $25 each day.
- A consistent revenue pattern shows growth opportunities in scaling dinner services.

#### 12. Dish Performance by Time of Day
- Grilled Chicken and Spaghetti dominate dinner orders at night with high ratings (4.77 and 4.63).
- Caesar Salad leads the lunch segment with a rating of 4.37.
- Breakfast dishes (Oatmeal and Pancakes) perform well in the morning but have limited orders.


### Key Business Insights

- Focus on Dinner and Popular Dishes: Invest in improving dinner offerings, specifically Grilled Chicken and Spaghetti, to capitalize on their high ratings and revenue.
- Target the 26-35 Age Group: This segment contributes the most revenue and orders, making them ideal for loyalty programs and personalized promotions.
- Expand in Low-Revenue Locations: Cities like Miami and Austin present opportunities for growth through targeted marketing campaigns.
- Improve Lunch and Breakfast Offerings: Boost these segments with promotions and faster preparation options to increase their contribution.
- Enhance Cooking Session Experiences: Optimize session duration for high-demand dishes to improve satisfaction and engagement further.
- Seasonal Promotions: Leverage the December peak in revenue to introduce seasonal campaigns, driving additional orders.
