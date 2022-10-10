# -You-Are-What-You-Eat-Customer-Segmentation

<p align="center">
  <img src=""
 />
</p>



# 01. Business Task
In this case we are asked by a grocery company to help them to a segmentation of thier customer base. Thy believe there might be some changes in terms of lifestyles and customer behaviours but they do not exactly.

Full code is available here.

# 02. Data
The data consists of two databases: a database of transactions by customers (transactions)

<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/194894351-de12f2bf-7533-4df3-87ea-d98514c135e9.png"
 />
</p>

and one of the product categories (product areas) 

<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/194894515-c2048cdb-8cdc-4cea-9f5f-e9241b058f97.png"
 />
</p>

# 03. Data Wrangling
First of all, we have to merge the two databases into one

```python
transactions=pd.merge(transactions,product_areas, how='inner', on= 'product_area_id')
```
Then  we drop the 'Non - Food' columns, since we are not interested in this, as we are addressing food shopping

```python
transactions.drop(transactions[transactions['product_area_name']=='Non-Food'].index, inplace=True)
```
Then we aggregate the sales on customer level

```python
transactions_summary=transactions.groupby(['customer_id','product_area_name' ])['sales_cost'].sum().reset_index()
```
And pivot the data, to get each user_id as row

```python
transactions_summary_pivot=transactions.pivot_table(index='customer_id',
                                                    columns='product_area_name',
                                                    values='sales_cost',
                                                    aggfunc='sum',
                                                    fill_value=0,
                                                    margins=True,
                                                    margins_name='Total').rename_axis(None, axis=1)
 ```
 Lastly, we want to turn the sales into sales percentages
 
 ```python
 
 ```
