# Coffee-Rewards

Ongoing Project!!!


## Data Credibility

- Reliable:  The current dataset is accurate and complete; however, bias issues cannot be determined. The **age** distribution of customers ranges from **18 to 118**. Notably, **all rows with an age of 118 have null values in the 'gender' and 'income' columns**. Additionally, there are 253 rows corresponding to customers aged 90-101, with 17 of them between 100-101. Unlike the 118-year-old entries, **the 90-101 age group has no missing values in other columns**, suggesting they may not be typos. However, **it is worth noting that customers over 100 purchasing coffee might be unrealistic and could indicate potential data quality concerns.** I removed only the rows where the age was 118.

- Original: The data was downloaded from Maven Analytics but was originally sourced from Kaggle, via Udacity.

- Comprehensive: Contains all essential information needed to answer key questions.

- Current: Not up to date — there is no information regarding the date of data collection. However, this dataset contains fictional data, which was intended to represent data collected in the 30 days prior to the start of the analysis.

- Cited: The data is not cited, but it is in the public domain.


## Data Cleaning 


### Table: Customers 

This table serves as a Dimension table. 

Number of Rows: 17.000

Number of Columns: 5

**Missing values** were found in the 'Gender' and 'Income' columns, corresponding to entries where the 'Age' is 118.

**Cleaning:**

- **Became Member on:** This column is populated with dates but is stored as an **integer** in the format of YYYY-MM-DD. I transformed it before converting it to a  YYYY-MM-DD **date** format. 

```ruby
#date(
    Number.IntegerDivide([became_member_on], 10000), // Extract year
    Number.IntegerDivide(Number.Mod([became_member_on], 10000), 100), // Extract month
    Number.Mod([became_member_on], 100) // Extract day
)
```


-**Age:** Rows with an age of 118 were removed. Filtering out these rows also eliminated all null entries in the 'Gender' and 'Income' columns. 

![image](https://github.com/user-attachments/assets/1fb84450-eb25-4396-81b4-7aecfbe7a250)


![image](https://github.com/user-attachments/assets/f8dcb289-e119-4ef5-850b-8a663c4b3bce)


### Table: Offers

This table serves as a Dimension table. **It needed no cleaning.**

Number of Rows: 10

Number of Columns: 6

No missing values or inconsistencies. 

**Duplicates:**  The 'offer_type' column contains duplicate values, which is expected since multiple offers can belong to the same type. However, each offer has a unique 'offer_id'.

**Channels Column:** The values in the 'channels' column are stored as lists, as some offers are available across multiple platforms.


### Table: Events_fact

This table serves as the fact table because it contains transactional data (customer interactions with offers). Each row represents an event, such as an offer being received, viewed, or completed.

Number of Rows: 306534

Number of Columns: 5

No missing values or inconsistencies.

**Cleaning:**

- **Event column:** To optimize performance and ensure consistency, the event column was transformed into a separate dimension table (event_type). This reduces storage space, improves query efficiency, and replaces repetitive text with an integer key for better performance. As a result, the 'event' column was replaced with event_id.


![image](https://github.com/user-attachments/assets/9f4e2bc5-6ffb-4bf1-b378-05c8f64d6d75)


![image](https://github.com/user-attachments/assets/350c14a2-d35d-4880-9e3d-13472de63219)


- **Value:** Contains  inconsistent data formats in the form of nested JSON-like data, e.g., {'offer id': '9b98b8c7a33c4b65b9aebfe6a799e6d9'}, {'amount': 0.05}, {'amount': 0.30000000000000004}. Further investigation revealed that this column **is context-dependent**, meaning its contents vary based on the event type.


1. Offer-related events (offer received, offer viewed, offer completed) → Contained an offer ID.
2. Transactions (transaction) → Contained only the transaction amount.
3. Completed offers (offer completed) → Included both an offer ID and a reward amount.

![image](https://github.com/user-attachments/assets/651c36e0-4008-4d09-a241-3ddf858bb9fb)

![image](https://github.com/user-attachments/assets/e6c2d8b5-ebb4-4375-9989-dae6fa2202c2)

![image](https://github.com/user-attachments/assets/242b8d10-0b8a-4367-a802-2c8a78b89772)



In addition:

There is a small inconsistency in the **key naming**:

1. "offer completed" has ['offer_id'] with underscore

![image](https://github.com/user-attachments/assets/677a381b-c9cf-419c-89b5-0ab3b415e130)


2. "offer received" and "offer viewed" have ['offer id'] with space

![image](https://github.com/user-attachments/assets/d7a51b95-5045-422a-8448-a669aa7c6b05)


![image](https://github.com/user-attachments/assets/dccaa300-f9c1-4ab1-950a-d822242bc70b)


4. "transaction" contains only ['amount']

![image](https://github.com/user-attachments/assets/36cdf935-0ad3-417e-b131-025e3fa487d2)

**And all together from Python (pandas)**

![image](https://github.com/user-attachments/assets/5be8957c-d3f9-4dc0-989b-a640cb8d9545)



I used Python to parse and extract structured data from the 'value' column. Using ast.literal_eval(), I converted string representations of dictionaries into actual dictionaries and extracted the relevant fields. The extracted values were then assigned to three new columns: 'transaction_amount', 'offer_id', and 'reward'. Python was chosen for its efficiency in handling complex data structures and automating the transformation process.


![image](https://github.com/user-attachments/assets/802b20c5-e56e-4d74-b76b-b40b993ef2f7)


After extracting the new columns, NULL values appear in certain rows because the corresponding event category does not require those values. For example, if the event is "received order," the reward column will be NULL since rewards are only applicable to "completed orders." Similarly, transaction amounts will be NULL when related to offer_id and reward. 

![image](https://github.com/user-attachments/assets/6eaef01d-ad9c-4f24-a024-a2141c9c9a88)

![image](https://github.com/user-attachments/assets/083a046c-f7b9-46d5-9f06-ce5a614521b8)


Transaction_amount: Nulls were replaced with 0, as NULL transaction_amount **means no transaction happened**, so replacing it with 0 is fine.

Reward: A NULL reward **doesn't mean the customer received 0 rewards** —it simply indicates that the row isn't eligible for rewards, such as in "offer received" or "offer viewed" events. To avoid confusion, I left it as NULL, even though it's not the best practice. However, given the context, this approach makes the most sense here.



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

![image](https://github.com/user-attachments/assets/0584ec38-2552-476b-bfbd-45c43bdbf892)


The column 'time' in the Fact table represents the number of hours elapsed since the start of the 30-day period, essentially acting as a timestamp for each event. 


## Data Analysis 

I grouped customers based on age and income to analyze patterns more effectively. The age groups were defined within the range 18 to 101, while the income groups were categorized between $30,000 and $120,000. This segmentation allows for better insights into customer behavior, offer engagement, and spending habits across different demographics.

Age Groups:
- 18-25 (Young Adults)
- 26-35 (Early Career)
- 36-50 (Mid-Career)
- 51-65 (Late Career)
- 66+ (Retirement Age)

Income Groups:
- 30K-50K (Low Income)
- 50K-75K (Middle Income)
- 75K-100K (Upper Middle Income)
- 100K-120K (High Income)




