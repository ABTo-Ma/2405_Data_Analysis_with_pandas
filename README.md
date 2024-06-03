# 2405_Data_Analysis_with_pandas
As part of a learning exercise at WBS-Coding School, a fictitious case study was created. An online marketplace specializing in high-end technology products based in Spain wants to decide wether or not to apply discounts.
By using pandas, business question from the make up company where answered.The results will be presented in a business presentation. 
## About the project: resources, tools and colaboratores
# Resources: 
4 files: orders.cvs, orderlines.cvs, products.cvs and brands.cvs provided by the company. 
orders.cvs —Every row in this file represents an order.
order_id – a unique identifier for each order
created_date – a timestamp for when the order was created
total_paid – the total amount paid by the customer for this order, in euros
state
“Shopping basket” – products have been placed in the shopping basket
“Place Order” – the order has been placed, but is awaiting shipment details
“Pending” – the order is awaiting payment confirmation
“Completed” – the order has been placed and paid, and the transaction is completed
“Cancelled” – the order has been cancelled and the payment returned to the customer
orderlines.cvs – Every row represents each one of the different products involved in an order.
id – a unique identifier for each row in this file
id_order – corresponds to orders.order_id
product_id – an old identifier for each product, nowadays not in use
product_quantity – how many units of that product were purchased on that order
sku – stock keeping unit: a unique identifier for each product. The first 3 letters refer to the brad.
unit_price – the unitary price (in euros) of each product at the moment of placing that order
date – timestamp for the processing of that product
products.cvs 
sku
name – product name
desc – product description
price – base price of the product, in euros
promo_price – promotional price, in euros
in_stock – whether or not the product was in stock at the moment of the data extraction
type – a numerical code for product type
brands.cvs
short – the 3-character code by which the brand can be identified in the first 3 characters of products.sku
long – brand name
# Tools:
Python
# Colaboratores:
Teams of 3 people
# Time span:
32 hours
## About the project: Challenges  
The extensive amount of provided data was highly inconsistent with errors in most of the files.
Mastering the analysis would require good teamwork.
To accurately identify key aspects for decision-making within realistic financial expectations, conducting research in the sales trends was crucial.
The insights gained from the analysis had to be condensed into a 5-minute business presentation.
## About the project: Result
A successful analysis of the given data was conducted. Clear recommendations for business decisions, as well as future insights, were provided. Good teamwork was facilitated through communication and clear division of tasks. Expectations were met.
## Codes
# Data Cleaning:
# import pandas
import pandas as pd
# import seaborn
import seaborn as sns
# import matplot 
import matplotlib.pyplot as plt
# import numpy
import numpy as np
import warnings
warnings.filterwarnings("ignore")

# orders.csv
url = "https://drive.google.com/file/d/1Vu0q91qZw6lqhIqbjoXYvYAQTmVHh6uZ/view?usp=sharing"
path = "https://drive.google.com/uc?export=download&id="+url.split("/")[-2]
orders = pd.read_csv(path, parse_dates=[‘created_date'])

# orderlines.csv
url = "https://drive.google.com/file/d/1FYhN_2AzTBFuWcfHaRuKcuCE6CWXsWtG/view?usp=sharing"
path = "https://drive.google.com/uc?export=download&id="+url.split("/")[-2]
orderlines = pd.read_csv(path)

# products.cvs
url = "https://drive.google.com/file/d/1afxwDXfl-7cQ_qLwyDitfcCx3u7WMvkU/view?usp=sharing"
path = "https://drive.google.com/uc?export=download&id="+url.split("/")[-2]
products = pd.read_csv(path)

# brands.cvs
url = "https://drive.google.com/file/d/1m1ThDDIYRTTii-rqM5SEQjJ8McidJskD/view?usp=sharing"
path = "https://drive.google.com/uc?export=download&id="+url.split("/")[-2]
brands = pd.read_csv(path)

# Make copies of the data orders, orderlines, products and brands
orders_df = orders.copy()
orderlines_df = orderlines.copy()
products_df = products.copy()
brands_df = brands.copy()

# Get the information on the data
orders_df.info()
orderlines_df.info()
products_df.info()
brands_df.info()

# Inspecting for duplicates
# orders_df
orders_df.duplicated().sum()
# orderlines_df
orderlines_df.duplicated().sum()
# products_df
products_df.duplicated().sum()
# brands_df
brands_df.duplicated().sum()

#Delete the missing entries
#orders_df
orders_df.dropna(subset=['total_paid'], inplace=True)
orders_df.reset_index()
#products_df
products_df.dropna(subset=['desc'], inplace=True)
products_df.dropna(subset=['price'], inplace=True)
products_df.dropna(subset=['type'], inplace=True)
products_df.reset_index()

#Delete duplicates
#products_df
products_df.drop_duplicates(inplace=True)
products_df.reset_index()

#Transform Datatypes for dates
def change_obj_to_datetime64_n_columns(df, *df_columns):
    for column in df_columns:
        df[column] = pd.to_datetime(df[column])
    return df

#orders_df
change_obj_to_datetime64_n_columns(orders_df,’created_date’)
orders_df.info()
#orderlines_df
change_obj_to_datetime64_n_columns(orderlines_df,’date’)
orderlines_df.info()

#Trasform Datatypes for currency/payments into float
def remove_separators_convert_to_float(number):
    # Check for commas
    number = number.str.replace(',', '.')
    # Remove separators for thousands
    number = number.str.replace(r'\.(?=\d{3}\.)', '', regex=True)
    # Remove additional separators when 3 digits after decimal point
    number = number.str.replace(r'^\.(?=\d{3})', '', regex=True)
    # Convert to float
    number = number.astype(float)  
    return number
#orderlines_df
orderlines_df['unit_price'] = remove_separators_convert_to_float(orderlines_df['unit_price'])
#products_df
products_df['price'] = remove_separators_convert_to_float(products_df[‘price'])
products_df['promo_price'] = remove_separators_convert_to_float(products_df[‘promo_price’])

#Correct the promo_price column
for n, (Number_1, Number_2) in enumerate(zip(products_df['price'],products_df['promo_price'])):
  if Number_2/Number_1 > 1:
    products_df.at[n, 'promo_price'] = (Number_2)/10
products_df

#Create a new column in the products_df to show the given discount
products_df['Discount [%]'] = round((1-(products_df['promo_price']/products_df['price']))*100,2)
products_df

#Delete irrelevant columns
#orderlines_df
orderlines_df = orderlines_df.drop(‘product_id’, axis= 1, inplace=True)
 
#Save the cleaned files
#orders_df
orders_df.to_csv(orders_cl_1.csv, date_format=‘%Y-%m-%d’)
#orderlines_df
orderlines_df.to_csv(orderlines_cl_1.csv, date_format=‘%Y-%m-%d’)
#products_df
products_df.to_csv(products_cl_1.csv, date_format=‘%Y-%m-%d’)
#brands_df
brands_df.to_csv(brands_cl_1.csv, date_format=‘%Y-%m-%d’)
## Codes
# Analysis:
# import pandas
import pandas as pd

# orders_cl_1.csv
url = "https://drive.google.com/file/d/1Vu0q91qZw6lqhIqbjoXYvYAQTmVHh6uZ/view?usp=sharing"
path = "https://drive.google.com/uc?export=download&id="+url.split("/")[-2]
orders_cl_1 = pd.read_csv(path, parse_dates=[‘created_date'])

# orderlines_cl_1.csv
url = "https://drive.google.com/file/d/1FYhN_2AzTBFuWcfHaRuKcuCE6CWXsWtG/view?usp=sharing"
path = "https://drive.google.com/uc?export=download&id="+url.split("/")[-2]
orderlines_cl_1 = pd.read_csv(path, parse_dates=['date'])

# products_cl_1.cvs
url = "https://drive.google.com/file/d/1afxwDXfl-7cQ_qLwyDitfcCx3u7WMvkU/view?usp=sharing"
path = "https://drive.google.com/uc?export=download&id="+url.split("/")[-2]
products_cl_1 = pd.read_csv(path)

# brands_cl_1.cvs
url = "https://drive.google.com/file/d/1m1ThDDIYRTTii-rqM5SEQjJ8McidJskD/view?usp=sharing"
path = "https://drive.google.com/uc?export=download&id="+url.split("/")[-2]
brands_cl_1 = pd.read_csv(path)

# Make copies of the data orders, orderlines, products and brands
orders_df = orders_cl_1.copy()
orderlines_df = orderlines_cl_1.copy()
products_df = products_cl_1.copy()
brands_df = brands_cl_1.copy()

# Get the information on the data
orders_df.info()
orderlines_df.info()
products_df.info()
brands_df.info()

#Scope of the data
#orders_df
orders_df_order_last = orders_df["created_date"].max().strftime("%A, %d %b %y")
orders_df_order_first = orders_df["created_date"].min().strftime("%A, %d %b %y")
print('The last order was on: ',orders_df_order_last)
print('The first order was on: ‘,orders_df_order_first)

#Group by year and month
#orders_df 
def df_by_year_and_month(df,date_column):
  single_df_year_month={}
  for year in range(df[date_column].dt.year.min(), df[date_column].dt.year.max() + 1):
    for month in range(1,13):
      filtered_df = df.loc[(df[date_column].dt.year == year) & (df[date_column].dt.month == month)]
      single_df_year_month[(year, month)] = filtered_df
  return single_df_year_month

orders_df_01_17 = df_by_year_and_month(orders_df, 'created_date')[(2017, 1)]
orders_df_02_17 = df_by_year_and_month(orders_df, 'created_date')[(2017, 2)]
orders_df_03_17 = df_by_year_and_month(orders_df, 'created_date')[(2017, 3)]
orders_df_04_17 = df_by_year_and_month(orders_df, 'created_date')[(2017, 4)]
orders_df_05_17 = df_by_year_and_month(orders_df, 'created_date')[(2017, 5)]
orders_df_06_17 = df_by_year_and_month(orders_df, 'created_date')[(2017, 6)]
orders_df_07_17 = df_by_year_and_month(orders_df, 'created_date')[(2017, 7)]
orders_df_08_17 = df_by_year_and_month(orders_df, 'created_date')[(2017, 8)]
orders_df_09_17 = df_by_year_and_month(orders_df, 'created_date')[(2017, 9)]
orders_df_10_17 = df_by_year_and_month(orders_df, 'created_date')[(2017, 10)]
orders_df_11_17 = df_by_year_and_month(orders_df, 'created_date')[(2017, 11)]
orders_df_12_17 = df_by_year_and_month(orders_df, 'created_date')[(2017, 12)]
orders_df_01_18 = df_by_year_and_month(orders_df, 'created_date')[(2018, 1)]
orders_df_02_18 = df_by_year_and_month(orders_df, 'created_date')[(2018, 2)]
orders_df_03_18 = df_by_year_and_month(orders_df, 'created_date')[(2018, 3)]

#Orders in special dates

#Apple WWDC 2017 from 06.05.2017 to 09.05.2017
#orders_df
orders_apple_wwdc = sum((orders_df[‘created_date’] >= “2017-05-06”) & (orders_df['created_date'] <= “2017-05-06”))
#orderlines_df
orderlines_apple_wwdc = sum((orderlines_df['date'] >= "2017-05-06") & (orderlines_df['date'] <= "2017-05-06") & (orderlines_df['sku'].str.startswith('APP')))

#Apple Special Event 12.09.2017
#orders_df
orders_apple_special_event = sum(orders_df[‘created_date’] == “2017-09-12”)
#orderlines_df
orderlines_apple_special_event = sum((orderlines_df['date'] == "2017-09-12") & (orderlines_df[‘sku'].str.startswith('APP')))

#iPhone 8 22.09.2017
#orders_df
orders_apple_iPhone8 = sum(orders_df[‘created_date’] == “2017-09-22”)
#orderlines_df
orderlines_apple_iPhone8 = sum((orderlines_df['date'] == "2017-09-22") & (orderlines_df[‘sku'].str.startswith('APP')))

#iPhone X available in Spain 30.09.2017
#orders_df
orders_apple_iPhone8 = sum(orders_df[‘created_date’] == “2017-09-30”)
#orderlines_df
orderlines_apple_iPhone8 = sum((orderlines_df['date'] == "2017-09-30") & (orderlines_df[‘sku'].str.startswith('APP')))

#Black Friday 24.11.2017
orders_black_friday = sum(orders_df['created_date'] == "2017-11-24")
#Cyber Monday 27.11.2017
orders_cyber_monday = sum(orders_df['created_date'] == “2017-11-27")

products_df = products_cl.copy()
orderlines_df = orderlines_qu.copy()
products_orderlines_merge_df = orderlines_df.merge(products_df, how='left', left_on='sku', right_on='sku')
products_orderlines_merge_df.drop('product_id', axis= 1, inplace=True)
products_orderlines_merge_df['total_payment'] = products_orderlines_merge_df['price']*products_orderlines_merge_df['product_quantity']
products_orderlines_merge_df ['discount[%]'] =round( (1-(products_orderlines_merge_df['unit_price']/products_orderlines_merge_df['price']))*100,0)
products_orderlines_merge_df['discount?'] = products_orderlines_merge_df['discount[%]'].apply(lambda x: 1 if x != 0 else 0)
products_orderlines_merge_df
#Group by year and month
def df_by_year_and_month(df,date_column):
  single_df_year_month={}
  for year in range(df[date_column].dt.year.min(), df[date_column].dt.year.max() + 1):
    for month in range(1,13):
      filtered_df = df.loc[(df[date_column].dt.year == year) & (df[date_column].dt.month == month)]
      single_df_year_month[(year, month)] = filtered_df
  return single_df_year_month
#Apple WWDC (06.05-09.05.2017)
products_orderlines_merge_df_05_17 = df_by_year_and_month(products_orderlines_merge_df, 'date')[(2017, 5)]
#Apple Special Event(12.09.2017), iPhone 8 release (22.09.2017), iPhone X (30.09.2017)
products_orderlines_merge_df_09_17 = df_by_year_and_month(products_orderlines_merge_df, 'date')[(2017, 9)]
#Black Friday(24.11.2017) and Cyber Monday(27.11.2017)
products_orderlines_merge_df_11_17 = df_by_year_and_month(products_orderlines_merge_df, 'date')[(2017, 11)]
#Christmas(25.12.2017)
products_orderlines_merge_df_12_17 = df_by_year_and_month(products_orderlines_merge_df, 'date')[(2017, 12)]
#Reyes(06.01.2017/2018)
products_orderlines_merge_df_01_17 = df_by_year_and_month(products_orderlines_merge_df, 'date')[(2017, 1)]
products_orderlines_merge_df_01_18 = df_by_year_and_month(products_orderlines_merge_df, 'date')[(2018, 1)]
#Merge seasonal sales
seasonal = [products_orderlines_merge_df_05_17,products_orderlines_merge_df_09_17,products_orderlines_merge_df_11_17,products_orderlines_merge_df_12_17,products_orderlines_merge_df_01_17,products_orderlines_merge_df_01_18]
products_orderlines_seasonal_sales_df = pd.concat(seasonal, ignore_index=1)
products_orderlines_seasonal_sales_df['date_only'] = products_orderlines_seasonal_sales_df['date'].dt.date
sales = products_orderlines_seasonal_sales_df.groupby(['date_only'],as_index=0).sum(numeric_only = 1)
sales['date_only'] = pd.to_datetime(sales['date_only'])
#sales.info()
sales_01_17 = sales[(sales['date_only']>=pd.to_datetime('2017-01-01'))& (sales['date_only']<=pd.to_datetime('2017-01-31'))]
sales_05_17 = sales[(sales['date_only']>=pd.to_datetime('2017-05-01'))& (sales['date_only']<=pd.to_datetime('2017-05-31'))]
sales_09_17 = sales[(sales['date_only']>=pd.to_datetime('2017-09-01'))& (sales['date_only']<=pd.to_datetime('2017-09-30'))]
sales_11_17 = sales[(sales['date_only']>=pd.to_datetime('2017-11-01'))& (sales['date_only']<=pd.to_datetime('2017-11-30'))]
sales_12_17 = sales[(sales['date_only']>=pd.to_datetime('2017-12-01'))& (sales['date_only']<=pd.to_datetime('2017-12-31'))]
sales_01_18 = sales[(sales['date_only']>=pd.to_datetime('2018-01-01'))& (sales['date_only']<=pd.to_datetime('2018-01-31'))]
#Plot
plot_sales_01_17 = sns.lineplot(data=sales_01_17,
                         x='date_only',
                         y='total_payment',
                         palette = 'deep');
                         
plot_sales_05_17 = sns.lineplot(data=sales_05_17,
                         x='date_only',
                         y='total_payment',
                         palette = 'deep');
plot_sales_09_17 = sns.lineplot(data=sales_09_17,
                         x='date_only',
                         y='total_payment',
                         palette = 'deep');
plot_sales_11_17 = sns.lineplot(data=sales_11_17,
                         x='date_only',
                         y='total_payment',
                         palette = 'deep');
plot_sales_12_17 = sns.lineplot(data=sales_12_17,
                         x='date_only',
                         y='total_payment',
                         palette = 'deep');
plot_sales_01_18 = sns.lineplot(data=sales_01_18,
                         x='date_only',
                         y='total_payment',
                         palette = 'deep');
#x- and y-labels
plot_sales_01_18.set_xlabel('Date');
plot_sales_01_18.set_ylabel('Daily revenue [€]’);

products_orderlines_merge_df.sum(numeric_only=1)
sales_24_27_11_17 = sales[(sales['date_only']>=pd.to_datetime('2017-11-24'))& (sales['date_only']<=pd.to_datetime('2017-11-27'))]
sales_24_27_11_17.sum(numeric_only = 1)
#products_orderlines_merge_df.info()
Streu = products_orderlines_merge_df[(products_orderlines_merge_df['date']>=pd.to_datetime('2017-11-24'))& (products_orderlines_merge_df['date']<=pd.to_datetime('2017-11-27'))]
Streu
Streudiagram_1 = Streu.groupby(by=['discount[%]'], as_index=0).sum(numeric_only=1)
Streudiagram_1
def classify_discount(discount):
    if discount > 90:
        return '90'
    elif discount > 80:
        return '80'
    elif discount > 70:
        return '70'
    elif discount > 60:
        return '60'
    elif discount > 50:
        return '50'
    elif discount > 40:
        return '40'
    elif discount > 30:
        return '30'
    elif discount > 20:
        return '20'
    elif discount > 10:
        return '10'
    elif discount > 11:
        return '5'
    elif discount == 0:
        return '0'
    else:
        return 'DELETE'
#Streudiagram_1.info()
Streudiagram_1['Classification'] = Streudiagram_1['discount[%]'].apply(classify_discount)
Streudiagram_2 = Streudiagram_1.groupby(by=['Classification'], as_index=0).sum(numeric_only=1)
Streudiagram_2 = Streudiagram_2.drop([9])
Streudiagram_2 
plot_Streudiagram_2 = sns.barplot(data=Streudiagram_2,
                         x='Classification',
                         y='product_quantity',
                         palette = 'deep')
#x- and y-labels
plot_Streudiagram_2.set_xlabel('Discount range [%]');
plot_Streudiagram_2.set_ylabel('Products sold');

#Products with the highest orders
#orderlines_df
highest_orders_df = []
highest_orders_df = orderlines_df.groupby('sku')['product_quantity'].sum()
highest_orders_df = highest_orders_df.reset_index()
total_product_quantity = highest_orders_df['product_quantity'].sum()
print(total_product_quantity)
highest_orders_df.nlargest(10,’product_quantity',keep='all')

#Categorization by number of orders made
def classify_number_of_sales(sales):
    if sales > 2000:
        return 'Group 1: >2000'
    elif sales > 1000:
        return 'Group 2: 1001-2000'
    elif sales > 500:
        return 'Group 3: 501-1000’
    elif sales > 450:
        return 'Group 4: 451-500’
    elif sales > 400:
        return 'Group 5: 401-450’
    elif sales > 350:
        return 'Group 6: 351-400’
    elif sales > 300:
        return 'Group 7: 301-350’
    elif sales > 250:
        return 'Group 8: 251-300’
    elif sales > 200:
        return 'Group 9: 201-250’
    elif sales > 150:
        return 'Group 10: 151-200’
    elif sales > 100:
        return 'Group 11: 101-150’
    elif sales > 50:
        return 'Group 12: 51-100'
    else:
        return 'Group 13: <=50’
highest_orders_df['Classification'] =highest_orders_df['product_quantity'].apply(classify_number_of_sales)
orders_df_class=highest_orders_df
orders_df_class.groupby('Classification').count()
orders_df_class_stats=orders_df_class.groupby('Classification').count()
orders_df_class_stats['f']=orders_df_class_stats['sku']/6798
orders_df_class_stats

#Grouping by brands
orderlines_df['short'] = orderlines_df['sku'].str[:3]
#Create a new data frame to explore the sales by brands
brands_sales_df = orderlines_df.merge(brands_df, how='left', left_on='short', right_on='short')
brands_sales_df
sales_by_brand_df = brands_sales_df.groupby('short')['product_id'].count().reset_index()
sales_by_brand_df.rename(columns={'product_id': 'product_count'}, inplace=True)
sales_by_brand_df['f']=sales_by_brand_df['product_count']/(sales_by_brand_df['product_count'].sum())
sales_by_brand_df = sales_by_brand_df.merge(brands_df, how='left', left_on='short', right_on='short')
# Brands with the most sales
sales_by_brand_df.nlargest(10,’f’,keep='all')

# The brand Pack does nos exist
Pack = products_orderlines_merge_df
Pack ['short'] = Pack['sku'].str[:3]
Pack ['Pack?'] = Pack['short'].apply(lambda x: 0 if x != 'PAC' else 1)
Pack_is = Pack['Pack?'].apply(lambda x: Pack['name'] if x != 0 else 'NaN')
Pack['Pack_is'] = np.where(Pack['Pack?'] == 1, Pack['name'], np.nan)
Pack_is = Pack.dropna(subset=['Pack_is'])
Pack_is


# For visualization, graphs were made with condensed information
<img width="586" alt="Screenshot 2024-04-30 at 16 54 33" src="https://github.com/ABTo-Ma/Analysis.Visualization_Brazilian_Market/assets/168551372/69e1ad74-48a4-4bbc-bb57-04d1eafdd8bd">
