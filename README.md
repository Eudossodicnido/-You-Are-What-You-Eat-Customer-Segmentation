# -You-Are-What-You-Eat-Customer-Segmentation

<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/194903847-4da4fda7-eb60-4a51-9f94-24373324eb03.png"
 />
</p>

# Summary
- [01. Business Task]()
- [02. Data]()
- [03. Data Wrangling]()
- [04. Data cleaning and prepartion]()
- [05. Clustering]()
- [06. Results and business outcomes]()


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
then  we drop the 'Non - Food' columns, since we are not interested in this, as we are addressing food shopping,

```python
transactions.drop(transactions[transactions['product_area_name']=='Non-Food'].index, inplace=True)
```
then we aggregate the sales on customer level,

```python
transactions_summary=transactions.groupby(['customer_id','product_area_name' ])['sales_cost'].sum().reset_index()
```
and pivot the data, to get each user_id as row.

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
 data_for_clustering=transactions_summary_pivot.drop(['Total'], axis=1)
 ```
 # 04. Data Cleaning and preparation
 We can now check for missing values (and there are not in our case).

 ``python
  #Checking for missing values
 data_for_clustering.isna().sum()
 ```
 
 It is now time to prepare the data. As we are going to use K-Means, which is a distance base algorithm, it is quite important to normalise the data. we can do it using  MinMaxScaler from Sklearn.
 
``python
#Normalising data
scale_norm=MinMaxScaler()
data_for_clustering_scaled=pd.DataFrame(scale_norm.fit_transform(data_for_clustering), columns=data_for_clustering.columns)
 ```
 
 We are now redy to go!
 
 # 05. Clustering
 In order to start with clustering, we first have to decide the numbers of clusters to use (or the so called number of Ks).
 To do so, we are going to see how Within cluster sum of squares (or WCSS) changes in relations to different numbers for K and choose the one that better answers the trade off between the two.
 
 
<p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/194899360-e908b1fe-2f7c-4670-adaa-b9fe26c9e1b1.png"
 />
</p>

In our case it seems that 3 is a good number to use for K.

We can now create our model 

```python
kmeans=KMeans(n_clusters=3, random_state=42)
kmeans.fit(data_for_clustering_scaled)
 ```
 
 # 06. Results and business outcomes

 Let us start by taking a look at the results
 
```python
#Add cluster label to data
data_for_clustering['cluster']=kmeans.labels_
#Check cluster size
data_for_clustering['cluster'].value_counts()
 ```

- Cluster 0: 641
- Cluster 1: 103
- Cluster 2: 127

We have a quite densely populated cluster and two quite smaller ones. Let's profile them.  

```python
cluster_summary=data_for_clustering.groupby('cluster')[['Dairy','Fruit','Meat','Vegetables']].mean().reset_index()
cluster_summary
 ```
 <p align="center">
  <img src="https://user-images.githubusercontent.com/69009356/194900471-ab21949b-0e11-466e-9c8f-696067f8c233.png"
 />
</p>
 
These clusters seem to make sense: the bigger cluster (n.0) has purchases among all the categories, so it seems to represent a "general" public with traditional purchaces.
Cluster 1 spends basically zero in meat, while quite it spends quite a lot in cheese and in the other categories. It would seems like this cluster could be a "vegetarian" cluster.
Cluster 2, on the other hand, spends 0 in both meat and diary, focusing its purchaces in fruit and vegetables. It would seem this could be a "vegan" cluster.

It does indeed seem like that there are some changing in the customer base and its habits and spending behaviour.
The business can now, numerically, address their customers better by knowing in which segment they belong to and propose them their marketing solutions based on them.


