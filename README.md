# Congressional-Votes-Scrape

The roll call votes in on the clerk.house.gov/votes website is quite cumbursome when it comes to analyzing how members vote on certain bills. This code attempts to scrape this website by each Congress. Their website has an XML version which makes it easy for a reapeatable URL structure. Each Congress is a year 118 2nd is 2024, for example. So the code will be year-specific representing that year's Congress. 

# Process

The origional idea was to have the code export Excel files to then manipulate in Excel, but the file sizes are too large and Excel crashes. It now dumps into a SQL database where you can use SQL queries and then export to get the data you want. This code just gathers and dumps into a SQL database. 
