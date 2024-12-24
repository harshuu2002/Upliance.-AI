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
