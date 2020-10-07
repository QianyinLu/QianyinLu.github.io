Title: Manage Spotify Dataset using Python and SQL
Category: Database
Date: 2020-10-07 10:01
Modified: 2020-10-09 11:30
Tags: SQL, Relational database
Slug: Spotify-database
Authors: Presnie Lu
Summary: Use SQLite3 schema to store this data in at least 3rd normal form (3NF) and find names of all playlists that contain instrumentals.

**This blog is used to store [Spotify songs](https://github.com/rfordatascience/tidytuesday/blob/master/data/2020/2020-01-21/readme.md) dataset in at least 3rd normal form (3NF) with SQLite3 and SQL query to find the names of all playlists that contain instrumentals.**  

  

# Identify the relationship between variables

Before building the table, it is first necessary to identify what primary keys should we use and how many tables will be needed to reduce the data to a 3rd normal form. Based on [Wikipedia](https://en.wikipedia.org/wiki/Third_normal_formhttps://en.wikipedia.org/wiki/Third_normal_form), third normal form schema requires all the attributes from the data set are functionally dependent on solely the primary key. Therefore, according to this information, I listed out several potential primary keys to start with and I used Python to check whether these keys are valid. 
    
    :::python
    #read file
    df = pd.read_csv("spotify_songs.csv")
    
    #check whether same track_id has different value on other values
    track_grouped = df.groupby("track_id")
    tracknames = []
    for i in df.columns:
        dup = track_grouped[i].nunique()
        if len(dup[dup>1])== 0:
            print(dup[dup>1])
            tracknames.append(dup[dup>1].name)
    
    #check whether same album has different value on other values
    album_grouped = df.groupby("track_album_id")
    albumnames = []
    for i in df.columns:
        dup = album_grouped[i].nunique()
        if len(dup[dup>1])== 0:
            print(dup[dup>1])
            albumnames.append(dup[dup>1].name)
        
    #check whether same album has different value on other values
    playlist_grouped = df.groupby("playlist_id")
    for i in df.columns:
        dup = playlist_grouped[i].nunique()
        if len(dup[dup>1])== 0:
            print(dup[dup>1])
    
    #check whether subgenre can indicate genre
    subgenre_grouped = df.groupby("playlist_subgenre")
    for i in df.columns:
        dup = subgenre_grouped[i].nunique()
        if len(dup[dup>1])== 0:
            print(dup[dup>1])

The basic idea behind this is we need to separate out the attributes from the table if it does not directly depends on the primary key. The way to check whehter the primary key is valid or not is by grouping the data using the key. If the primary key can not determine other attributes solely, that is, with the same primary key, the other attributes are different, then the key is not valid. In the spotify data set, track ID is definitely not the primary key for all other attributes. In fact, it is easy to imagine that the album release date and name should depend on album directly; Similarly, playlist related information should be splitted from the table. In more detailed, playlist name depends on playlist ID; Genre can be determined by looking at subgenre. Using Python, I affirmed that these are valid primary key because attributes within groups determined by these primary key are the same. 
  

# Building tables based on primary keys

## Track

By the first step, we proved some attributes depends only on track_id. Thus, we only include these attributes in the track table. 

    :::python 
    #track table
    Tracks = df[tracknames]
    #album related information should be included in another table
    Tracks = Tracks.drop(["track_album_name", "track_album_release_date"], 1).drop_duplicates() 
    
## Album

The Album table includes related information from the album only.

    :::python
    #album table
    album = df[albumnames].drop_duplicates()
    
    

## Playlist

The next step is to build separate tables and make sure these tables can be related using some foreign keys.  
I firstly worked on the Playlist table. I first figured out that based on id we can know the name of the playlist. Therefore, I generated a table called playlist_name for matching id with names using playlist_id as the primary key. Similarly, because one subgenre only belongs to one genre, another table called playlist_genre was built by letting subgenre becomes the primary key. Notice that playlists with same ID can have different genres/subgenres and same genre/subgenre definitely can belong to different playlists. In order to solve this problem, I set the row number as the primary key which I called it playlist_Index. 

    :::python
    #playlist table
    playlist_all = df.iloc[:,df.columns.str.contains("playlist")]
    playlist = playlist_all[['playlist_id', 'playlist_subgenre']].drop_duplicates().reset_index()
    playlist["index"] = playlist.index #many to many relationship, needs an external index
    playlist.columns = ["Playlist_Index", "Id", "Subgenre"]
    playlist_name = playlist_all[playlist_all.columns[:2][::-1]].drop_duplicates()
    playlist_genre = playlist_all[playlist_all.columns[2:4][::-1]].drop_duplicates()


## All_music Combined table

A similar question of many-to-many mapping also exists between individual songs and playlist. A song can belong to different playlists and a playlist will have many songs. Therefore, I solve this by using the row number as the primary key to relate the overall track table with the playlist table. In this table, track_id is included so that we can connect this table with track and Playlist_Genre_ID (which is the playlist_Index from the playlist table) so that we can refer to the information of playlist and its genre. These both attributes does not directly rely on each other. The intuition behind this table is that the primary key represents different combinations of an individual song in certain playlist with tags of some subgenre. 

    :::python
    #combined
    df["ID"] = df.index
    alltbl = pd.merge(df, playlist, how = "inner", left_on =["playlist_id","playlist_subgenre"],
         right_on = ["Id", "Subgenre"])
    all_music = alltbl[["ID", "track_id", "Playlist_Index"]] #need id because track_id to playlist is many to many relationship
    all_music.columns = ["Music_ID","Track_ID", "Playlist_Genre_ID"]

   
# Store the data into SQLite3 

We first create a new database to store the data using the following command:

    :::python
    #load sql
    %load_ext sql
    %sql sqlite:///spotify.db

Then, in this database, we need to build empty tables as a basic frame to store the data. We need these tables to be the same structure as what we discussed before in order to build right connections between them.

    :::python
    #sql query
    %%sql
    DROP TABLE IF EXISTS All_Music;
    DROP TABLE IF EXISTS Playlist;
    DROP TABLE IF EXISTS Tracks;
    DROP TABLE IF EXISTS Album;
    DROP TABLE IF EXISTS Playlist_Name;
    DROP TABLE IF EXISTS Playlist_Genre;

    CREATE TABLE Playlist_Genre (
        playlist_subgenre TEXT NOT NULL PRIMARY KEY,
        playlist_genre TEXT NOT NULL
    );

    CREATE TABLE Playlist_Name (
        playlist_id TEXT NOT NULL PRIMARY KEY,
        playlist_name TEXT NOT NULL
    );

    CREATE TABLE Playlist (
        Playlist_Index INTEGER NOT NULL PRIMARY KEY,
        Id TEXT NOT NULL,
        Subgenre TEXT NOT NULL,
            FOREIGN KEY (Id) REFERENCES Playlist_Name(playlist_id),
            FOREIGN KEY (Subgenre) REFERENCES Playlist_Genre(playlist_subgenre)
    );


    CREATE TABLE Album (
        track_album_id TEXT NOT NULL PRIMARY KEY,
        track_album_name TEXT,
        track_album_release_date TEXT
    );

    CREATE TABLE Tracks (
        track_id TEXT NOT NULL PRIMARY KEY,
        track_name TEXT,
        track_artist TEXT,
        track_popularity INTEGER NOT NULL,
        track_album_id TEXT NOT NULL,
        danceability REAL NOT NULL,
        energy REAL NOT NULL,
        key INTEGER NOT NULL,
        loudness REAL NOT NULL,
        mode INTEGER NOT NULL,
        speechiness REAL NOT NULL,
        acousticness REAL NOT NULL,
        instrumentalness REAL NOT NULL,
        liveness REAL NOT NULL,
        valence REAL NOT NULL,
        tempo REAL NOT NULL,
        duration_ms INTEGER NOT NULL,
            FOREIGN KEY (track_album_id) REFERENCES Album(track_album_id)
    );

    CREATE TABLE All_Music (
        Music_ID INTEGER NOT NULL PRIMARY KEY,
        Track_ID TEXT NOT NULL,
        Playlist_Genre_ID INTEGER NOT NULL,
            FOREIGN KEY (Track_ID) REFERENCES Tracks(track_id),
            FOREIGN KEY (Playlist_Genre_ID) REFERENCES Playlist(Playlist_Index)
    );

  

After this, we need to connect the panda dataframe with the database and insert the values into those tables. 
    
    :::python
    #connect with pd dataframe, insert value
    import sqlite3
    con = sqlite3.connect("spotify.db")
    playlist.to_sql("Playlist", con, if_exists='append',index = False)
    album.to_sql("Album", con, if_exists='append',index = False)
    Tracks.to_sql("Tracks", con, if_exists='append',index = False)
    all_music.to_sql("All_Music",con, if_exists='append',index = False)
    playlist_name.to_sql("Playlist_Name", con, if_exists='append',index = False)
    playlist_genre.to_sql("Playlist_Genre", con, if_exists='append',index = False)

With the above commands, the database eventually looks like this:
![Relational Database](/img/relationalDB.png "Relational Database")  

# Find the names of all playlists that contain instrumentals

Base on the description of the dataset, instrumentalness represents whether a track contains no vocals. I chose 0.5 as the thredshold because values above 0.5 are intended to represent instrumental tracks. With such information, I can then filter out playlists that contain tracks satisfying this criterion. The basic logic behind the following command is that base on track table, I can get tracks with instrumentalness; Using the all_music table, I can connect tracks with playlists (see what playlists contain those tracks); Finally, in the playlist table, we can see that is the id of those playlists contain instrumental tracks and we find the names of those playlists using playlist_name table. 

    :::python
    #sql query 
    %%sql
    select distinct Playlist_Name.playlist_name
    from Tracks
    join All_music 
        on Tracks.track_id = All_music.Track_ID
    join Playlist
        on Playlist.Playlist_index = All_music.Playlist_Genre_ID
    join Playlist_Name
        on Playlist.Id = Playlist_Name.playlist_id
    where instrumentalness > 0.5
