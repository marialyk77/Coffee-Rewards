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

