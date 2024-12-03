# -Optimizing-Last-Mile-Delivery-Through-Geospatial-Clustering-of-Nearby-Societies-
Optimized the routing process for over 600 vehicles, reducing the vehicle count by 15%, which led to significant cost savings and improved delivery timelines. By implementing clustering techniques (DBSCAN), travel distances and fuel consumption were minimized, enhancing operational efficiency.

![image](https://github.com/user-attachments/assets/96c9cf03-4d71-4d7f-bc75-46f6714cc6ff)


# Chart-1 Earlier, societies had multiple clusters, chart 2- but after applying DBSCAN, there is only one cluster with outliers societies
![image](https://github.com/user-attachments/assets/71d3520d-57a1-4f6b-ba8a-24e6a68f9f16)



```sql
import pandas as pd
from sklearn.cluster import DBSCAN
import numpy as np
import matplotlib.pyplot as plt
```


**The code from google.colab import drive imports the drive module from Google Colab, allowing you to interact with your Google Drive files.

The drive.mount('/content/drive') command mounts your Google Drive to the Colab environment, providing access to your files stored in Google Drive. Once mounted, you can read, write, and manipulate files in Google Drive directly from your Colab notebook. The mounted drive is accessible at the path /content/drive.**

```sql
from google.colab import drive
drive.mount('/content/drive')
```

The variable file_path stores the location of the file D.csv within your Google Drive. The path "/content/drive/MyDrive/Reduce vehicle count and grouping societies/D.csv" points to the file in the mounted Google Drive directory, allowing you to access it from your Colab environment.

```sql
file_path = "/content/drive/MyDrive/Reduce vehicle  count and grouping societies/D.csv"  # Replace with your file path
```

data = pd.read_csv(file_path, sep=",") loads the CSV file from file_path into a Pandas DataFrame, using commas as the separator.Print the data.

```sql
data = pd.read_csv(file_path,sep=",")
```
print(data.head(5))


coords = data[['Latitude', 'Longitude']].to_numpy() extracts the 'Latitude' and 'Longitude' columns from the DataFrame data and converts them into a NumPy array."""

```sql
coords = data[['Latitude', 'Longitude']].to_numpy()
```


from geopy.distance import great_circle imports the great_circle function from the geopy library, which calculates the shortest distance between two points on the Earth's surface, using their latitude and longitude coordinates.

```sql
from geopy.distance import great_circle
```

The code performs the following actions:
Set clustering radius: epsilon = 2 / kms_per_radian sets the radius for clustering to 2 km, converting it to radians. The variable kms_per_radian likely holds the number of kilometers per radian of latitude.
DBSCAN Clustering: The DBSCAN algorithm is applied to the coordinates (coords) to identify clusters:
eps=epsilon sets the radius for grouping points.
min_samples=2 specifies the minimum number of points needed to form a cluster.
algorithm='ball_tree' selects the algorithm used for the distance computation.
metric='haversine' uses the Haversine formula to calculate distances between points on the Earth's surface, considering the curvature of the Earth.

```sql
kms_per_radian = 6371.0088

# Set clustering radius to 3 km (convert to radians)
epsilon = 2 / kms_per_radian
# Assuming coords is a numpy array with latitude and longitude
db = DBSCAN(eps=epsilon, min_samples=2, algorithm='ball_tree', metric='haversine').fit(np.radians(coords))
```

data['Cluster'] = db.labels_ assigns the cluster labels obtained from the DBSCAN algorithm (db.labels_) to a new column called 'Cluster' in the data DataFrame. Each point in the DataFrame will be labeled with its corresponding cluster, where:
A label of -1 indicates noise (points that don't belong to any cluster).
Other values represent the cluster number assigned to each point.

```sql
data['Cluster'] = db.labels_
print(data.head(5))
```

Define output path: output_path = "clustered_data.xlsx" specifies the file path where the clustered data will be saved, in this case, as an Excel file.
Save DataFrame to Excel: data.to_excel(output_path, index=False) saves the data DataFrame (with the added 'Cluster' column) to an Excel file at the specified output_path. The index=False argument prevents the DataFrame's index from being written to the Excel file.
Print confirmation: print(f"Clustered data saved to {output_path}") outputs a message confirming the file has been saved, with the specified path.

```sql
output_path = "clustered_data.xlsx"
data.to_excel(output_path, index=False)
print(f"Clustered data saved to {output_path}")
```

The code creates an interactive scatter plot using Plotly Express. It:
Plots Longitude on the x-axis and Latitude on the y-axis.
Colors the points based on the Cluster column.
Displays the Society Name and Cluster on hover.
Adds a title "Spread of Societies" and labels for the axes.
Finally, fig.show() renders the plot.

```sql
import plotly.express as px
import pandas as pd
# Create an interactive scatter plot
fig = px.scatter(
    data,
    x='Longitude',
    y='Latitude',
    color='Cluster',
    hover_data=['Society Name', 'Cluster'],
    title='Spread of Societies',
    labels={'Longitude': 'Longitude', 'Latitude': 'Latitude'}
)

# Show plot
fig.show()
```
![image](https://github.com/user-attachments/assets/ab0b37f0-5dfc-42cd-8531-1fc9f7d76a8b)


```sql
df=data
```
Filter out noise points: df = df[df["Cluster"] >= 0] filters the DataFrame to include only rows where the 'Cluster' column has values greater than or equal to 0, effectively removing noise points (labeled as -1 by DBSCAN).
Rename column: df = df.rename(columns={"Society Name": "Society_Name"}) renames the column "Society Name" to "Society_Name" for consistency or style.
Display first 5 rows: print(df.head(5)) prints the first 5 rows of the modified DataFrame.
This results in a cleaned and renamed DataFrame with no noise points, ready for further analysis or visualization.

```sql
#Let's filter this data and remove the societies that are outliers.
df=df[df["Cluster"]>=0]
df=df.rename(columns={"Society Name":"Society_Name"})
print(df.head(5))
```

The code performs the following actions:
Concatenate Society_Name and Latitude: df.loc[:, 'cc'] = df["Latitude"].astype(str) + df["Society_Name"] creates a new column 'cc' by concatenating the Latitude (converted to a string) and Society_Name. This combines both values into a single string for each row.
Display first 5 rows: print(df.head(5)) prints the first 5 rows of the DataFrame, now with the additional 'cc' column.
This results in a new column 'cc', where each entry contains the Latitude followed by the corresponding Society_Name.

```sql
#lets concatenate society  name and latitude
df.loc[:, 'cc']  =df["Latitude"].astype(str) +df["Society_Name"]
print(df.head(5))
```
```sql
df1 = pd.read_csv("/content/drive/MyDrive/Reduce vehicle  count and grouping societies/societies_orders.csv",sep=",")
df1=df1.rename(columns={"Society Name":"Society_Name"})
print(df1.head(10))
```

The code performs the following actions:
Create an in-memory SQLite database:
conn = sqlite3.connect(':memory:') creates an in-memory SQLite database, meaning the database exists only during the runtime of the program.
Create a cursor:
cursor = conn.cursor() initializes a cursor to interact with the SQLite database.
Load DataFrames into SQLite:
df.to_sql('df', conn, index=False, if_exists='replace') stores the DataFrame df in the in-memory SQLite database as a table named df. If the table already exists, it will be replaced.
df1.to_sql('df1', conn, index=False, if_exists='replace') does the same for the DataFrame df1, creating a table named df1.
Execute SQL Query:
query = "SELECT df.Society_Name, df.Latitude, df.Longitude, df.Cluster, Orders FROM df JOIN df1 ON df.cc = df1.cc" is an SQL query that performs an inner join between the two tables (df and df1) on the 'cc' column, selecting specific columns from both tables.
Fetch and display results:
result = pd.read_sql(query, conn) runs the query and loads the results into a Pandas DataFrame result.
print(result.head(55)) prints the first 55 rows of the query result.
This process simulates querying a database using SQLite with two DataFrames (df and df1) stored in an in-memory database, where data from both tables is joined based on the 'cc' column.
#An Excel file was used as a prototype instead of the actual database to address privacy concerns."
import sqlite3

```sql
# Create an in-memory SQLite database
conn = sqlite3.connect(':memory:')
cursor = conn.cursor()
# Load DataFrame into SQLite
df.to_sql('df', conn, index=False, if_exists='replace')
df1.to_sql('df1', conn, index=False, if_exists='replace')
# Example Query to Fetch All Columns
query = "SELECT df.Society_Name,df.Latitude,df.Longitude,df.Cluster,Orders FROM df join df1 on df.cc=df1.cc"
result = pd.read_sql(query, conn)
# Display the query result
print(result.head(55))
result
print(type(result))
```


The code result1 = result["Orders"].sum() calculates the sum of the values in the "Orders" column of the result DataFrame.
The variable result1 will hold the total sum of all the orders in that column.

```sql
result1=result["Orders"].sum()
result1
```

while Total orders are 532.
A Bolero can carry up to 300 orders, while a Tata Ace can handle 250 orders. Since the total number of orders exceeds 300, two Boleros can be deployed to deliver the orders.

