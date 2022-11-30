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