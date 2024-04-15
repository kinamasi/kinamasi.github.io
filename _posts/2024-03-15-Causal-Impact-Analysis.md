---

layout:post
title: Causal Impact Analysis
image:"/posts/coffee_python.jpg"
tags:[Pyhton,Coffee]

---



# Casual Impact Analysis
""" 
Task: Understand and measure the effect of the delivery membership campaign
overall know what the sales are for the membership
"""

# Import required packages
from causalimpact import CausalImpact
import pandas as pd


# Import & Create data
transactions = pd.read_excel("data/grocery_database.xlsx",sheet_name= "transactions")
campaign_data = pd.read_excel("data/grocery_database.xlsx",sheet_name= "campaign_data")

# aggregate transactions data to customer, data level
customer_daily_sales = transactions.groupby(["customer_id","transaction_date"])["sales_cost"].sum().reset_index()

# Merge on signup flag
customer_daily_sales = pd.merge(customer_daily_sales,campaign_data, how="inner",on="customer_id")


# Pivot data to aggregate daily sales by signup group
"""
since the data is at customer and date level, for causal impact, need the data to be at 
date level (sales over time) and the mean sales per customer for each of the signed up and
did not groups into columns
"""
causal_impact_df = customer_daily_sales.pivot_table(index = "transaction_date",
                                                    columns = "signup_flag",
                                                    values = "sales_cost",
                                                    aggfunc = "mean")

# provide a frequency for DateTimeIndex (avoid a warning message)
causal_impact_df.index.freq = "D"

# for causal impact we need the impacted group in the first column
causal_impact_df = causal_impact_df[[1,0]]

#rename columns to something more meaningful
causal_impact_df.columns=["member","non_member"]

###############################################################################
# Apply Causal Impact
###############################################################################
"""
specify the start and end dates of the pre period and post period (6 months data) The membership 
went live on the 1st of July. Pre period will run from the 1st of April and 30th June.
Post period will run from the 1st of July to Sept 30th
"""
pre_period = ["2020-04-01", "2020-06-30"]
post_period = ["2020-07-01", "2020-09-30"]

ci = CausalImpact(causal_impact_df, pre_period, post_period)

# Plot impact
ci.plot()

# Get summary statistics and report
print(ci.summary())
print(ci.summary(output = "report"))
