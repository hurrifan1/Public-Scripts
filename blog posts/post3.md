#### Creating a QlikSense Dashboard for a Law Firm

For this post, I am going to detail my attempt at creating a QlikSense dashboard using an old dataset containing data from a law firm in Northeastern US. The inspiration for tackling this project comes from an idea I had to land a job that I really want. The company I'm interestd in working for uses QlikView dashboards (I know, not the exact same as QlikSense) to bring law firms into the modern age and give them insight into their data. So, I figured it would behoove me to make a dashboard myself so that I have something of substance to show their hiring manager.  

So, below, I provide a step-by-step of the entire process of making this work! To begin...  

##### Finding the data  
My first step was simply to find the best data to use. I initially worried that I wouldn't be able to find any data that would allow me to simulate a dashboard with basic lawyer info and, hopefully, some numbers to play with. Even though I found a great dataset [here](https://www.stats.ox.ac.uk/~snijders/siena/Lazega_lawyers_data.htm), it was really more of a jumping-off point, because I was ultimately going to need to "invent" a lot of the data myself. This project is really just to prove that I know something about Qlik software so I don't need a massive trove of accurate data.

##### Setting up the csv file  
One of the downsides of using this dataset is that it comes in a .dat file, which isn't super helpful. I'll have to copy the data into Excel so that it's formatted correctly and prepared to push into QlikSense.  

##### Add in random datapoints
I used the following websites to help me create fake records for the clients and lawyers:
- https://www.randomlists.com
- http://listofrandomnames.com/
- http://random-name-generator.info/ 

After I made the fake names, I merged them with the downloaded dataset, which, for the lawyers, included columns like Seniority, Position, Gender, Office, Law School, etc. I made my own clients dataframe, which included columns like Status, Billable Hours, Outstanding (Fees), Found Via, etc. The dataframes had 71 and 200 rows each, respectively, excluding the header rows. In hindsight, I wish I had made 500 fake clients instead of 200 just to have more numbers to work with.  

As far as building in made up numbers, I wanted to make the most of Qlik by having monetary values to input. So, I gave each client a random (but believable) amount of billable hours, which was then multiplied by the hourly rate assigned to their principal (read: attorney). I then paid off a random amount of each client's owed fees, just to make things more interesting.  

##### Upload files and prep data in Qlik
Next on the list was to upload the CSV files into Qlik so that they can be properly formatted and keyed. After uploading both the lawyer and client sheets as CSV files, I was taken to the Qlik Data Manager view. Here, I first had to make sure each sheet was properly delimited and forced to have the first row as column headers. Then, I had to connect the tables together (primary key, anyone?). The lawyer table used the Seniority column as an index of each attorney in the firm, so I linked that column to the Principal column in the clients table, since that used the same index to identify that client's lead attorney.

##### The fun part: Making the visualizations
So, I may be skipping a few of the nitty gritty steps, but that's okay. Time for the fun part of this project! There was a bit of a learning curve at first, but that's mostly because I didn't know going into this that Qlik uses its own scripting language (loosely based on SQL, I believe...). However, after playing around with the program, I realized that I don't have to use very much scripting at all. Once that was established, I actually had a lot of fun testing out different chart types and trying to find the coolest ways to display my data.  

Here's the first iteration of my lawyer dashboard:  


As shown in the screenshot, you can filter the data by lawyer name, client name, and client status. The charts that I chose are Years with Firm, Law School, Lawyer Age Distribution, Fees Inflow vs. Outstanding, Office Locations, and Billable Hours by Client. My goal when choosing which charts to include was not necessarily to pick realistic ones, but rather to choose ones that may produce any amount of interesting information.  

I actually had a lot of fun customizing the appearance of the dashboard, including choosing the colors and color gradients for the charts. I tried to maintain a color scheme of blue, yellow, and white throughout but I added some reds in there for variety.

#### In summary...
...I learned a lot from this project! I learned that finding the right dataset isn't always as easy as it may seem and that you may have to 'invent' a few of your variables in order to get the full effect of the simulation. I also discovered that Qlik Sense can be a pretty intuitive tool once you get the hang of things. I plan on using Qlik more in the future to further explore its features and capabilities.
