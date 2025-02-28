# Coffee-Rewards

Ongoing Project!!



## Dataset Exploration

### Table: Customers 

Number of Rows: 17.000

Number of Columns: 5

####  Data Completness:

Missing Values Found In: Gender, Income 

#### Data Consistency:

- Age column: Ranges from 18 to 118.


#### Data Credibility

- Reliable:  The current dataset is accurate and complete; however, bias issues cannot be determined. The age distribution ranges from 18 to 118. Notably, all rows with an age of 118 have null values in the gender and income columns. Additionally, there are 253 rows corresponding to customers aged 90-101, with 17 of them between 100-101. Unlike the 118-year-old entries, the 90-101 age group has no missing values in other columns, suggesting they may not be typos. However, it is worth noting that customers over 100 purchasing coffee might be unrealistic and could indicate potential data quality concerns.

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

