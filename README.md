# EDA-Project
----
### Importing the Data

Upon importing, it sprung one crucial error: it is not in UTF8. UTF8 is like the alphabet of computers. They are a collection of characters that is used to show texts in websites. The data not being UTF8 means that there are unknown characters in the data, which can imply that it is corrupted. 

```
# Imports all the needed data
import pandas as pd
import matplotlib.pyplot as plt

# Upon import the first time, I encountered an error which states that the data is not in UTF8
# This code makes sure that the data runs using the code 'ISO-8859-1'
spd = pd.read_csv('spotify-2023.csv', encoding='ISO-8859-1')
spd

```
ISO-8859 is the suggested code by chat gpt. Reading data that is not UTF8 is above my paygrade 


### Changing to UTF8 Values
```
def change_utf8(value):
    if isinstance(value, bytes):
        # If the value is a byte string, decode it to UTF-8
        value = value.decode('utf-8', errors='ignore')  # Ignore decoding errors
    try:
        # Attempt to encode and decode as UTF-8 to check validity
        value.encode('utf-8').decode('utf-8')
        return value  # If successful, return the original string
    except UnicodeEncodeError:
        # If there's an error, remove or replace non-UTF8 characters directly
        return ''.join([char for char in value if ord(char) < 128])
```
I created a function for changing each value to UTF8. This makes it so that I could just iterate the code in a for loop later on.

```
# Loop over each cell and change non-UTF8 characters
for col in spd.columns:
    spd[col] = [change_utf8(cell) if isinstance(cell, str) or isinstance(cell, bytes) else cell for cell in spd[col]]

print(spd)

```
----

### Part 1 : Overview of the Dataset

Experience: Upon successfully converting all unknown characters to UTF8, its time to move on to answering the needed parameters to successfully complete the activity. This part is relatively simple as I just need to input a few functions in order to find the characteristics of the Dataset

#### How many rows and columns does the dataset contain?
```
[rows,columns] = spd.shape

print('Rows of Spotify Data Set:', rows)
print('Columns of Spotify Data Set:', columns)
```
I used this code to identify all the rows and columns in the given data set. This gives
Rows of Spotify Data Set: 953
Columns of Spotify Data Set: 24

```
[rows,columns] = spd.shape
```
This is inspired by the syntax taught to me in MatLab on Ramp. I tried it and I am surprised that this works in python

#### What are the data types of each column? Are there any missing values?
```
for col in spd:
    print("Data type of ", col, "is ", spd[col].dtype)
```
Coding is the only career where you are allowed to be lazy. For this question, I just iterated each row and determine the data types of each column. For the question about missing values in the dataset, my answer will be brought up in the following codes later on. 

----

### Part 2 : Basic Descriptive Statistics

Experience: First run of the streams column showed an error. For some reason, the streams for the song "Love Grows (Where My Rosemary Goes)" has the value "BPM110KeyAModeMajorDanceability53Valence75Ener..."  which is a string.

To fix, I first find its location using
```
# A song has the a non-numeric value in streams, detected by this code
non_numeric_rows = spd[spd['streams'].str.contains('[^0-9.]')]
non_numeric_rows
```

For convinience, I replaced the missing value for 0, since the original value is nowhere to be found
```
# Replaces 574 streams with 0 as undefined
spd.loc[574, 'streams'] = 0
```

Then, upon close inspection, all the values in streams are of 'object datatype'. This is because upon converting to UTF8, some values are too large, so the program just converts it to a object. However, in order to run the next sets of questions, I change it to float. Float because int is too small to handle the data
```
# Convert all str in streams into float
Streams = spd['streams'].astype('float')

```

#### What are the mean, median, and standard deviation of the streams column? 

This is pretty straightforward. I just printed the needed values using .mean(), .median(), and .std()
```
print('Mean of streams:', Streams.mean())
print('Median of streams:', Streams.median())
print('Standard Deviation of streams:', Streams.std())

```
to get the following:
Mean of streams: 513597931.3137461
Median of streams: 290228626.0
Standard Deviation of streams: 566803887.0588316


#### What is the distribution of released_year and artist_count? Are there any noticeable trends or outliers?
```
# Box plot for both columns side by side
spd[['released_year']].plot(kind='box')
plt.title('Box Plot of Released Year')
plt.ylabel('Values')
plt.show()
```
The best graph to express this is a box plot. A box plot allows the user to gauge the distribution of the population while showing notable outliers in the data. The box plot generated shows the following:

![image](https://github.com/user-attachments/assets/0960fc7a-fb5f-4996-94bb-68bbd19a642c)

The plot indicates that majority of the released songs that topped the charts are recent songs, mostly on the 2020s. This shows that majority of popular songs for a given year are more likely the ones that are released relatively new. The interesting part of this data is that songs from the 1960s are still getting streams even if they are released decades before. 

```
# Box plot for both columns side by side
spd[['artist_count']].plot(kind='box')
plt.title('Box Plot of Artist Count')
plt.ylabel('Values')
plt.show()

```
![image](https://github.com/user-attachments/assets/77f03358-3459-4886-afa8-347ee8032d20)

The plot indicates that majority of songs that have streams are those created by just 1-2 artist. However, this data is flawed in that band and groups with a name is registered as just that name.This indicates that BTS, ColdPlay, and other bands are under the one artist mark. Notable outliers have 8 people working on one song. Whatever works for them! =)

----

## Part 3: Top Performers

Experience: Apparently, I stored the converted float streams in a variable so the original column in the data set is not converted to floats. Fixed it using the following: 

```
# Converts it to string, and removes all white spaces since it affects .astype(float)
spd['streams'] = spd['streams'].astype(str).str.strip()

# Convert to float
spd['streams'] = spd['streams'].astype('float')

```
The main function is .astype('float'), however, it does not work because there are spaces in the streams column. Had to remove it first using the .strip() function


### Which track has the highest number of streams? Display the top 5 most streamed tracks.

```
# Arrange by number of streams and get the top 5 using the .head and ascending = false
spd.sort_values(by='streams', ascending = False).head(5)

```
Used the sort values function with the attribute 'ascending = false' to arrange the streams in descending order. '.head(5)' function is used to only show the top 5. This gave:
![image](https://github.com/user-attachments/assets/b6b764a4-51e4-46ab-8727-7d62da7474ac)
Interesting... 

### Who are the top 5 most frequent artists based on the number of tracks in the dataset?

This is a bit tricky...

```
# Single Ranking 
# Counts the values in the artist name column
artist_count_solo = spd['artist(s)_name'].value_counts()
artist_count_solo.head(5)
```
This is the most straight forward answer which based on the column itself. I used the value_counts function to count the number of occurances each artist is seen on the column. Giving:

Taylor Swift    34
The Weeknd      22
Bad Bunny       19
SZA             19
Harry Styles    17

which are mostly artist who tend to release songs and not collab. 

```
# If collabs are separated
expanded_artists_separate = spd['artist(s)_name'].str.split(', ').explode().value_counts()
expanded_artists_separate.head(5)
```
However, collaboration among artists are a big hit, and it is shown on the column. Artists separated by commas are registered only as 1 artist in the first code. This code fixes that by spliting the string, exploding the values, and then counting. This shows:

Bad Bunny         40
Taylor Swift      38
The Weeknd        37
SZA               23
Kendrick Lamar    23

where Bad Bunny overtakes taylow due to his collabs with other artists. 

** Modifications that include explode needed to be stored in a variable so that the original data is not affected. However, both codes are stored in a variable for convinience purposes and cleanliness of results

---

## Part 4: Temporal Trends

Experience: The data found here are quite interesting and goes to show that time and events makes a factor on how a song will do. Those streaming in January are probably in new year's parties.

### Analyze the trends in the number of tracks released over time. Plot the number of tracks released per year.

```
spd[['released_year']].plot(kind = "hist")
plt.title('Number of Tracks released')
plt.ylabel('Year Released')
plt.show()
```

For this code, I decided to use histogram because its the one I am most familiar with. It gets the job done where x-axis shows the year they are released and y-axis shows how many song. This is accurate since it is the same result as the box plot earlier. Used basic Matplotlib funcitons to make it more readable such as plt.title(), plt.ylabel(), and plt.show()

![image](https://github.com/user-attachments/assets/01bf1dec-ad6f-41cf-bd14-cb1f93fbe4e5)

### Does the number of tracks released per month follow any noticeable patterns? Which month sees the most releases?

```
# Group by month and count the occurrences for each month
monthly_counts = spd['released_month'].value_counts().sort_index()

# Plotting as a bar chart
monthly_counts.plot(kind="bar")
plt.title('Number of Tracks Released Over Time')

# Set custom x-axis ticks and labels
plt.xticks(ticks=range(12), labels=[
    'Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 
    'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'
])
plt.ylabel('Number of Tracks')
plt.xlabel('Month')
plt.show()
```

For finding the number of tracks released, I tried using a bar graph. I used .value_counts() again to count how often a value repeats in spd['released_month'], and sorted it in sorted_index. 
It is stored in a variable since we do not want to be changing the original data set. Used .plot(kind = 'bar') just to try the function.

The next few lines of code are there for readability, such as plt.title, plt.xticks., plt.x(and y)label

![image](https://github.com/user-attachments/assets/f91940d6-0971-4610-ad1d-14149a5203f8)

---

## Part 5: Genre and Music Characteristics

Experience: This is the most interesting part of the data as we see all types of genres and attributes that affect the songs successs and streams. 

### Examine the correlation between streams and musical attributes like bpm, danceability_%, and energy_%. Which attributes seem to influence streams the most?


Similar functions are used all throughout: .scatter, the various label functions, and other that make up the component such as adding subplots. 

Scatter plot is used because it is the purest type of graph. It shows the diversity, the deviation, and the population of each range. This makes relationship more easier to derive. 


#### Streams vs BPM

```
fig1 = plt.figure(figsize=(15, 6))  # Increase figure size for better readability
ax = fig1.add_subplot(111) # Adds a subplot for no reason =)

# Scatter plot with adjustments
ax.scatter(spd['bpm'], spd['streams'])



# Adding labels, title, and grid
ax.set_xlabel('BPM')
ax.set_ylabel('Streams')
ax.set_title('Scatter Plot of Streams vs. BPM')

plt.show()
```
![image](https://github.com/user-attachments/assets/86cd66f9-08d2-4c39-8c83-288b13c183ac)

The graph shows that majority of most streamed songs are around 80 to 140 beats per minute. Interesting as it is slow that people can follow it, yet fast to make it exciting. 

#### Streams vs Danceability
```
fig2 = plt.figure(figsize=(15, 6))  # Increase figure size for better readability
ax = fig2.add_subplot(111)

# Scatter plot with adjustments
ax.scatter(spd['danceability_%'], spd['streams'])


# Adding labels, title, and grid
ax.set_xlabel('Danceability')
ax.set_ylabel('Streams')
ax.set_title('Scatter Plot of Streams vs. Danceability')

plt.show()
```
![image](https://github.com/user-attachments/assets/4fcb420c-1182-48aa-8c0b-208223ffefab)

The graph shows that the majority of the most streamed songs are danceable by the public. This is evident through majority of dots falling under the 60-90 % mark of the graph. Most of this can be attributed to Tiktok, which is a trending platform where a persong typicaly dance to a music. In turn it made a lot of songs popular in the 2020s. 

#### Streams vs Energy 
```
fig3 = plt.figure(figsize=(15, 6))  # Increase figure size for better readability
ax = fig3.add_subplot(111)

# Scatter plot with adjustments
ax.scatter(spd['energy_%'], spd['streams'])



# Adding labels, title, and grid
ax.set_xlabel('Energy')
ax.set_ylabel('Streams')
ax.set_title('Scatter Plot of Streams vs. Energy')

plt.show()
```
![image](https://github.com/user-attachments/assets/414d9db6-e0e6-48c5-9a03-8283aaeed63b)

The graph shows that energy plays a big part in getting streams. 40-90 % of the most streamed songs fall in this range which indicates that people tend to stream or play music that gets them hyped up more. 

### Is there a correlation between danceability_% and energy_%? How about valence_% and acousticness_%?

#### Danceability vs Energy
```
fig4 = plt.figure(figsize=(15, 6))  # Increase figure size for better readability
ax = fig4.add_subplot(111)

# Scatter plot with adjustments
ax.scatter(spd['energy_%'], spd['danceability_%'])



# Adding labels, title, and grid
ax.set_xlabel('Energy')
ax.set_ylabel('Danceability')
ax.set_title(' Plot of Danceability vs. Energy')

plt.show()
```
![image](https://github.com/user-attachments/assets/92de222b-957a-4b70-8b8e-c6baafee72aa)


We are tasked to find a correlation between energy and danceability of a song. In this chart, we see that dots are clustered in a way that shows a diagonal cloud. While the correlation is not perfect, this is enough to indicate that the energy of the song can impact how danceable it was. The more the energy the song has, the danceable it was.  


#### Valence vs Acousticness
```
fig5 = plt.figure(figsize=(15, 6))  # Increase figure size for better readability
ax = fig5.add_subplot(111)

# Scatter plot with adjustments
ax.scatter(spd['valence_%'], spd['acousticness_%'])



# Adding labels, title, and grid
ax.set_xlabel('Valence')
ax.set_ylabel('Acousticness')
ax.set_title('Scatter Plot of Valence vs. Acousticness')

plt.show()
```
![image](https://github.com/user-attachments/assets/5c59a17f-dba4-45cf-8553-2620faf20770)

Unlike the previous graph, this one shows no correlation. It does not show that when valence is low, acousticness is low and so on. The data is scattered across the plot, hence we can conclude that they have no correlation. 

----

## Part 6: Platform Popularity

Experience: I kept running the deezer playlist and shazam charts column and it always show that it is a string. It does not work if I used the .astype('int') function which I kept on running. The problem was that there where commas indicating the thousandth and millionth mark! Fixed it through these codes:

```
spd['in_deezer_playlists'] = spd['in_deezer_playlists'].str.replace(',','').astype('int')
spd
```
.replace indicates that the program replaces ',' with '' or an empty set. This effectively removes all the commas in the deezer playlist column

```
# Replaces ',' with '' and fill in NA values with 0
spd['in_shazam_charts'] = spd['in_shazam_charts'].str.replace(',','').fillna(0).astype('int')
spd
```
Aside from having commas, shazam chart also have some missing values. The .fillna(0) exchanges these N/A values with 0. Cannot recover the real values 


### How do the numbers of tracks in spotify_playlists, spotify_charts, and apple_playlists compare? Which platform seems to favor the most popular tracks?


```
# Creates a figure with a great deal of length
fig6 = plt.figure(figsize = (12, 9))

# Create a subplot for box plot for spotify playlist
spplot = fig6.add_subplot(131)
spplot.boxplot(spd['in_spotify_playlists'])
spplot.set_title("In Spotify Playlists")

# Create a subplot for box plot for spotify charts
scplot = fig6.add_subplot(132)
scplot.boxplot(spd['in_spotify_charts'])
scplot.set_title("In Spotify Charts")

# Create a subplot for box plot for apple playlist
applot = fig6.add_subplot(133)
applot.boxplot(spd['in_apple_playlists'])
applot.set_title("In Apple Playlists")

# make it so that labels are cleanly placed by having a tight layout
plt.tight_layout() 
plt.show()

```

Similar to the boxplots I have created before except I used subplots to make each chart side by side. The function plt.tight_layout is used so that the labels are cleanly placed in the figure. Without it, some of the labels will overlap with the graph. This shows the following graph: 

![image](https://github.com/user-attachments/assets/ebde1bfe-2a97-4a60-a32c-1686b25f1c56)

The box plot here indicates that the streamed songs are mostly found in the spotify playlist marks. If we notice the y-axis labels and compare each graph, we see that majority of most streamed songs are found in spotify playlist compared to apple playlist. This indicates that majority of users are in spotify and more likely to add a song there in a playlist. Comparing spotify chart with spotify playlist, we see that most song in the database rarely get featured in the spotify chart. We can prove such as most songs lay on the lower part of the graph of the playlist (0 to 20) where as if we compare it to the numbers that spotify playlists produces, we see that majority of the songs rarely stay atop the charts. 

#### Platform Wars
```
# Top Songs of the year
top = spd.sort_values(by='streams', ascending = False).head(10)
top = top[['track_name','in_spotify_playlists', 'in_spotify_charts', 'in_apple_playlists', 'in_apple_charts', 'in_deezer_playlists', 'in_deezer_charts', 'in_shazam_charts']].reset_index(drop = 'True')
top
```
For these, I gathered the top 10 songs in terms of streams since the question asked for the most popular tracks. Here are the results:
![image](https://github.com/user-attachments/assets/688c7068-686a-4447-a15d-dc854f587381)


```
# Creates a figure with a large size
fig7 = plt.figure(figsize = (12, 9))

# Adds a plot for spotify
spTopCPlot = fig7.add_subplot(141)
spTopCPlot.boxplot(top['in_spotify_charts'])
spTopCPlot.set_title("In Spotify Charts")

# Adds a plot for apple music
apTopCPlot = fig7.add_subplot(142)
apTopCPlot.boxplot(top['in_apple_charts'])
apTopCPlot.set_title("In Apple Charts")

# Adds a plot for deezer
deTopCPlot = fig7.add_subplot(143)
deTopCPlot.boxplot(top['in_deezer_charts'])
deTopCPlot.set_title("In Deezer Charts")

# Adds a plot for shazam
shTopCPlot = fig7.add_subplot(144)
shTopCPlot.boxplot(top['in_shazam_charts'])
shTopCPlot.set_title("In Shazam Charts")


plt.tight_layout() 
plt.show()
```

The codes used are similar to the box plot codes I used before. This time I made it so that 4 charts could be shown at a time by increasing the number of subplots and adusting their widths to entertain another plot. This shows the following

![image](https://github.com/user-attachments/assets/e57dee1d-219e-4585-a7c3-82693c7ae3e4)

In this graph, we see that the top 10 songs are mostly favored in spotify, in terms of number of appearances to charts, while having the most diverse. Apple music came in second in which most the top songs are always featured 100 times and more. Deezer and Shazam rarely feature the top songs in their charts, only showcasing them roughly 40-10 times, except a few notable outliers. These can be due to other factors which cannot be found in the Data Set


```
# Creates a figure with a large size
fig8 = plt.figure(figsize = (12, 9))

# Adds a plot for spotify
spTopPPlot = fig8.add_subplot(131)
spTopPPlot.boxplot(top['in_spotify_playlists'])
spTopPPlot.set_title("In Spotify Playlists")

# Adds a plot for apple music
apTopPPlot = fig8.add_subplot(132)
apTopPPlot.boxplot(top['in_apple_playlists'])
apTopPPlot.set_title("In Apple Playlists")

# Adds a plot for deezer
deTopPPlot = fig8.add_subplot(133)
deTopPPlot.boxplot(top['in_deezer_playlists'])
deTopPPlot.set_title("In Deezer Playlists")

plt.tight_layout() 
plt.show()
```
Graph:
![image](https://github.com/user-attachments/assets/78828695-78a6-4a5c-b11a-c3bf46b2d794)

In this graph we can see that spotify playlists still reigns supreme in terms of having top songs in most of their users playlist. In an interesting turn of events, deezer beats apple having roughtly 1000 - 3000 instances in which top songs are featured in playlists. Apple performes poorly with an IQR of 300 to 490. This indicates that people tend to create playlists more on spotify and deezer than apple.


## Part 7: Advance Analysis

Experience: Upon initial trial of the code, I noticed that 

```
# Replaces missing values with the key of C
spd['key'] = spd['key'].replace("", "C")
spd['key'] = spd['key'].fillna("C")
spd.loc[spd['track_name'] =='Flowers']

```





