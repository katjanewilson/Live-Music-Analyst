# Live Music Analyst
Commissioned data journalism article with [The Pudding](https://pudding.cool/) 

## Project Page

[Live Music Analyst](https://pudding.cool/2021/02/jukebox/) 

## Methods Overview

An estimated 10,000 artists have recorded live albums in recording history. Our list of artists for this project comes from Wikipedia’s list of live recorded albums, compared to the set of Pop/Rock artists who recorded at least one live recorded album by 9/16/2020, as they appear on AllMusic. We pulled the intersected list of artist names (about 10,950) from the Spotify API, and ended up with around 8,000 artists’ Spotify API data. We culled this list of artists by selecting only artists who had charted on the Billboard Hot 100.

Raw SpotifyAPI data includes song level variables such as acoustics, speech levels, and duration of a song, and album level variables such as album name and year recorded. We hard coded a binary indicator of live/studio by identifying tracks with the string “Live” or “Sessions” in the track name or in the album name. We removed tracks with “Commentary”, “Rehearsals”, “Intro”, “Movie Soundtrack”, or “Demos” from the analysis, and only kept artists who had at least 7 recorded live songs, per the previously created variable, as an indicator that live songs are representative of a live album recording, rather than a one off recording.

Using the new binary indicator of live/studio, we matched tracks by track name and artist name. This yielded multiple matches for a single song. For instance, seven live recordings of “Folsom Prison Blues” exist in the data set, and four studio recordings (Mercury Albums, Total Johnny Cash Sun Collection, All About the Blue Train, and I Walk the Line), giving us 28 possible matches to compare among Live to Studio. To cull the data for the project, we scraped the “popularity index” from our original artist list, and kept only the tracks with the highest popularity index. Using this new, parsed down, matched list of live and studio versions, we calculated the difference of audio features (acoustics, duration, instrumentals) from the live to studio version for each track.

``` r
# data import
theroots <- get_artist_audio_features('the roots')
pac <- get_artist_audio_features('2pac')
run_dmc <- get_artist_audio_features('run dmc')
sublime <- get_artist_audio_features('sublime')
daftpunk <- get_artist_audio_features('daft punk')
yes <- get_artist_audio_features('yes')

# live tracks
grouped <- small_total %>%
  mutate(livemarker = case_when(
    str_detect(track_name, "- Live") |
      str_detect(track_name, "(Live)") |
      str_detect(album_name, "Live") ~ "live"
  ))
grouped$livemarker[is.na(grouped$livemarker)] <- "studio"
grouped <- grouped %>%
  select(livemarker, track_name, artist_name, album_name, danceability, energy, tempo, liveness, valence,
         speechiness, acousticness, instrumentalness) %>%
  group_by(artist_name, livemarker) %>%
  mutate(n = n()) %>%
  filter(n > 6)

# auditory features
grouped$liveness <- as.numeric(grouped$liveness)
grouped_liveness <- grouped %>%
  group_by(artist_name, livemarker) %>%
  summarise(mean_liveness = mean(liveness))
data_wide <- spread(grouped_liveness, livemarker, mean_liveness)
#remove any rows that have an NA
data_wide <- na.omit(data_wide)
#calculate difference
data_wide_liveness <- data_wide   %>%
  select(artist_name, studio, live) %>%
  mutate(difference_liveness = (studio-live),
         difference_liveness_abs = abs(studio-live),
         polarity = ifelse(difference_liveness <0, "livealbum", "studioalbum"))
top_livealbum_liveness <- data_wide_liveness %>%
  filter(polarity == "livealbum") %>%
  arrange(desc(difference_liveness_abs))

```

### Citations

* [Spotify API](https://developer.spotify.com/documentation/web-api/libraries/)
* [Data Visualization](https://d3js.org/)
* [Data Cleaning](https://cran.r-project.org/web/packages/tidyverse/index.html)



