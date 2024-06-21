## Data QA/QC Internship task Datahut

# Summary Document

## Table of Contents

- [Data Overview](#data-overview)
- [Removing Duplicates](#removing-duplicates)
- [Handling Null Values](#handling-null-values)
- [Correcting Email Formats](#correcting-email-formats)
- [Standardising Date Formats](#standardising-date-formats)
- [Correcting Department Names](#correcting-department-names)
- [Handling Noise in Salary and Age](#handling-noise-in-salary-and-age)
  
## Data Overview

Data cleaning is vital for accurate and meaningful analysis. This project focuses on transforming a raw employee database into a reliable and analyzable dataset by addressing errors, inconsistencies, and missing values.

**Data Dictionary**

- ID: Unique identifier for each employee.
- Name: Employee's full name.
- Age: Employee's age.
- Email: Employee’s official email address.
- Join Date: Date of employment.
- Salary: Annual salary.
- Department: Department where the employee works.

# Data Processing

1. ## Removing Duplicates

The 'ID' column, intended as a unique identifier, contained 1000 duplicate entries. These duplicates were removed to maintain the integrity of the dataset.

```python
data.drop_duplicates(subset='ID', keep='first', inplace=True)
```

2. ## Handling Null Values

Dataset contained 6 columns with null values. Proceeded with handling them after conducting Exploratory Data Analysis (EDA) to examine potential relationships between variables. EDA revealed no significant correlations among the variables.

Method of handling : 

**Email**
- Removed records with null values in the 'Email' column. The email address is essential as it is the primary means of communication and uniquely identifies each employee. Therefore, retaining only records with valid email addresses ensures data integrity and utility.

```python
data.dropna(subset=['Email'], inplace=True)
```

**Name**
- Replaced null values with the placeholder 'Unknown'. Since the 'Name' field is a categorical variable and most people do not share the same first and last name, using a placeholder ensures completeness while avoiding confusion with existing names.

```python
data['Name'].fillna('Unknown', inplace=True)
```

**Age**
- Imputed null values with random values drawn from the existing age data. The distribution graph of the 'Age' column indicated a roughly uniform distribution. By using random values from the existing data, this method preserves the uniform nature of the age distribution, ensuring the imputed values blend seamlessly into the dataset.

```python
data['Age'] = data['Age'].apply(lambda x: np.random.uniform(18, 90) if pd.isnull(x) else x)
```

**Join Date**
- Filled null values with random dates between 2020 and the last recorded join date in 2024. The dataset showed a significant increase in recruitments during these years. This approach maintains the temporal relevance of the data and aligns with the observed hiring trends, providing realistic imputed values that reflect the actual hiring patterns.

```python
start_date = datetime(2020, 1, 1)
end_date = datetime(2024, 6, 13)

# Function to randomly generate dates within a range
def random_date(start, end):
    return start + timedelta(days=random.randint(0, (end - start).days))

# Filter rows with missing 'Join Date'
missing_dates = data['Join Date'].isnull()

# Generate random dates and fill missing 'Join Date'
data.loc[missing_dates, 'Join Date'] = [random_date(start_date, end_date) for _ in range(missing_dates.sum())]
```

**Department**
- Replaced null entries with random departments. Analysis of the dataset indicated a relatively even distribution of employees across departments, making random assignment suitable for maintaining data uniformity without introducing bias.

```python
existing_departments = data['Department'].dropna().unique()

# Function to uniformly impute null values with existing departments
def fill_missing_department(dept):
    if pd.isnull(dept):
        return random.choice(existing_departments)
    else:
        return dept

# Apply the function to 'Department' column
data['Department'] = data['Department'].apply(fill_missing_department)
```

**Salary**
- Imputed null values with the median salary of the respective department. The histogram of salary data showed positive skewness, making the median a more robust measure than the mean since its less affected by skewed data. This method ensures that imputed salary values are representative of typical earnings within each department, maintaining data integrity and consistency for analysis purposes.

```python
data['Salary'].fillna(data.groupby('Department')['Salary'].transform('median'), inplace=True)
```

3. ## Correcting Email formats
- Standardized email formats by creating a function that added '@' before domain names that were missing it, ensuring all email addresses conform to the standard format (e.g., username@domain.com).
```python
def correct_email_provider(email):
    # Remove leading and trailing whitespaces
    email = email.strip()
    providers = ['gmail', 'yahoo', 'hotmail']
    for provider in providers:
        # Check if the email ends with the provider without '@'
        if email.endswith(provider + ".com") and not email.endswith('@' + provider + ".com"):
            email = email.replace(provider + ".com", '@' + provider + ".com")
    return email

# Apply the function to correct the 'Email' column
data['Email'] = data['Email'].apply(correct_email_provider)
```

- Removed addresses that did not follow a standard email format using regex within a function, ensuring that all retained emails are valid and usable for communication and analysis purposes.
```python
# Define a function to validate email addresses
def is_valid_email(email):
    # Remove leading and trailing whitespaces
    email = email.strip()
    # Regular expression for validating an email address
    email_regex = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return re.match(email_regex, email) is not None

# Apply the function to filter the DataFrame

data = data[data['Email'].apply(is_valid_email)]
```

4. ## Standardising Date Formats
- Ensured all dates in the 'Join Date' column followed a consistent format (e.g., YYYY-MM-DD) by applying a function that standardized the date format across the column.
```python
def parse_mixed_date_format(date_str):
    try:
        # First, try parsing as day/month/year
        return pd.to_datetime(date_str, format='%d/%m/%Y')
    except ValueError:
        try:
            # If the first format fails, try parsing as year-month-day
            return pd.to_datetime(date_str, format='%Y-%m-%d')
        except ValueError:
            # If both formats fail, return NaT
            return pd.NaT

# Apply the parsing function to 'Join Date'
data['Join Date'] = data['Join Date'].apply(parse_mixed_date_format)
```

5. ## Correcting Department Names:
- Standardized department names by correcting typos and variations to ensure consistency. Departments were standardized to common formats such as HR, Engineering, Marketing, Sales, and Support.
```python
data.loc[data['Department'].str.contains('Support', case=False, na=False), 'Department'] = 'Support'
data.loc[data['Department'].str.contains('Engineering', case=False, na=False), 'Department'] = 'Engineering'
data.loc[data['Department'].str.contains('Sales', case=False, na=False), 'Department'] = 'Sales'
data.loc[data['Department'].str.contains('Marketing', case=False, na=False), 'Department'] = 'Marketing'  
data.loc[data['Department'].str.contains('HR', case=False, na=False), 'Department'] = 'HR'
```

6. ## Handling noise in Salary and Age
- Converted the data type of both Salary and Age columns to integer to remove noise and ensure consistency in numerical representation.
```python
data['Salary'] = data['Salary'].astype(int)
data['Age'] = data['Age'].astype(int)
```

- Ensured Salary values do not contain outliers by visualizing the distribution using a box plot. This graphical representation helps identify any extreme values that could distort analysis, ensuring the dataset accurately represents the salary distribution within reasonable bounds.
```python
data['Salary'].plot(kind='box')
```

7. ## Cleaning Name Fields
- Removed honorifics such as 'Mr.', 'Mrs.', 'Dr.', etc., from the beginning of names using regex to standardize name entries.

- Eliminated trailing capital letters indicating professional titles like 'DDS', 'MD', etc., using regex to ensure consistency in name formats.

- Corrected trailing capital letters with typos, such as 'DDSraise', by removing them using regex.

```python
def clean_name(name):
    # Remove 'Mr.' or 'Mrs.' and so on at the beginning (case insensitive)
    #cleaned_name = re.sub(r'^(Mr\.|Mrs\.|Miss\.|Dr\.)\s+', '', name, flags=re.IGNORECASE)
    cleaned_name = re.sub(r'^([A-Za-z]+\.)?\s*', '', name, flags=re.IGNORECASE)
    
    # Remove any trailing capital letters
    cleaned_name = re.sub(r'[A-Z]+$', '', cleaned_name)

    # Remove any leading capital letters
    cleaned_name = re.sub(r'\b[A-Z]{2,}[a-z]+\b', '', cleaned_name)

    return cleaned_name.strip()  # Remove any leading or trailing whitespace

# Apply the cleaning function to the 'Name' column
data['Name'] = data['Name'].apply(clean_name)
```

- Removed extraneous words or typos attached to last names, such as 'Tonya Philipsnetwork', 'Richard Bryantwhile', using NLTK's WordNet Corpus Reader to enhance the accuracy and clarity of names in the dataset.

```python
# Function to fetch all common English words from NLTK
def fetch_common_words():
    # List of common English words from NLTK words corpus
    common_words = set(wn.words())
    
    # Convert to lowercase for consistent matching
    common_words = {word.lower() for word in common_words}
    
    return common_words

# Fetch common words
common_words = fetch_common_words()

# Function to clean the last name by removing the longest matching common suffix word
# Only applies to last names with more than 7 letters
def clean_last_name(name, common_words):
    # Split the name into parts
    name_parts = name.split()
    
    if len(name_parts) < 2:
        # If there's no last name part, return the name as is
        return name.title()

    # Extract the last name part
    last_name = name_parts[-1].lower()

    if len(last_name) > 7:  # Only consider last names with more than 7 letters
        # Iterate over possible suffix lengths to find the longest match
        for i in range(len(last_name), 0, -1):
            suffix = last_name[-i:]
            if suffix in common_words:
                last_name = last_name[:-i].strip()
                break
        else:
        # If no match found, return the original name
          return last_name

    # Reconstruct the full name with the cleaned last name
    name_parts[-1] = last_name.capitalize()
    cleaned_name = ' '.join(name_parts)

    return cleaned_name

# Apply the function to the 'Name' column
data['Name'] = data['Name'].apply(lambda x: clean_last_name(x, common_words))
```

## Saving the cleaned data set
```python
data.to_csv('cleaned_dataset.csv', index=False)
```





