from statistics import  correlation
 import pandas as  pd 
 file_path = r"C:\Users\priya\Downloads\QVI_data_task 2.csv"
 data = pd.read_csv(file_path)
 print print(data.head())
 import  matplotlib.pyplot as  plt
 import  seaborn as sns 
 sns.set_theme(style="whitegrid") 
plt.rcParams['axes.titlelocation'] = 'center'  # Center plot titles
 import numpy as np
 # Convert date column to datetime format
 data['YEARMON'] = pd.to_datetime(data['DATE']).dt.to_period('M')
 # Group by store number and month
 store_sales = data.groupby(['STORE_NBR', 'YEARMON']).agg(
    totSales=('TOT_SALES', 'sum'),
    nCustomers ('LYLTY_CARD_NBR', pd.Series.nunique),
    nTxnPerCust=('LYLTY_CARD_NBR', lambda lambda x: len(x) / x.nunique())
 ).reset_index()
 # Define pre-trial period
 pre_trial_period = ["2018-07", "2018-08", "2018-09", "2018-10", "2018-11", "2018-12", "2019
01"]
 # Filter data for pre-trial period
 pre_trial_data = store_sales[store_sales['YEARMON'].astype(str).isin(pre_trial_period)]
 print print(pre_trial_data.head())
 print print(pre_trial_data.columns)
 def def compute_correlations(pre_trial_data, trial_store):
    trial_data = pre_trial_data[pre_trial_data['STORE_NBR'] == 
trial_store].set_index("YEARMON")
    def def safe_corr(group, col):
        """ Computes correlation safely (avoiding NaN due to constant values) """
        store_data = group.set_index("YEARMON")[col]
        merged_data = pd.DataFrame({'trial': trial_data[col], 'control': store_data}).dropna()
        if if merged_data['trial'].nunique() > 1 and and merged_data['control'].nunique() > 1:
            return return merged_data['trial'].corr(merged_data['control'])
        else else:
            return 0  # Default to 0 if correlation can't be computed
    correlations = pre_trial_data.groupby('STORE_NBR',group_keys=False).apply(lambda lambda group: 
pd.Series({
        'salesCorrelation': safe_corr(group, 'totSales'),
        'customerCorrelation': safe_corr(group, 'nCustomers'),
        'txnCorrelation': safe_corr(group, 'nTxnPerCust')
    })).reset_index()
    
 
    return return correlations
 # Example: Compute correlations for trial store 77
 correlations_77 = compute_correlations(pre_trial_data, 77)
 correlations_86=compute_correlations(pre_trial_data,86)
 correlations_88=compute_correlations(pre_trial_data,88)
 print print(correlations_77.head())
print print(correlations_86.head())
 print print(correlations_88.head())
 # Compute a final score by summing the correlations for store 77
 correlations_77['controlScore'] = (correlations_77['salesCorrelation'] + 
                                   correlations_77['customerCorrelation'] + 
                                   correlations_77['txnCorrelation'])
 # Sort stores by highest score (best match)
 best_control_store_77 = correlations_77.sort_values(by='controlScore', 
ascending=False).iloc[0]
 # Print the best control store
 print print("Best control store for trial store 77:", best_control_store_77['STORE_NBR'])
 print print(best_control_store_77)
 # Function to compute magnitude distance
 def def compute_magnitude_distance(pre_trial_data, trial_store):
    trial_data = pre_trial_data[pre_trial_data['STORE_NBR'] == 
trial_store].set_index("YEARMON")
    def def magnitude_difference(group, col):
        store_data = group.set_index("YEARMON")[col]
        merged_data = pd.DataFrame({'trial': trial_data[col], 'control': store_data}).dropna()
        return return (merged_data['trial'] - merged_data['control']).abs().sum()
    magnitude_distance = pre_trial_data.groupby('STORE_NBR', group_keys=False).apply(lambda lambda 
group: pd.Series({
        'salesMagnitude': magnitude_difference(group, 'totSales'),
        'customerMagnitude': magnitude_difference(group, 'nCustomers'),
        'txnMagnitude': magnitude_difference(group, 'nTxnPerCust'),
    })).reset_index()
    # Compute total magnitude distance (lower is better)
    magnitude_distance['totalMagnitude'] = (magnitude_distance['salesMagnitude'] +
                                            magnitude_distance['customerMagnitude'] +
                                            magnitude_distance['txnMagnitude'])
    return return magnitude_distance
 # Example: Compute magnitude distance for trial store 77
 magnitude_distance_77 = compute_magnitude_distance(pre_trial_data, 77)
 # Merge correlation and magnitude distance data
 final_selection = correlations_77.merge(magnitude_distance_77, on='STORE_NBR')
 # Compute final score (higher is better)
 final_selection['finalScore'] = (final_selection['salesCorrelation'] + 
                                 final_selection['customerCorrelation'] + 
                                 final_selection['txnCorrelation']) - 
(0.1*final_selection['totalMagnitude'])
 # Exclude the trial store from the control store selection
 final_selection = final_selection[final_selection['STORE_NBR'] != 77]  # Change 77 to the 
current trial store
 # Sort and pick the best control store
 best_control_store_77 = final_selection.sort_values(by='finalScore', ascending=False).iloc[0]
 # Print the best control store
 print print("Best control store for trial store 77:", best_control_store_77['STORE_NBR'])
 print print(best_control_store_77)
 # Compute a final score by summing the correlations for store 86
 correlations_86['controlScore'] = (correlations_86['salesCorrelation'] + 
                                   correlations_86['customerCorrelation'] + 
                                   correlations_86['txnCorrelation'])
 # Sort stores by highest score (best match)
 best_control_store_86 = correlations_86.sort_values(by='controlScore', 
ascending=False).iloc[0]
 # Print the best control store
 print print("Best control store for trial store 77:", best_control_store_86['STORE_NBR'])
 print print(best_control_store_86)
 # Function to compute magnitude distance
 def def compute_magnitude_distance(pre_trial_data, trial_store):
    trial_data = pre_trial_data[pre_trial_data['STORE_NBR'] == 
trial_store].set_index("YEARMON")
    def def magnitude_difference(group, col):
        store_data = group.set_index("YEARMON")[col]
        merged_data = pd.DataFrame({'trial': trial_data[col], 'control': store_data}).dropna()
        return return (merged_data['trial'] - merged_data['control']).abs().sum()
    magnitude_distance = pre_trial_data.groupby('STORE_NBR', group_keys=False).apply(lambda lambda 
group: pd.Series({
        'salesMagnitude': magnitude_difference(group, 'totSales'),
        'customerMagnitude': magnitude_difference(group, 'nCustomers'),
        'txnMagnitude': magnitude_difference(group, 'nTxnPerCust'),
    })).reset_index()
    # Compute total magnitude distance (lower is better)
    magnitude_distance['totalMagnitude'] = (magnitude_distance['salesMagnitude'] +
                                            magnitude_distance['customerMagnitude'] +
                                            magnitude_distance['txnMagnitude'])
    return return magnitude_distance
 # Example: Compute magnitude distance for trial store 77
 magnitude_distance_86 = compute_magnitude_distance(pre_trial_data, 86)
 # Merge correlation and magnitude distance data
 final_selection = correlations_86.merge(magnitude_distance_86, on='STORE_NBR')
 # Compute final score (higher is better)
 final_selection['finalScore'] = (final_selection['salesCorrelation'] + 
                                 final_selection['customerCorrelation'] + 
                                 final_selection['txnCorrelation']) - 
(0.1*final_selection['totalMagnitude'])
 # Exclude the trial store from the control store selection
 final_selection = final_selection[final_selection['STORE_NBR'] != 86]  # Change 77 to the 
current trial store
 # Sort and pick the best control store
 best_control_store_86 = final_selection.sort_values(by='finalScore', ascending=False).iloc[0]
 # Print the best control store
 print print("Best control store for trial store 86:", best_control_store_86['STORE_NBR'])
 print print(best_control_store_86)
 # Compute a final score by summing the correlations for store 88
 correlations_88['controlScore'] = (correlations_88['salesCorrelation'] + 
                                   correlations_88['customerCorrelation'] + 
                                   correlations_88['txnCorrelation'])
 # Sort stores by highest score (best match)
best_control_store_88 = correlations_88.sort_values(by='controlScore', 
ascending=False).iloc[0]
 # Print the best control store
 print print("Best control store for trial store 88:", best_control_store_88['STORE_NBR'])
 print print(best_control_store_88)
 # Function to compute magnitude distance
 def def compute_magnitude_distance(pre_trial_data, trial_store):
    trial_data = pre_trial_data[pre_trial_data['STORE_NBR'] == 
trial_store].set_index("YEARMON")
    def def magnitude_difference(group, col):
        store_data = group.set_index("YEARMON")[col]
        merged_data = pd.DataFrame({'trial': trial_data[col], 'control': store_data}).dropna()
        return return (merged_data['trial'] - merged_data['control']).abs().sum()
    magnitude_distance = pre_trial_data.groupby('STORE_NBR', group_keys=False).apply(lambda lambda 
group: pd.Series({
        'salesMagnitude': magnitude_difference(group, 'totSales'),
        'customerMagnitude': magnitude_difference(group, 'nCustomers'),
        'txnMagnitude': magnitude_difference(group, 'nTxnPerCust'),
    })).reset_index()
    # Compute total magnitude distance (lower is better)
    magnitude_distance['totalMagnitude'] = (magnitude_distance['salesMagnitude'] +
                                            magnitude_distance['customerMagnitude'] +
                                            magnitude_distance['txnMagnitude'])
    return return magnitude_distance
 # Example: Compute magnitude distance for trial store 77
 magnitude_distance_88 = compute_magnitude_distance(pre_trial_data, 88)
 # Merge correlation and magnitude distance data
 final_selection = correlations_88.merge(magnitude_distance_88, on='STORE_NBR')
 # Compute final score (higher is better)
 final_selection['finalScore'] = (final_selection['salesCorrelation'] + 
                                 final_selection['customerCorrelation'] + 
                                 final_selection['txnCorrelation']) - 
(0.1*final_selection['totalMagnitude'])
 # Exclude the trial store from the control store selection
 final_selection = final_selection[final_selection['STORE_NBR'] != 88]  # Change 77 to the 
current trial store
 # Sort and pick the best control store
 best_control_store_88 = final_selection.sort_values(by='finalScore', ascending=False).iloc[0]
 # Print the best control store
 print print("Best control store for trial store 88:", best_control_store_88['STORE_NBR'])
 print print(best_control_store_88)
 # Function to compute percentage difference in sales
 def def compare_sales_performance(pre_trial_data, trial_store, control_store):
    trial_sales = pre_trial_data[pre_trial_data['STORE_NBR'] == trial_store][['YEARMON', 
'totSales']]
    control_sales = pre_trial_data[pre_trial_data['STORE_NBR'] == control_store][['YEARMON', 
'totSales']]
    
    # Merge data on YEARMON
    sales_comparison = trial_sales.merge(control_sales, on='YEARMON', suffixes=('_trial', 
'_control'))
    
    # Calculate percentage difference
    sales_comparison['pctDifference'] = (sales_comparison['totSales_trial'] - 
sales_comparison['totSales_control']) / sales_comparison['totSales_control']
    
    return return sales_comparison
 # Example: Compare sales for trial store 77 and its control store
 sales_comparison_77 = compare_sales_performance(pre_trial_data, 77, control_store=233)# 
Assuming 233 is the selected control store
 sales_comparison_86 = compare_sales_performance(pre_trial_data, 86, control_store=155)
 sales_comparison_88 = compare_sales_performance(pre_trial_data, 88, control_store=237)
 from scipy.stats  import  ttest_ind
 # Function to perform t-test
 # Check for extra spaces in column names
 pre_trial_data.rename(columns=lambda lambda x: x.strip(), inplace=True)
 def def perform_t_test(pre_trial_data, control_data):
    t_stat, p_value = ttest_ind(pre_trial_data['totSales'], control_data['totSales'], 
equal_var=False)
    return return {"t_stat": t_stat, "p_value": p_value}
 # Ensure there is data for store 77
 trial_sales_77 = pre_trial_data[pre_trial_data['STORE_NBR'] == 77]
 control_sales_77 = pre_trial_data[pre_trial_data['STORE_NBR'] == 233]
 # Debug prints
 print print(trial_sales_77)  # Should NOT be empty
 print print(control_sales_77)  # Should NOT be empty
 # If both have data, perform t-test
 if if not not trial_sales_77.empty and and not not control_sales_77.empty:
    t_test_result_77 = perform_t_test(trial_sales_77, control_sales_77)
    print print(t_test_result_77)
 else else:
    print print("One of the stores has no data in pre_trial_data!")
    
# Ensure there is data for store 86
 trial_sales_86 = pre_trial_data[pre_trial_data['STORE_NBR'] == 86]
 control_sales_86 = pre_trial_data[pre_trial_data['STORE_NBR'] == 155]
 # Debug prints
 print print(trial_sales_86)  # Should NOT be empty
 print print(control_sales_86)  # Should NOT be empty
 # If both have data, perform t-test
 if if not not trial_sales_86.empty and and not not control_sales_86.empty:
    t_test_result_86 = perform_t_test(trial_sales_86, control_sales_86)
    print print(t_test_result_86)
 else else:
    print print("One of the stores has no data in pre_trial_data!")
 # Ensure there is data for store 88
 trial_sales_88 = pre_trial_data[pre_trial_data['STORE_NBR'] == 88]
 control_sales_88 = pre_trial_data[pre_trial_data['STORE_NBR'] == 237]
 # Debug prints
 print print(trial_sales_88)  # Should NOT be empty
 print print(control_sales_88)  # Should NOT be empty
 # If both have data, perform t-test
 if if not not trial_sales_88.empty and and not not control_sales_88.empty:
    t_test_result_88 = perform_t_test(trial_sales_88, control_sales_88)
    print print(t_test_result_88)
 else else:
    print print("One of the stores has no data in pre_trial_data!")
 import import matplotlib.pyplot matplotlib.pyplot as as plt plt
 # Function to plot sales performance
 def def plot_sales_performance(sales_comparison):
    plt.figure(figsize=(10,5))
    plt.plot(sales_comparison['YEARMON'].astype(str), sales_comparison['totSales_trial'], 
label="Trial Store", marker='o')
    plt.plot(sales_comparison['YEARMON'].astype(str), sales_comparison['totSales_control'], 
label="Control Store", marker='s')
    
    plt.xlabel("Year-Month")
    plt.ylabel("Total Sales")
    plt.title("Trial vs Control Store Sales Performance")
    plt.xticks(rotation=45)
    plt.legend()
    plt.show()
 # Example: Plot sales performance for store 77
 plot_sales_performance(sales_comparison_77)
 plot_sales_performance(sales_comparison_86)
 plot_sales_performance(sales_comparison_88)
 #TRIAL STORE 77:
 #Step 1: Visualizing Sales Trends Before the Trial: We need to check if the trial store (77) 
and its control store (233) have similar trends in total sales before the trial period.
 # Define trial and control stores
 trial_store = 77
 control_store = 233
 # Convert YEARMONTH to datetime format for plotting
 pre_trial_data["TransactionMonth"] = pre_trial_data["YEARMON"].dt.to_timestamp()
 # Create a new column to categorize stores
 pre_trial_data["Store_type"] = pre_trial_data["STORE_NBR"].apply(
    lambda lambda x: "Trial" if if x == trial_store else else ("Control" if if x == control_store else else "Other 
stores")
 )
 # Aggregate total sales by year-month and store type
 past_sales = pre_trial_data.groupby(["TransactionMonth", "Store_type"])
 ["totSales"].mean().reset_index()
 # Filter data to include only pre-trial period
 past_sales = past_sales[past_sales["TransactionMonth"] < "2019-03-01"]
 # Plot total sales trends
 plt.figure(figsize=(12, 6))
 sns.lineplot(data=past_sales, x="TransactionMonth", y="totSales", hue="Store_type", 
style="Store_type")
 plt.xlabel("Month of Operation")
 plt.ylabel("Total Sales")
 plt.title("Total Sales by Month (Trial vs Control)")
 plt.legend(title="Store Type")
 plt.xticks(rotation=45)
 plt.show()
#Step 2: Visualizing Number of Customers Trends
 # Aggregate total customers by year-month and store type
 past_customers = pre_trial_data.groupby(["TransactionMonth", "Store_type"])
 ["nCustomers"].mean().reset_index()
 # Filter pre-trial period
 past_customers = past_customers[past_customers["TransactionMonth"] < "2019-03-01"]
 # Plot number of customers trends
 plt.figure(figsize=(12, 6))
 sns.lineplot(data=past_customers, x="TransactionMonth", y="nCustomers", hue="Store_type", 
style="Store_type")
 plt.xlabel("Month of Operation")
 plt.ylabel("Total Number of Customers")
 plt.title("Total Number of Customers by Month (Trial vs Control)")
 plt.legend(title="Store Type")
 plt.xticks(rotation=45)
 plt.show()
 #Step 3: Scaling Pre-Trial Control Sales to Match Trial Store: We compute a scaling factor to 
adjust the control store sales to match the trial store.
 # Compute scaling factor
 scaling_factor_sales = (
 pre_trial_data[(pre_trial_data["STORE_NBR"] == trial_store) & 
((pre_trial_data["YEARMON"].dt.to_timestamp() < pd.Timestamp("2019-02-01"))]
 ["totSales"].sum() /
 pre_trial_data[(pre_trial_data["STORE_NBR"] == control_store) & 
((pre_trial_data["YEARMON"].dt.to_timestamp() < pd.Timestamp("2019-02-01"))]
 ["totSales"].sum()
 )
 # Apply scaling factor
 pre_trial_data.loc[pre_trial_data["STORE_NBR"] == control_store, "scaledSales"] = (
 pre_trial_data["totSales"] * scaling_factor_sales
 )
 # Step 4: Calculating Percentage Difference Between Scaled Control and Trial Sales We compute 
The percentage difference between the scaled control sales and trial sales.
 # Merge trial and scaled control sales on YEARMONTH
 percentage_diff_sales = pre_trial_data[pre_trial_data["STORE_NBR"] == control_store]
 [["TransactionMonth", "scaledSales"]].merge(
 pre_trial_data[pre_trial_data["STORE_NBR"] == trial_store][["TransactionMonth", 
"totSales"]],
 on="TransactionMonth"
 )
 # Calculate percentage difference
 percentage_diff_sales["percentageDiff"] = abs(percentage_diff_sales["scaledSales"] - 
percentage_diff_sales["totSales"]) / percentage_diff_sales["scaledSales"]
 #Step 5: Computing Standard Deviation for Pre-Trial Period. This is used to determine confidence 
intervals
 # Compute standard deviation of percentage differences in pre-trial period
 std_dev_sales = percentage_diff_sales[percentage_diff_sales["TransactionMonth"] < "2019-02-01"]
 ["percentageDiff"].std()
 degrees_of_freedom = 7  # Since we have 8 months in pre-trial period, df = 8-1
 #Step 6: Computing Confidence Intervals
 #We calculate 95th and 5th percentile confidence intervals.
 # Compute upper and lower confidence intervals
 past_sales_controls_95 = past_sales[past_sales["Store_type"] == "Control"].copy()
 past_sales_controls_5 = past_sales[past_sales["Store_type"] == "Control"].copy()
 past_sales_controls_95["totSales"] *= (1 + std_dev_sales * 2)
past_sales_controls_95["Store_type"] = "Control 95% Confidence Interval"
 past_sales_controls_5["totSales"] *= (1 - std_dev_sales * 2)
 past_sales_controls_5["Store_type"] = "Control 5% Confidence Interval"
 trial_assessment = pd.concat([past_sales, past_sales_controls_95, past_sales_controls_5])
 #Step 7: Plotting Sales with Confidence Intervals
 # Plot total sales with confidence intervals
 plt.figure(figsize=(12, 6))
 sns.lineplot(data=trial_assessment, x="TransactionMonth", y="totSales", hue="Store_type", 
style="Store_type")
 # Highlight trial period
 plt.axvspan(pd.to_datetime("2019-02-01"), pd.to_datetime("2019-04-30"), color="gray", 
alpha=0.3)
 plt.xlabel("Month of Operation")
 plt.ylabel("Total Sales")
 plt.title("Total Sales by Month (With Confidence Intervals)")
 plt.legend(title="Store Type")
 plt.xticks(rotation=45)
 plt.show()
 #The results show that the trial in store 77 is significantly different to its control store in 
The trial period as the trial store performance lies outside the 5% to 95% confidence interval 
#of the control store in two of the three trial months
 #Let’s have a look at assessing this for several customers as well.
 #Step 1: Scaling Pre-Trial Control Customers to Match Trial Store Customers
 scaling_factor_for_control_cust = (
 pre_trial_data[(pre_trial_data["STORE_NBR"] == trial_store) & 
(pre_trial_data["YEARMON"].dt.to_timestamp() < pd.Timestamp("2019-02-01")]
 ["nCustomers"].sum() /
 pre_trial_data[(pre_trial_data["STORE_NBR"] == control_store) & 
(pre_trial_data["YEARMON"].dt.to_timestamp() < pd.Timestamp("2019-02-01")]
 ["nCustomers"].sum()
 )
 #Step 2: Apply the Scaling Factor
 measure_over_time_custs = pre_trial_data.copy()
 measure_over_time_custs.loc[measure_over_time_custs["STORE_NBR"] == control_store, 
"controlCustomers"] = (
 measure_over_time_custs["nCustomers"] * scaling_factor_for_control_cust
 )
 #Step 3: Assign Store Type
 measure_over_time_custs["Store_type"] = measure_over_time_custs["STORE_NBR"].apply(
 lambda
 lambda x: "Trial" if
 if x == trial_store else
 else ("Control" if
 stores")
 )
 if x == control_store else
 else "Other 
#Step 4: Calculate Percentage Difference
 percentage_diff = measure_over_time_custs[measure_over_time_custs["STORE_NBR"] == 
control_store][["YEARMON", "controlCustomers"]].merge(
 measure_over_time_custs[measure_over_time_custs["STORE_NBR"] == trial_store][["YEARMON", 
"nCustomers"]],
 on="YEARMON"
 )
 percentage_diff["percentageDiff"] = abs(percentage_diff["controlCustomers"] - 
percentage_diff["nCustomers"]) / percentage_diff["controlCustomers"]
 #Step 5: Calculate Standard Deviation
 std_dev = percentage_diff[percentage_diff["YEARMON"].dt.to_timestamp() < pd.Timestamp("2019
02-01")]["percentageDiff"].std()
degrees_of_freedom = 7  # 8 months in pre-trial, so df = 8-1
 #Step 6: Calculate 95th and 5th Percentile Confidence Intervals
 past_customers_controls_95 = measure_over_time_custs[measure_over_time_custs["Store_type"] == 
"Control"].copy()
 past_customers_controls_95["nCustomers"] = past_customers_controls_95["nCustomers"] * (1 + 
std_dev * 2)
 past_customers_controls_95["Store_type"] = "Control 95th % c
 % confidence interval"
 #Step 7: Calculate 5th Percentile
 past_customers_controls_5 = measure_over_time_custs[measure_over_time_custs["Store_type"] == 
"Control"].copy()
 past_customers_controls_5["nCustomers"] = past_customers_controls_5["nCustomers"] * (1 - 
std_dev * 2)
 past_customers_controls_5["Store_type"] = "Control 5th % c
 % confidence interval"
 # Step 8: Combine the Data
 trial_assessment = pd.concat([measure_over_time_custs, past_customers_controls_95, 
past_customers_controls_5])
 # Convert YEARMON to datetime format
 trial_assessment["TransactionMonth"] = trial_assessment["YEARMON"].dt.to_timestamp()
 # Filter for plotting
 plot_data = trial_assessment[(trial_assessment["YEARMON"].dt.to_timestamp() > 
pd.Timestamp("2018-06-01")) &
 (trial_assessment["YEARMON"].dt.to_timestamp() < 
pd.Timestamp("2019-05-01"))]
 plt.figure(figsize=(12, 6))
 sns.lineplot(data=plot_data, x="TransactionMonth", y="nCustomers", hue="Store_type", 
style="Store_type")
 # Highlight trial period (Feb 2019 - April 2019)
 plt.axvspan(pd.Timestamp("2019-02-01"), pd.Timestamp("2019-04-30"), color="gray", alpha=0.2)
 plt.xlabel("Month of Operation")
 plt.ylabel("Total Number of Customers")
 plt.title("Total Number of Customers by Month")
 plt.legend(title="Store Type")
 plt.show()
 #TRIAL STORE 86:
 #Step 1: Visualizing Sales Trends Before the Trial: We need to check if the trial store (86) 
and its control store (155) have similar trends in total sales before the trial period.
 # Define trial and control stores
 trial_store = 86
 control_store = 155
 # Convert YEARMONTH to datetime format for plotting
 pre_trial_data["TransactionMonth"] = pre_trial_data["YEARMON"].dt.to_timestamp()
 # Create a new column to categorize stores
 pre_trial_data["Store_type"] = pre_trial_data["STORE_NBR"].apply(
 lambda
 lambda x: "Trial" if
 if x == trial_store else
 else ("Control" if
 stores")
 )
 if x == control_store else
 else "Other 
# Aggregate total sales by year-month and store type
 past_sales = pre_trial_data.groupby(["TransactionMonth", "Store_type"])
 ["totSales"].mean().reset_index()
# Filter data to include only the pre-trial period
 past_sales = past_sales[past_sales["TransactionMonth"] < "2019-03-01"]
 # Plot total sales trends
 plt.figure(figsize=(12, 6))
 sns.lineplot(data=past_sales, x="TransactionMonth", y="totSales", hue="Store_type", 
style="Store_type")
 plt.xlabel("Month of Operation")
 plt.ylabel("Total Sales")
 plt.title("Total Sales by Month (Trial vs Control)")
 plt.legend(title="Store Type")
 plt.xticks(rotation=45)
 plt.show()
 #Step 2: Visualizing Number of Customers Trends
 # Aggregate total customers by year-month and store type
 past_customers = pre_trial_data.groupby(["TransactionMonth", "Store_type"])
 ["nCustomers"].mean().reset_index()
 # Filter pre-trial period
 past_customers = past_customers[past_customers["TransactionMonth"] < "2019-03-01"]
 # Plot the number of customers trends
 plt.figure(figsize=(12, 6))
 sns.lineplot(data=past_customers, x="TransactionMonth", y="nCustomers", hue="Store_type", 
style="Store_type")
 plt.xlabel("Month of Operation")
 plt.ylabel("Total Number of Customers")
 plt.title("Total Number of Customers by Month (Trial vs Control)")
 plt.legend(title="Store Type")
 plt.xticks(rotation=45)
 plt.show()
 #Step 3: Scaling Pre-Trial Control Sales to Match Trial Store: We compute a scaling factor to 
Adjust the control store sales to match the trial store.
 # Compute scaling factor
 scaling_factor_sales = (
 pre_trial_data[(pre_trial_data["STORE_NBR"] == trial_store) & 
((pre_trial_data["YEARMON"].dt.to_timestamp() < pd.Timestamp("2019-02-01")))]
 ["totSales"].sum() /
 pre_trial_data[(pre_trial_data["STORE_NBR"] == control_store) & 
((pre_trial_data["YEARMON"].dt.to_timestamp() < pd.Timestamp("2019-02-01")))]
 ["totSales"].sum()
 )
 # Apply scaling factor
 pre_trial_data.loc[pre_trial_data["STORE_NBR"] == control_store, "scaledSales"] = (
 pre_trial_data["totSales"] * scaling_factor_sales
 )
 # Step 4: Calculating Percentage Difference Between Scaled Control and Trial Sales We compute 
The percentage difference between the scaled control sales and trial sales.
 # Merge trial and scaled control sales on YEARMONTH
 percentage_diff_sales = pre_trial_data[pre_trial_data["STORE_NBR"] == control_store]
 [["TransactionMonth", "scaledSales"]].merge(
 pre_trial_data[pre_trial_data["STORE_NBR"] == trial_store][["TransactionMonth", 
"totSales"]],
 on="TransactionMonth"
 )
 # Calculate percentage difference
 percentage_diff_sales["percentageDiff"] = abs(percentage_diff_sales["scaledSales"] - 
percentage_diff_sales["totSales"]) / percentage_diff_sales["scaledSales"]
 #Step 5: Computing Standard Deviation for Pre-Trial Period,This is used to determine confidence 
intervals
 # Compute standard deviation of percentage differences in pre-trial period
 std_dev_sales = percentage_diff_sales[percentage_diff_sales["TransactionMonth"] < "2019-02-01"]
 ["percentageDiff"].std()
 degrees_of_freedom = 7  # Since we have 8 months in pre-trial period, df = 8-1
 #Step 6: Computing Confidence Intervals
 #We calculate 95th and 5th percentile confidence intervals.
 # Compute upper and lower confidence intervals
 past_sales_controls_95 = past_sales[past_sales["Store_type"] == "Control"].copy()
 past_sales_controls_5 = past_sales[past_sales["Store_type"] == "Control"].copy()
 past_sales_controls_95["totSales"] *= (1 + std_dev_sales * 2)
 past_sales_controls_95["Store_type"] = "Control 95% Confidence Interval"
 past_sales_controls_5["totSales"] *= (1 - std_dev_sales * 2)
 past_sales_controls_5["Store_type"] = "Control 5% Confidence Interval"
 trial_assessment = pd.concat([past_sales, past_sales_controls_95, past_sales_controls_5])
 #Step 7: Plotting Sales with Confidence Intervals
 # Plot total sales with confidence intervals
 plt.figure(figsize=(12, 6))
 sns.lineplot(data=trial_assessment, x="TransactionMonth", y="totSales", hue="Store_type", 
style="Store_type")
 # Highlight trial period
 plt.axvspan(pd.to_datetime("2019-02-01"), pd.to_datetime("2019-04-30"), color="gray", 
alpha=0.3)
 plt.xlabel("Month of Operation")
 plt.ylabel("Total Sales")
 plt.title("Total Sales by Month (With Confidence Intervals)")
 plt.legend(title="Store Type")
 plt.xticks(rotation=45)
 plt.show()
 #The results show that the trial in store 86 is not significantly different to its control 
store in the trial period
 #as the trial store performance lies inside the 5% to 95% confidence interval of the control 
store in two of the
 #three trial months.
 #Let’s look at assessing this for several customers as well.
 #Let’s have a look at assessing this for the number of customers as well
 #Step 1: Scaling Pre-Trial Control Customers to Match Trial Store Customers
 scaling_factor_for_control_cust = (
 pre_trial_data[(pre_trial_data["STORE_NBR"] == trial_store) & 
(pre_trial_data["YEARMON"].dt.to_timestamp() < pd.Timestamp("2019-02-01")]
 ["nCustomers"].sum() /
 pre_trial_data[(pre_trial_data["STORE_NBR"] == control_store) & 
(pre_trial_data["YEARMON"].dt.to_timestamp() < pd.Timestamp("2019-02-01")]
 ["nCustomers"].sum()
 )
 #Step 2: Apply the Scaling Factor
 measure_over_time_custs = pre_trial_data.copy()
 measure_over_time_custs.loc[measure_over_time_custs["STORE_NBR"] == control_store, 
"controlCustomers"] = (
 measure_over_time_custs["nCustomers"] * scaling_factor_for_control_cust
 )
 #Step 3: Assign Store Type
 measure_over_time_custs["Store_type"] = measure_over_time_custs["STORE_NBR"].apply(
 lambda
 lambda x: "Trial" if
 if x == trial_store else
 else ("Control" if
 if x == control_store else
 else "Other 
stores")
)
 #Step 4: Calculate Percentage Difference
 percentage_diff = measure_over_time_custs[measure_over_time_custs["STORE_NBR"] == 
control_store][["YEARMON", "controlCustomers"]].merge(
 measure_over_time_custs[measure_over_time_custs["STORE_NBR"] == trial_store][["YEARMON", 
"nCustomers"]],
 on="YEARMON"
 )
 percentage_diff["percentageDiff"] = abs(percentage_diff["controlCustomers"] - 
percentage_diff["nCustomers"]) / percentage_diff["controlCustomers"]
 #Step 5: Calculate Standard Deviation
 std_dev = percentage_diff[percentage_diff["YEARMON"].dt.to_timestamp() < pd.Timestamp("2019
02-01")]["percentageDiff"].std()
 degrees_of_freedom = 7  # 8 months in pre-trial, so df = 8-1
 #Step 6: Calculate 95th and 5th Percentile Confidence Intervals
 past_customers_controls_95 = measure_over_time_custs[measure_over_time_custs["Store_type"] == 
"Control"].copy()
 past_customers_controls_95["nCustomers"] = past_customers_controls_95["nCustomers"] * (1 + 
std_dev * 2)
 past_customers_controls_95["Store_type"] = "Control 95th % c
 % confidence interval"
 #Step 7: Calculate 5th Percentile
 past_customers_controls_5 = measure_over_time_custs[measure_over_time_custs["Store_type"] == 
"Control"].copy()
 past_customers_controls_5["nCustomers"] = past_customers_controls_5["nCustomers"] * (1 - 
std_dev * 2)
 past_customers_controls_5["Store_type"] = "Control 5th % c
 % confidence interval"
 # Step 8: Combine the Data
 trial_assessment = pd.concat([measure_over_time_custs, past_customers_controls_95, 
past_customers_controls_5])
 # Convert YEARMON to datetime format
 trial_assessment["TransactionMonth"] = trial_assessment["YEARMON"].dt.to_timestamp()
 # Filter for plotting
 plot_data = trial_assessment[(trial_assessment["YEARMON"].dt.to_timestamp() > 
pd.Timestamp("2018-06-01")) &
 (trial_assessment["YEARMON"].dt.to_timestamp() < 
pd.Timestamp("2019-05-01")]
 plt.figure(figsize=(12, 6))
 sns.lineplot(data=plot_data, x="TransactionMonth", y="nCustomers", hue="Store_type", 
style="Store_type")
 # Highlight trial period (Feb 2019 - April 2019)
 plt.axvspan(pd.Timestamp("2019-02-01"), pd.Timestamp("2019-04-30"), color="gray", alpha=0.2)
 plt.xlabel("Month of Operation")
 plt.ylabel("Total Number of Customers")
 plt.title("Total Number of Customers by Month")
 plt.legend(title="Store Type")
 plt.show()
 #It looks like the number of customers is significantly higher in all of the three months. This 
seems to
 #suggest that the trial had a significant impact on increasing the number of customers in the trial 
store 86 but
 #as we saw, sales were not significantly higher. We should check with the Category Manager if 
there were
 #special deals in the trial store that may have resulted in lower prices, impacting the 
results.
#TRIAL STORE 88:
 #Step 1: Visualizing Sales Trends Before the Trial: We need to check if the trial store (88) 
and its control store (237) have similar trends in total sales before the trial period.
 # Define trial and control stores
 trial_store = 88
 control_store = 237
 # Convert YEARMONTH to datetime format for plotting
 pre_trial_data["TransactionMonth"] = pre_trial_data["YEARMON"].dt.to_timestamp()
 # Create a new column to categorize stores
 pre_trial_data["Store_type"] = pre_trial_data["STORE_NBR"].apply(
 lambda
 lambda x: "Trial" if
 if x == trial_store else
 else ("Control" if
 stores")
 )
 if x == control_store else
 else "Other 
# Aggregate total sales by year-month and store type
 past_sales = pre_trial_data.groupby(["TransactionMonth", "Store_type"])
 ["totSales"].mean().reset_index()
 # Filter data to include only the pre-trial period
 past_sales = past_sales[past_sales["TransactionMonth"] < "2019-03-01"]
 # Plot total sales trends
 plt.figure(figsize=(12, 6))
 sns.lineplot(data=past_sales, x="TransactionMonth", y="totSales", hue="Store_type", 
style="Store_type")
 plt.xlabel("Month of Operation")
 plt.ylabel("Total Sales")
 plt.title("Total Sales by Month (Trial vs Control)")
 plt.legend(title="Store Type")
 plt.xticks(rotation=45)
 plt.show()
 #Step 2: Visualizing Number of Customers Trends
 # Aggregate total customers by year-month and store type
 past_customers = pre_trial_data.groupby(["TransactionMonth", "Store_type"])
 ["nCustomers"].mean().reset_index()
 # Filter pre-trial period
 past_customers = past_customers[past_customers["TransactionMonth"] < "2019-03-01"]
 # Plot the number of customers trends
 plt.figure(figsize=(12, 6))
 sns.lineplot(data=past_customers, x="TransactionMonth", y="nCustomers", hue="Store_type", 
style="Store_type")
 plt.xlabel("Month of Operation")
 plt.ylabel("Total Number of Customers")
 plt.title("Total Number of Customers by Month (Trial vs Control)")
 plt.legend(title="Store Type")
 plt.xticks(rotation=45)
 plt.show()
 #Step 3: Scaling Pre-Trial Control Sales to Match Trial Store: We compute a scaling factor to 
adjust the control store sales to match the trial store.
 # Compute scaling factor
 scaling_factor_sales = (
 pre_trial_data[(pre_trial_data["STORE_NBR"] == trial_store) & 
((pre_trial_data["YEARMON"].dt.to_timestamp() < pd.Timestamp("2019-02-01"))]
 ["totSales"].sum() /
 pre_trial_data[(pre_trial_data["STORE_NBR"] == control_store) & 
((pre_trial_data["YEARMON"].dt.to_timestamp() < pd.Timestamp("2019-02-01"))]
 ["totSales"].sum()
 )
 # Apply scaling factor
 pre_trial_data.loc[pre_trial_data["STORE_NBR"] == control_store, "scaledSales"] = (
 pre_trial_data["totSales"] * scaling_factor_sales
 )
 # Step 4: Calculating Percentage Difference Between Scaled Control and Trial Sales. We compute 
The percentage difference between the scaled control sales and trial sales.
 # Merge trial and scaled control sales on YEARMONTH
 percentage_diff_sales = pre_trial_data[pre_trial_data["STORE_NBR"] == control_store]
 [["TransactionMonth", "scaledSales"]].merge(
 pre_trial_data[pre_trial_data["STORE_NBR"] == trial_store][["TransactionMonth", 
"totSales"]],
 on="TransactionMonth"
 )
 # Calculate percentage difference
 percentage_diff_sales["percentageDiff"] = abs(percentage_diff_sales["scaledSales"] - 
percentage_diff_sales["totSales"]) / percentage_diff_sales["scaledSales"]
 #Step 5: Computing Standard Deviation for Pre-Trial Period. This is used to determine confidence 
intervals
 # Compute standard deviation of percentage differences in pre-trial period
 std_dev_sales = percentage_diff_sales[percentage_diff_sales["TransactionMonth"] < "2019-02-01"]
 ["percentageDiff"].std()
 degrees_of_freedom = 7  # Since we have 8 months in pre-trial period, df = 8-1
 #Step 6: Computing Confidence Intervals
 #We calculate 95th and 5th percentile confidence intervals.
 # Compute upper and lower confidence intervals
 past_sales_controls_95 = past_sales[past_sales["Store_type"] == "Control"].copy()
 past_sales_controls_5 = past_sales[past_sales["Store_type"] == "Control"].copy()
 past_sales_controls_95["totSales"] *= (1 + std_dev_sales * 2)
 past_sales_controls_95["Store_type"] = "Control 95% Confidence Interval"
 past_sales_controls_5["totSales"] *= (1 - std_dev_sales * 2)
 past_sales_controls_5["Store_type"] = "Control 5% Confidence Interval"
 trial_assessment = pd.concat([past_sales, past_sales_controls_95, past_sales_controls_5])
 #Step 7: Plotting Sales with Confidence Intervals
 # Plot total sales with confidence intervals
 plt.figure(figsize=(12, 6))
 sns.lineplot(data=trial_assessment, x="TransactionMonth", y="totSales", hue="Store_type", 
style="Store_type")
 # Highlight trial period
 plt.axvspan(pd.to_datetime("2019-02-01"), pd.to_datetime("2019-04-30"), color="gray", 
alpha=0.3)
 plt.xlabel("Month of Operation")
 plt.ylabel("Total Sales")
 plt.title("Total Sales by Month (With Confidence Intervals)")
 plt.legend(title="Store Type")
 plt.xticks(rotation=45)
 plt.show()
 #The results show that the trial in store 88 is significantly different to its control store in 
The trial period as the trial store performance lies outside of the 5% to 95% confidence 
interval 
#of the control store in two of the three trial months.
 #Let’s have a look at assessing this for the number of customers as well
#Step 1: Scaling Pre-Trial Control Customers to Match Trial Store Customers
 scaling_factor_for_control_cust = (
    pre_trial_data[(pre_trial_data["STORE_NBR"] == trial_store) & 
(pre_trial_data["YEARMON"].dt.to_timestamp() < pd.Timestamp("2019-02-01")]
 ["nCustomers"].sum() /
    pre_trial_data[(pre_trial_data["STORE_NBR"] == control_store) & 
(pre_trial_data["YEARMON"].dt.to_timestamp() < pd.Timestamp("2019-02-01")]
 ["nCustomers"].sum()
 )
 #Step 2: Apply the Scaling Factor
 measure_over_time_custs = pre_trial_data.copy()
 measure_over_time_custs.loc[measure_over_time_custs["STORE_NBR"] == control_store, 
"controlCustomers"] = (
    measure_over_time_custs["nCustomers"] * scaling_factor_for_control_cust
 )
 #Step 3: Assign Store Type
 measure_over_time_custs["Store_type"] = measure_over_time_custs["STORE_NBR"].apply(
    lambda lambda x: "Trial" if if x == trial_store else else ("Control" if if x == control_store else else "Other 
stores")
 )
 #Step 4: Calculate Percentage Difference
 percentage_diff = measure_over_time_custs[measure_over_time_custs["STORE_NBR"] == 
control_store][["YEARMON", "controlCustomers"]].merge(
    measure_over_time_custs[measure_over_time_custs["STORE_NBR"] == trial_store][["YEARMON", 
"nCustomers"]],
    on="YEARMON"
 )
 percentage_diff["percentageDiff"] = abs(percentage_diff["controlCustomers"] - 
percentage_diff["nCustomers"]) / percentage_diff["controlCustomers"]
 #Step 5: Calculate Standard Deviation
 std_dev = percentage_diff[percentage_diff["YEARMON"].dt.to_timestamp() < pd.Timestamp("2019
02-01")]["percentageDiff"].std()
 degrees_of_freedom = 7  # 8 months in pre-trial, so df = 8-1
 #Step 6: Calculate 95th and 5th Percentile Confidence Intervals
 past_customers_controls_95 = measure_over_time_custs[measure_over_time_custs["Store_type"] == 
"Control"].copy()
 past_customers_controls_95["nCustomers"] = past_customers_controls_95["nCustomers"] * (1 + 
std_dev * 2)
 past_customers_controls_95["Store_type"] = "Control 95th % c % confidence interval"
 #Step 7: Calculate 5th Percentile
 past_customers_controls_5 = measure_over_time_custs[measure_over_time_custs["Store_type"] == 
"Control"].copy()
 past_customers_controls_5["nCustomers"] = past_customers_controls_5["nCustomers"] * (1 - 
std_dev * 2)
 past_customers_controls_5["Store_type"] = "Control 5th % c % confidence interval"
 # Step 8: Combine the Data
 trial_assessment = pd.concat([measure_over_time_custs, past_customers_controls_95, 
past_customers_controls_5])
 # Convert YEARMON to datetime format
 trial_assessment["TransactionMonth"] = trial_assessment["YEARMON"].dt.to_timestamp()
 # Filter for plotting
 plot_data = trial_assessment[(trial_assessment["YEARMON"].dt.to_timestamp() > 
pd.Timestamp("2018-06-01")) &
                             (trial_assessment["YEARMON"].dt.to_timestamp() < 
pd.Timestamp("2019-05-01")]
plt.figure(figsize=(12, 6))
 sns.lineplot(data=plot_data, x="TransactionMonth", y="nCustomers", hue="Store_type", 
style="Store_type")
 # Highlight trial period (Feb 2019 - April 2019)
 plt.axvspan(pd.Timestamp("2019-02-01"), pd.Timestamp("2019-04-30"), color="gray", alpha=0.2)
 plt.xlabel("Month of Operation")
 plt.ylabel("Total Number of Customers")
 plt.title("Total Number of Customers by Month")
 plt.legend(title="Store Type")
 plt.show()
 # The total number of customers in the trial period for the trial store is significantly higher than 
The control store for two out of three months, which indicates a positive trial effect.
 #Conclusion
 # We’ve found control stores 233, 155, 237 for trial stores 77, 86 and 88 respectively.
 # The results for trial stores 77 and 88 during the trial period show a significant difference 
In at least two of the three trial months but this is not the case for trial store 86.
 #  We can check with the client if the implementation of the trial was different in the trial store 
86, but overall, the trial shows a significant increase in sales.
