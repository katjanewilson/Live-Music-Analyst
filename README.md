# Live Music Analyst
Commissioned data journalism article with [The Pudding](https://pudding.cool/) 

# Project Page

[Live Music Analyst](https://pudding.cool/2021/02/jukebox/) 


An estimated 10,000 artists have recorded live albums in recording history. One of our conceptual challenges in this project was figuring out how to get a comprehensive list of artists and albums, while still parsing down the data to create a functional tool. Alas, we might have missed your favorite artist, but hope that we struck some sort of comprehensive list, and encourage you to dig for that underground live version you can’t find here. Our list of artists comes from Wikipedia’s list of live recorded albums, compared to the set of Pop/Rock artists who recorded at least one live recorded album by 9/16/2020, as they appear on AllMusic. We pulled the intersected list of artist names (about 10,950) from the Spotify API, and ended up with around 8,000 artists’ Spotify API data. We culled this list of artists by selecting only artists who had charted on the Billboard Hot 100.

Raw SpotifyAPI data includes song level variables such as acoustics, speech levels, and duration of a song, and album level variables such as album name and year recorded. We hard coded a binary indicator of live/studio by identifying tracks with the string “Live” or “Sessions” in the track name or in the album name. We removed tracks with “Commentary”, “Rehearsals”, “Intro”, “Movie Soundtrack”, or “Demos” from the analysis, and only kept artists who had at least 7 recorded live songs, per the previously created variable, as an indicator that live songs are representative of a live album recording, rather than a one off recording.

Using the new binary indicator of live/studio, we matched tracks by track name and artist name. This yielded multiple matches for a single song. For instance, seven live recordings of “Folsom Prison Blues” exist in the data set, and four studio recordings (Mercury Albums, Total Johnny Cash Sun Collection, All About the Blue Train, and I Walk the Line), giving us 28 possible matches to compare among Live to Studio. To cull the data for the project, we scraped the “popularity index” from our original artist list, and kept only the tracks with the highest popularity index. Using this new, parsed down, matched list of live and studio versions, we calculated the difference of audio features (acoustics, duration, instrumentals) from the live to studio version for each track.

Music is in our hearts, and we will be the first ones back at the venues when they open up again. In the meantime, you can support live music during covid by getting your live performance ready for when the stage opens back, or find other ways to support live musicians.

## Info
* Spotify API
* Data Visualization (D3.js)
* Data Cleaning (Tidyverse)

