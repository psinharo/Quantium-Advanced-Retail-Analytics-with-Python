# Quantium-Advanced-Retail-Analytics-with-Python
  This project aims to leverage data analytics and Python to evaluate the effectiveness of a trial store  experiment in the retail sector. The analysis aimed to determine whether introducing strategic changes in selected  trial stores resulted in a significant improvement in sales performance compared to control stores
... analytics project\Quantium_data_analytics_project.py
 1
 # Re-import necessary libraries since the execution state was reset
 from collections import Counter
 from scipy.stats import ttest_ind
 import re
 import pandas as pd
 import matplotlib.pyplot as plt
 import seaborn as sns
 #Customer_Behaviour_data_exploration
 file_path = r"C:\Users\priya\Downloads\QVI_purchase_behaviour.csv"
 df = pd.read_csv(file_path)
 print(df.head()) 
# Display basic information
 purchase_behaviour_info = df.info()
 # Summary statistics
 purchase_behaviour_summary = df.describe()
 print(purchase_behaviour_summary)
 # Check for missing values
 missing_values = df.isnull().sum()
 print(missing_values)
 # Check for duplicate LYLTY_CARD_NBR entries
 duplicate_entries = df.duplicated(subset=['LYLTY_CARD_NBR']).sum()
 print(duplicate_entries)
 # Count unique values in LIFESTAGE and PREMIUM_CUSTOMER
 lifestage_counts = df["LIFESTAGE"].value_counts()
 print(lifestage_counts)
 premium_counts = df["PREMIUM_CUSTOMER"].value_counts()
 print(premium_counts)
 # Cross-analyze LIFESTAGE vs. PREMIUM_CUSTOMER to find key customer segments
 # Create a contingency table (cross-tabulation)
 lifestage_premium_crosstab = pd.crosstab(df["LIFESTAGE"], df
 ["PREMIUM_CUSTOMER"])
 print(lifestage_premium_crosstab)
 # Normalize to get percentage distribution within each lifestage category
 lifestage_premium_crosstab_normalized = lifestage_premium_crosstab.div(
... analytics project\Quantium_data_analytics_project.py
 2
 lifestage_premium_crosstab.sum(axis=1), axis=0
 ) * 100
 print(lifestage_premium_crosstab_normalized) 
#Transaction data exploration
 file_path1 = r"C:\Users\priya\Downloads\QVI_transaction_data.csv"
 df1 = pd.read_csv(file_path1)
 print(df1.head()) 
print("Column Data Types:\n", df1.dtypes)
 numeric_columns = ["DATE", "STORE_NBR", "LYLTY_CARD_NBR", "TXN_ID", 
"PROD_NBR" ,"PROD_NAME"
 ,"PROD_QTY" , "TOT_SALES"]  # Replace with actual
 column names expected to be numeric
 for col in numeric_columns:
 if not pd.api.types.is_numeric_dtype(df1[col]):
 print(f"⚠️ Warning: Column '{col}' is not in numeric format!")
 date_columns = ["DATE"]  # Replace with actual date column names
 for col in date_columns:
 try:
 df1[col] = pd.to_datetime(df1[col])
 print(f"✅ Column '{col}' successfully converted to datetime format.")
 except Exception as e:
 print(f"⚠️ Warning: Column '{col}' could not be converted to datetime.
 Error: {e}")
 # Display unique product names
 unique_products = df1["PROD_NAME"].unique()
 print("Unique Products in PROD_NAME:\n", unique_products)
 # Check for unwanted product types (e.g., non-chip items)
 unwanted_keywords = ["DIP", "SALSA", "DRINK", "HOT"]  # Modify if needed
 filtered_products = [p for p in unique_products if any(word in p.upper() for 
word in unwanted_keywords)]
 print("\n 
Potentially Unwanted Products:\n", filtered_products)
 # Filter out non-chip products (if needed)
 df1_cleaned = df1[~df1["PROD_NAME"].str.upper().str.contains('|'.join
 (unwanted_keywords))]
 print(f"\n✅ Cleaned dataset now has {df1_cleaned.shape[0]} rows after 
removing non-chip products.")
 # Function to clean product names (remove digits & special characters)
 def clean_product_name(name):
... analytics project\Quantium_data_analytics_project.py
 3
 name= re.sub(r"[^a-zA-Z\s]", "", name)# Keep only letters and spaces
 name = re.sub(r"g$", "", name)
 return name
 # Apply cleaning function to PROD_NAME
 df1["CLEANED_PROD_NAME"] = df1["PROD_NAME"].apply(clean_product_name)
 print(df1.head())
 # Split product names into individual words
 all_words = " ".join(df1["CLEANED_PROD_NAME"]).split()
 print(df1.head())
 # Count frequency of each word
 word_counts = Counter(all_words)
 # Sort words by frequency (most common first)
 sorted_words = sorted(word_counts.items(), key=lambda x: x[1], reverse=True)
 # Display top 20 most common words
 print("Top 20 most frequent words in product names:\n")
 for word, count in sorted_words[:20]:
 print(f"{word}: {count}")
 # Check for missing values
 print("Missing Values per Column:\n", df1.isnull().sum())
 #  Get summary statistics
 print("\nSummary Statistics:\n", df1.describe())
 #  Identify possible outliers using boxplots (only for numeric columns)
 numeric_columns = df1.select_dtypes(include=["number"]).columns  # Select only 
numeric columns
 # Create boxplots for numeric columns
 plt.figure(figsize=(10, 6))
 df1[numeric_columns].boxplot()
 plt.xticks(rotation=45)
 plt.title("Boxplot of Numeric Columns to Detect Outliers")
 plt.show()
 # Filter the dataset for transactions where PROD_QTY is 200
 outlier_transactions = df1[df1['PROD_QTY'] == 200]
 # Display the details of these transactions
 print(outlier_transactions)
 # Find the loyalty card number of the customer with PROD_QTY = 200
... analytics project\Quantium_data_analytics_project.py
 4
 outlier_customer_id = df1[df1['PROD_QTY'] == 200]['LYLTY_CARD_NBR'].iloc[0]
 # Display the customer ID
 print(f"Customer to be removed: {outlier_customer_id}")
 # Remove all transactions from this customer
 df1_filtered = df1[df1['LYLTY_CARD_NBR'] != outlier_customer_id]
 # Display confirmation
 print(f"Removed customer {outlier_customer_id}. New dataset size: 
{df1_filtered.shape}")
 # Confirm that the removed customer(s) do not exist in the filtered dataset
 print("Unique CARD_NBRs in cleaned dataset:", df1_filtered
 ['LYLTY_CARD_NBR'].nunique())
 # Ensure the DATE column is in datetime format
 df1_filtered['DATE'] = pd.to_datetime(df1_filtered['DATE'])
 # Count the number of transactions per date
 transactions_by_date = df1_filtered.groupby('DATE').size().reset_index
 (name='Transaction_Count')
 # Display summary of transaction count by date.
 print(transactions_by_date.info())
 # Create a complete date range from 1 Jul 2018 to 30 Jun 2019
 date_range = pd.date_range(start='2018-07-01', end='2019-06-30')
 # Convert to DataFrame
 complete_dates_df = pd.DataFrame(date_range, columns=['DATE'])
 # Merge with the existing transaction counts
 merged_df = complete_dates_df.merge(transactions_by_date, on='DATE', 
how='left')
 # Display the missing date(s)
 missing_dates = merged_df[merged_df['Transaction_Count'] == 0]
 print("Missing date(s):")
 print(missing_dates)
... analytics project\Quantium_data_analytics_project.py
 5
 # Plot transactions with missing date filled
 plt.figure(figsize=(12, 6))
 plt.plot(merged_df['DATE'], merged_df['Transaction_Count'], marker='o', 
linestyle='-')
 plt.xlabel('Date')
 plt.ylabel('Number of Transactions')
 plt.title('Number of Transactions Over Time (with Missing Dates Identified)')
 plt.xticks(rotation=45)
 plt.grid()
 # Highlight missing dates in red
 plt.scatter(missing_dates['DATE'], missing_dates['Transaction_Count'], 
color='red', label='Missing Date', zorder=3)
 plt.legend()
 plt.show()
 # Extract pack size using regex (only the first number found)
 df1["PACK_SIZE"] = df1["PROD_NAME"].apply(lambda x: int(re.search(r"\d+", 
x).group()) if re.search(r"\d+", x) else None)
 # Check if pack sizes are reasonable
 print(df1["PACK_SIZE"].value_counts().sort_index())
 # Summary of pack sizes
 print(df1.groupby("PACK_SIZE").size().reset_index(name="Count").sort_values
 ("PACK_SIZE"))
 # Set figure size
 plt.figure(figsize=(10, 5))
 # Plot histogram
 sns.histplot(df1["PACK_SIZE"], bins=20, discrete=True, kde=False)
 # Formatting
 plt.xlabel("Pack Size (g)")
 plt.ylabel("Number of Transactions")
 plt.title("Number of Transactions by Pack Size")
 plt.xticks(rotation=45)  # Rotate for readability
 plt.show()
 # Extract brand name (first word in PROD_NAME)
 df1["BRAND"] = df1["PROD_NAME"].apply(lambda x: x.split()[0])
 # Display unique brand names and their counts
 print(df1["BRAND"].value_counts().sort_index())
 # Standardize brand names
 df1["BRAND"] = df1["BRAND"].replace({
 "Red": "RRD",  # Red Rock Deli
... analytics project\Quantium_data_analytics_project.py
 6
 "Snbts": "SUNBITES", # Sunbites
 "Sunbites": "SUNBITES",
 "Infzns": "INFUZIONS",  # Infuzions
 "Infuzions": "INFUZIONS",
 "Woolworths": "WOOLWORTHS",  # Woolworths
 "WW": "WOOLWORTHS"
 })
 # Check if adjustments worked
 print(df1["BRAND"].value_counts())
 print(df1["BRAND"].unique())
 #This helps identify the most popular chip brands by total quantity sold.
 brand_sales = df1.groupby("BRAND")["PROD_QTY"].sum().reset_index()
 brand_sales = brand_sales.sort_values(by="PROD_QTY", ascending=False)
 # Display top brands
 print(brand_sales.head(10))
 #Brands with higher average spend may have premium pricing or larger pack 
sizes.
 brand_avg_spend = df1.groupby("BRAND")["TOT_SALES"].mean().reset_index()
 brand_avg_spend = brand_avg_spend.sort_values(by="TOT_SALES", ascending=False)
 # Display top brands by average spend per transaction
 print(brand_avg_spend.head(10))
 #High transaction count means frequent repeat purchases for a brand.
 brand_purchase_count = df1["BRAND"].value_counts().reset_index()
 brand_purchase_count.columns = ["BRAND", "TRANSACTION_COUNT"]
 # Display top brands by purchase frequency
 print(brand_purchase_count.head(10))
 # Plot top 10 brands by total sales
 plt.figure(figsize=(12, 6))
 sns.barplot(x=brand_sales["BRAND"][:10], y=brand_sales["PROD_QTY"][:10], 
palette="viridis")
 plt.xticks(rotation=45)
 plt.xlabel("Brand")
 plt.ylabel("Total Quantity Sold")
 plt.title("Top 10 Chip Brands by Total Sales")
 plt.show()
 merged_df =  df1.merge(df, on="LYLTY_CARD_NBR", how="left")
 # Check how many transactions still have missing customer info
 print(merged_df.head())
... analytics project\Quantium_data_analytics_project.py
 7
 # Check for missing customer details
 missing_customers = merged_df[merged_df['LIFESTAGE'].isnull()]  # Assuming 
'LIFESTAGE' is from customer data
 # Print the count of unmatched transactions
 print(f"Number of transactions without customer data: {missing_customers.shape
 [0]}")
 # Display a few rows to inspect
 print(missing_customers.head())
 # Save merged dataset to a CSV file
 merged_df.to_csv("final_dataset.csv", index=False)
 # Calculate total sales
 merged_df["TOTAL_SALES"] = merged_df["TOT_SALES"]
 # Aggregate sales by LIFESTAGE and PREMIUM_CUSTOMER
 sales_by_segment = (
 merged_df.groupby(["LIFESTAGE", "PREMIUM_CUSTOMER"])["TOTAL_SALES"]
 .sum()
 .reset_index()
 )
 # Plot the sales distribution
 plt.figure(figsize=(12, 6))
 sns.barplot(
 data=sales_by_segment,
 x="LIFESTAGE",
 y="TOTAL_SALES",
 hue="PREMIUM_CUSTOMER",
 palette="viridis",
 )
 plt.title("Total Sales by LIFESTAGE and PREMIUM_CUSTOMER", fontsize=14)
 plt.xlabel("Lifestage", fontsize=12)
 plt.ylabel("Total Sales", fontsize=12)
 plt.xticks(rotation=45)
 plt.legend(title="Premium Customer")
 plt.show()
 #We count unique customers in each LIFESTAGE and PREMIUM_CUSTOMER segment,This 
gives the number of unique customers in each segment.
 customers_per_segment = merged_df.groupby(["LIFESTAGE", "PREMIUM_CUSTOMER"])
 ["LYLTY_CARD_NBR"].nunique().reset_index()
 customers_per_segment.rename(columns={"LYLTY_CARD_NBR": "NUM_CUSTOMERS"}, 
inplace=True)
 print(customers_per_segment)
... analytics project\Quantium_data_analytics_project.py
 8
 #We calculate the total number of chips bought per segment and divide by the 
number of customers,This tells us how many chip packs a typical customer in 
each segment buys.
 chips_per_customer = (
 merged_df.groupby(["LIFESTAGE", "PREMIUM_CUSTOMER"])["PROD_QTY"].sum()
 / merged_df.groupby(["LIFESTAGE", "PREMIUM_CUSTOMER"])
 ["LYLTY_CARD_NBR"].nunique()
 ).reset_index()
 chips_per_customer.rename(columns={0: "CHIPS_PER_CUSTOMER"}, inplace=True)
 print(chips_per_customer)
 #We divide total sales by the total quantity of chips to get the average price 
per pack.
 avg_chip_price = (
 merged_df.groupby(["LIFESTAGE", "PREMIUM_CUSTOMER"])["TOT_SALES"].sum()
 / merged_df.groupby(["LIFESTAGE", "PREMIUM_CUSTOMER"])["PROD_QTY"].sum()
 ).reset_index()
 avg_chip_price.rename(columns={0: "AVG_CHIP_PRICE"}, inplace=True)
 print(avg_chip_price)
 # Extracting price per unit for each group
 mainstream_prices = merged_df[merged_df["PREMIUM_CUSTOMER"] == "Mainstream"]
 ["TOT_SALES"] / merged_df[merged_df["PREMIUM_CUSTOMER"] == "Mainstream"]
 ["PROD_QTY"]
 premium_prices = merged_df[merged_df["PREMIUM_CUSTOMER"] == "Premium"]
 ["TOT_SALES"] / merged_df[merged_df["PREMIUM_CUSTOMER"] == "Premium"]
 ["PROD_QTY"]
 budget_midage_prices = merged_df[(merged_df["LIFESTAGE"] == "Mid Age Singles/
 Couples") & (merged_df["PREMIUM_CUSTOMER"] == "Budget")]["TOT_SALES"] / 
merged_df[(merged_df["LIFESTAGE"] == "Mid Age Singles/Couples") & (merged_df
 ["PREMIUM_CUSTOMER"] == "Budget")]["PROD_QTY"]
 young_singles_prices = merged_df[(merged_df["LIFESTAGE"] == "Young Singles/
 Couples")]["TOT_SALES"] / merged_df[(merged_df["LIFESTAGE"] == "Young 
Singles/Couples")]["PROD_QTY"]
 #Perform Independent t-tests Mainstream vs. Premium Customers
 t_stat1, p_value1 = ttest_ind(mainstream_prices.dropna(), premium_prices.dropna
 (), equal_var=False)
 print(f"T-test Mainstream vs Premium Customers: t-statistic = {t_stat1:.4f}, p
value = {p_value1:.4f}")
 # Budget Mid-Age vs. Young Singles and Couples
 t_stat2, p_value2 = ttest_ind(budget_midage_prices.dropna(), 
young_singles_prices.dropna(), equal_var=False)
... analytics project\Quantium_data_analytics_project.py
 9
 print(f"T-test Budget Mid-Age vs Young Singles/Couples: t-statistic = 
{t_stat2:.4f}, p-value = {p_value2:.4f}")
 #1. Mainstream vs. Premium Customers Interpretation
 #T-statistic = 11.6221, p-value = 0.0000
 #The large t-statistic suggests a significant difference in average prices 
between Mainstream and Premium customers.
 #The p-value is 0.0000, meaning the difference is statistically significant at 
any reasonable significance level (e.g., 0.05).
 #This means Premium customers pay significantly more per unit of chips compared
 to Mainstream customers.
