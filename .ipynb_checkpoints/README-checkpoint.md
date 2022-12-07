## Nicole's Life in Music: Analyzing 7 Years worth of Music Streaming Data

The Spotify API I will be using gives me access to a plethora of data from the Spotify catalog. Some examples of the information I can fetch are: 

1. Song title, artist, album, and my personal playlist metadata
2. High-level audio features for each song
3. In-depth audio analysis for tracks
4. and more

First we install the Spotify Web API 

```python
pip install spotipy
```

To access all the data we want, we need register our app with spotify. To do this, go to https://developer.spotify.com/my-applications and create a new Application.

I took the credentials given by spotify and put them in a python executable script called mysecrets to use as my log in credentials when accessing the APi. 

Next we need to get an authorization code from spotify

```python
client_id = mysecrets.client_id
client_secret = mysecrets.client_secret

auth_headers = {
    "client_id": client_id,
    "response_type": "code",
    "redirect_uri": mysecrets.uri,
    "scope": "user-library-read"
} 
```

A page from spotify will pop up like below. Once you give access you will be taken to a blank page. You copy the URL after "code = " and that will be authorization number. 

![image](images/spotifyaccess.png)

Next we need our Authorization Token, which we will use to pass the API and start querying data

```python
encoded_credentials = base64.b64encode(client_id.encode() + b':' + client_secret.encode()).decode("utf-8")

token_headers = {
    "Authorization": "Basic " + encoded_credentials,
    "Content-Type": "application/x-www-form-urlencoded"
}

token_data = {
    "grant_type": "authorization_code",
    "code": code,
    "redirect_uri": "http://localhost:7777/callback"
}

r = requests.post("https://accounts.spotify.com/api/token", data = token_data, headers = token_headers)
```
When using this API Spotify says our client credentials must be in a base64 encoded format to gain authorization access.

Here we are also indicating that we are going to be passing in our authorization code to access or library. Then we make a request to the https://accounts.spotify.com/api/token endpoint with our token_headers and token_data attached.

Now we are connected to the SPotify API! I'm going to run a quick query to make sure it's working

```python
user_headers = {
    "Authorization": "Bearer " + token,
    "Content-Type": "application/json"
}
user_params = {
    "limit": 50
}
user_tracks_response = requests.get("https://api.spotify.com/v1/me/tracks", params=user_params, headers=user_headers)
print(user_tracks_response.json())user_headers = {
    "Authorization": "Bearer " + token,
    "Content-Type": "application/json"
}

user_params = {
    "limit": 5
}
user_tracks_response = requests.get("https://api.spotify.com/v1/me/tracks", params=user_params, headers=user_headers)
print(user_tracks_response.json())
```

KEEP IN MIND THE TOCKEN EXPIRES AFTER 1 HOUR SO YOU WILL HAVE TO RERUN THIS PART AND GET A NEW TOKEN EVERY HOUR. So as we are extracting our data we want to save it else where so this won't hinder us during our analysis.

Moving forward the spotify language has it's own language so to learn how to call all the endpoints I'm going to need in this project I studied Spotify's API documentation found here: https://developer.spotify.com/documentation/web-api/reference/#/ 

I think the best way to approach this data set is to create function to iterate through my track data when I get it.

First I need to get the track ID's from the playlists I will be examining. I couldnt find a way to just extract them from the API so I manually went into my spotify to get them. To do this I collected all the Playlist URI's assigned to the 7 playlists I will be looping through.

![image](images/playlist.png)

My Playlists:
 - 2015 -> 7eBMr6C6GgyUdphppfsHGG
 - 2016 -> 5s30jHYXIa8pqpZ5llc8UW
 - 2017 -> 0y3DRV98utNLVxPkFUHr60
 - 2018 -> 4oICEU7lJwh5B1bi11Fl81
 - 2019 -> 0A6Mf0tkJTZyvoMBq29dzP
 - 2020 -> 1LNWFkUgYkCcqhjl1zArEJ
 - 2021 -> 2my6pETpT9mbvgUmbbE123
 - 2022 -> 08hpQsg5CMFiO8zngrxBwm
 
We create a function to get all the track_ids from our playlists.

```python
def get_track_id(user,playlist_id):
    track_id = []
    playlist = sp.user_playlist(user,playlist_id)
    for item in playlist['tracks']['items']:
        track = item['track']
        track_id.append(track['id'])
    return track_id
```

We then create lists of our playlists, that contain all the track ids in each playlist.

```python
p2015 = get_track_id("spotify","7eBMr6C6GgyUdphppfsHGG")
p2016 = get_track_id("spotify","5s30jHYXIa8pqpZ5llc8UW")
p2017 = get_track_id("spotify","0y3DRV98utNLVxPkFUHr60")
p2018 = get_track_id("spotify","4oICEU7lJwh5B1bi11Fl81")
p2019 = get_track_id("spotify","0A6Mf0tkJTZyvoMBq29dzP")
p2020 = get_track_id("spotify","1LNWFkUgYkCcqhjl1zArEJ")
p2021 = get_track_id("spotify","2my6pETpT9mbvgUmbbE123")
p2022 = get_track_id("spotify","08hpQsg5CMFiO8zngrxBwm")
```

Next we create another function that will extract all the related information about each song in the playlist.

```python
def get_features(id):
    track_info = sp.track(id)
    features_info = sp.audio_features(id)
    
    #song information
    
    name = track_info['name']
    album = track_info['album']['name']
    artist = track_info['album']['artists'][0]['name']
    release_date = track_info['album']['release_date']
    duration = track_info['duration_ms']
    popularity = track_info['popularity']
    
    #song audio features
    
    acousticness = features_info[0]['acousticness']
    danceability = features_info[0]['danceability']
    energy = features_info[0]['energy']
    instrumentalness = features_info[0]['instrumentalness']
    liveness = features_info[0]['liveness']
    loudness = features_info[0]['loudness']
    speechiness = features_info[0]['speechiness']
    tempo = features_info[0]['tempo']
    time_signature = features_info[0]['time_signature']
    
    track_data = [name, album, artist, release_date, duration, popularity, acousticness,
                  danceability, energy, instrumentalness, liveness, loudness, speechiness, tempo, time_signature]
    return track_data
    
```

Then we take this information and put it into a dataframe. Spotify has a limit on the number of executions you're allowed to run at once, so we use the time.sleep() function to create a delay, that prevents too many requests at once.

```python
track_list_2015 = []
for i in range(len(p2015)):
    time.sleep(.3)
    track_data = get_features(p2015[i])
    track_list_2015.append(track_data)
    
playlist_2015 = pd.DataFrame(track_list_2015, columns = ['Title', 'Album', 'Artist', 'Release_Date', 'Duration', 'Popularity', 'Acousticness',
                  'Danceability', 'Energy', 'Instrumentalness', 'Liveness', 'Loudness', 'Speechiness','Tempo', 'Time_Signature'])
playlist_2015.head(3)
```

Here is a sample of our dataframe

![image](images/dataframe.png)


Now we do this for all of our 7 playlists.
