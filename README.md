# Coffee-Rewards

Ongoing Project!!



## Dataset Exploration

### Table: Customers 

This table serves as a Dimension table. 

Number of Rows: 17.000

Number of Columns: 5

####  Data Completness:

Missing Values Found In: Gender, Income 


#### Data Credibility

- Reliable:  The current dataset is accurate and complete; however, bias issues cannot be determined. The **age** distribution ranges from **18 to 118**. Notably, **all rows with an age of 118 have null values in the 'gender' and 'income' columns**. Additionally, there are 253 rows corresponding to customers aged 90-101, with 17 of them between 100-101. Unlike the 118-year-old entries, **the 90-101 age group has no missing values in other columns**, suggesting they may not be typos. However, **it is worth noting that customers over 100 purchasing coffee might be unrealistic and could indicate potential data quality concerns.**

- Original: The data was downloaded from Maven Analytics but was originally sourced from Kaggle, via Udacity.

- Comprehensive: Contains all essential information needed to answer key questions.

- Current: Not up to date — there is no information regarding the date of data collection. However, this dataset contains fictional data, which was intended to represent data collected in the 30 days prior to the start of the analysis.

- Cited: The data is not cited, but it is in the public domain.

#### Data Cleaning 

**A**. **Changing Data Type** in the following collumn: 

- Became Member on: This column represents dates but is stored as an **integer** in the format of YYYY-MM-DD. I transformed it before converting it to a  YYYY-MM-DD **date** format. 

```ruby
#date(
    Number.IntegerDivide([became_member_on], 10000), // Extract year
    Number.IntegerDivide(Number.Mod([became_member_on], 10000), 100), // Extract month
    Number.Mod([became_member_on], 100) // Extract day
)
```

- **Income:** Originally, it was Whole Number, I changed it to 'Fixed Decimal Number'.
  
**B**. **Filtering rows**: 

- Age column: Rows with an age of 118 were removed, as they consistently lacked additional information, including gender and income. Filtering out these rows also eliminated all previously null entries in other columns. 

![image](https://github.com/user-attachments/assets/1fb84450-eb25-4396-81b4-7aecfbe7a250)


![image](https://github.com/user-attachments/assets/f8dcb289-e119-4ef5-850b-8a663c4b3bce)


### Table: Offers

This table serves as a Dimension table. 

Number of Rows: 10

Number of Columns: 6

####  Data Completness:

No missing values or inconsistencies. 

Duplicates:  The 'offer_type' column contains duplicate values, which is expected since multiple offers can belong to the same type. However, each offer has a unique 'offer_id'.

Channels Column: The values in the 'channels' column are stored as lists, as some offers are available across multiple platforms.

#### Data Cleaning 

The table 'offers' needed no cleaning. 


### Table: Events 

This table serves as the fact table because it contains transactional data (customer interactions with offers). Each row represents an event, such as an offer being received, viewed, or completed.

Number of Rows: 306534

Number of Columns: 5

####  Data Completness:

No missing values or inconsistencies. 

#### Data Cleaning 

- Column Value: Contains  inconsistent data formats in the form of nested JSON-like data, e.g., {'offer id': '9b98b8c7a33c4b65b9aebfe6a799e6d9'}, {'amount': 0.05}, {'amount': 0.30000000000000004}. Further investigation revealed that this column **is context-dependent**, meaning its contents vary based on the event type. Specifically, It stored **transaction amounts for purchases**, **offer IDs for offer-related events**, and **rewards for completed offers.**. This inconsistency needed to be addressed before establishing relationships with the offers dimention table.


![image](https://github.com/user-attachments/assets/e74f4d8f-4d71-4cc5-9258-b46247aefbf3)


![image](https://github.com/user-attachments/assets/4e2b545a-dc16-4b0b-8783-1fd60c47cd43)


**Handling Mixed Data in the Value Column**


I utilized Python to parse and extract structured data from the 'value' column, which contained nested dictionaries with varying keys. Using ast.literal_eval(), I converted the string representations of dictionaries into actual dictionaries and then extracted relevant fields. The extracted values were then assigned to three new columns: 'transaction_amount', 'offer_id', and 'reward'. Python was chosen for its efficiency in handling complex data structures and its ability to automate the transformation process.






- Event column: To optimize performance and ensure consistency, the event column was transformed into a separate dimension table (dim_event). This reduces storage space, improves query efficiency, and replaces repetitive text with an integer key for better performance. As a result, the 'event' column was replaced with event_id.


### Table: Dates 

To improve performance and enable efficient time-based analysis, a separate Dates Table was created. This allows for better date hierarchies, optimized relationships, and enhanced filtering across the dataset.

```ruby
let
    // Get the minimum and maximum date from customers_dim
    MinDate = List.Min(customers_dim[Date]),
    MaxDate = List.Max(customers_dim[Date]),

    // Generate a list of dates from MinDate to MaxDate
    DateList = List.Dates(MinDate, Number.From(MaxDate - MinDate) + 1, #duration(1,0,0,0)),

    // Convert list to a table
    DateTable = Table.FromList(DateList, Splitter.SplitByNothing(), {"Date"}),

    // Change type to Date
    ChangedType = Table.TransformColumnTypes(DateTable, {{"Date", type date}}),

    // Add necessary columns
    AddYear = Table.AddColumn(ChangedType, "Year", each Date.Year([Date]), Int64.Type),
    AddMonth = Table.AddColumn(AddYear, "Month", each Date.Month([Date]), Int64.Type),
    AddMonthName = Table.AddColumn(AddMonth, "Month Name", each Date.ToText([Date], "MMM"), type text),
    AddWeek = Table.AddColumn(AddMonthName, "Week Number", each Date.WeekOfYear([Date]), Int64.Type),
    AddDayName = Table.AddColumn(AddWeek, "Day Name", each Date.ToText([Date], "ddd"), type text),
    AddQuarter = Table.AddColumn(AddDayName, "Quarter", each "Q" & Number.ToText(Date.QuarterOfYear([Date])), type text),
    #"Filtered Rows" = Table.SelectRows(AddQuarter, each true)
in
    #"Filtered Rows"
```

## Modeling 

![image](https://github.com/user-attachments/assets/dfc68d6d-c25f-4aa4-9fc8-0866ec3b6b7e)

The column 'time' in the Fact table represents the number of hours elapsed since the start of the 30-day period, essentially acting as a timestamp for each event. 



