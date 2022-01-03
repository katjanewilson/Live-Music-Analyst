# Live Music Analyst
Commissioned data journalism article with [The Pudding](https://pudding.cool/) 

## Project Page

[Live Music Analyst](https://pudding.cool/2021/02/jukebox/) 

## Methods Overview

An estimated 10,000 artists have recorded live albums in recording history. Our list of artists for this project comes from Wikipedia’s list of live recorded albums, compared to the set of Pop/Rock artists who recorded at least one live recorded album by 9/16/2020, as they appear on AllMusic. We pulled the intersected list of artist names (about 10,950) from the Spotify API, and ended up with around 8,000 artists’ Spotify API data. We culled this list of artists by selecting only artists who had charted on the Billboard Hot 100.

Raw SpotifyAPI data includes song level variables such as acoustics, speech levels, and duration of a song, and album level variables such as album name and year recorded. We hard coded a binary indicator of live/studio by identifying tracks with the string “Live” or “Sessions” in the track name or in the album name. We removed tracks with “Commentary”, “Rehearsals”, “Intro”, “Movie Soundtrack”, or “Demos” from the analysis, and only kept artists who had at least 7 recorded live songs, per the previously created variable, as an indicator that live songs are representative of a live album recording, rather than a one off recording.

Using the new binary indicator of live/studio, we matched tracks by track name and artist name. This yielded multiple matches for a single song. For instance, seven live recordings of “Folsom Prison Blues” exist in the data set, and four studio recordings (Mercury Albums, Total Johnny Cash Sun Collection, All About the Blue Train, and I Walk the Line), giving us 28 possible matches to compare among Live to Studio. To cull the data for the project, we scraped the “popularity index” from our original artist list, and kept only the tracks with the highest popularity index. Using this new, parsed down, matched list of live and studio versions, we calculated the difference of audio features (acoustics, duration, instrumentals) from the live to studio version for each track.

``` r
### function 1: cleans the live frames

clean_live_frame_function <- function(livedata, albumname)  {
  artist_live_edits <- livedata %>%
    mutate(livemarker = ifelse(livedata$album_name == albumname, 1, 0))
  artist_live_edits2 <- artist_live_edits %>%
    mutate(omitvariable = case_when(
      (str_detect(album_name, "Sessions") |
         str_detect(album_name, "Live") |
         str_detect(album_name, "Festival") |
         str_detect(album_name, "Concerts"))& livemarker == 0 ~ "omit"))
  artist_live_edits2 <- artist_live_edits2 %>%
    mutate(track_name = gsub("\\-.*","", track_name))
  artist_live_edits2 <- artist_live_edits2 %>%
    mutate(omitvariable2 = case_when(
      (str_detect(track_name, "Live") & livemarker == 0 ~"omit")
    ))
  artist_live_edits2 <- artist_live_edits2 %>%
    filter(is.na(omitvariable) & is.na(omitvariable2))
  artist_live_edits2 <- artist_live_edits2 %>%
    distinct()
}


mymorningjacket_livedata <- accumulatedlive %>%
  filter(artist_name == "My Morning Jacket")
mymorning_clean <- clean_live_frame_function(mymorningjacket_livedata, "Okonokos")

### function2: slope graph creator

slope_graph_creater <- function(clean_live_frame, song) {
  
  song_string<- as.character(song)
  song_string <- substr(song_string, 1,5)
  
  jimi_hendrix_live_slope_try <- clean_live_frame %>%
    filter(str_detect(track_name, song_string))
  
  jimi_hendrix_live_slope_try <- jimi_hendrix_live_slope_try %>%
    select(livemarker, liveness, danceability, acousticness,
           energy, valence, livemarker)
  jimi_hendrix_live_slope_try$Liveness <- as.numeric(jimi_hendrix_live_slope_try$liveness)
  jimi_hendrix_live_slope_try$Danceability <- as.numeric(jimi_hendrix_live_slope_try$danceability)
  jimi_hendrix_live_slope_try$Acoustics <- as.numeric(jimi_hendrix_live_slope_try$acousticness)
  jimi_hendrix_live_slope_try$Energy <- as.numeric(jimi_hendrix_live_slope_try$energy)
  jimi_hendrix_live_slope_try$Valence <- as.numeric(jimi_hendrix_live_slope_try$valence)
  jimi_hendrix_live_slope_try$livemarker<- as.factor(jimi_hendrix_live_slope_try$livemarker)
  jimi_hendrix_live_slope_try2 <- jimi_hendrix_live_slope_try %>%
    group_by(livemarker) %>%
    summarise(Acoustics = mean(Acoustics),
              Danceability = mean(Danceability),
              Energy = mean(Energy),
              Valence = mean(Valence),
              Liveness = mean(Liveness))
  #install.packages("scales")
  library(ggplot2)
  jimi2 <- jimi_hendrix_live_slope_try %>%
    select(livemarker, Liveness, Danceability, Energy, Valence, Acoustics) %>%
    group_by(livemarker) %>%
    mutate(Liveness = mean(Liveness),
           Danceability = mean(Danceability),
           Acoustics = mean(Acoustics),
           Valence = mean(Valence),
           Energy = mean(Energy))%>%
    select(livemarker, Liveness, Danceability, Acoustics, Valence, Energy) %>%
    group_by(livemarker, Liveness, Danceability, Acoustics, Valence, Energy) %>%
    summarise()
  jimi2 <- gather(jimi2, condition, measurement, Liveness, Danceability, Acoustics, Valence, Energy) %>%
    arrange(livemarker, condition)
  jimi2$livemarker <- as.factor(jimi2$livemarker)
  jimi2$condition <- as.factor(jimi2$condition)
  jimi2$livemarker <- ifelse(jimi2$livemarker == 1, "Live", " Studio")
  return(ggplot(data = jimi2, aes(x = livemarker, y = measurement, group = condition)) +
           geom_line(aes(color = condition), size = 1, alpha = .5) +
           geom_point(aes(color = condition), size = 2) +
           #  Labelling as desired
           labs(
             title = song
           ) +
           ylab(element_blank())+
           xlab(element_blank()) +
           theme(legend.title = element_blank(),
                 panel.grid.major = element_blank(),
                 panel.grid.minor = element_blank(),
                 panel.background = element_blank(),
                 axis.text.x = element_text(size =10),
                 title = element_text(size = 9)) +
           theme(plot.title = element_text(hjust=0.5))+
           scale_color_viridis_d()
  ) 
  
}

## loop that puts the slopes into a list

function_slope_for_each_song <- function(artist_live_data, album) {
  fz_clean <- clean_live_frame_function(artist_live_data, album)
  fz_clean_live <- fz_clean %>%
    filter(livemarker == 1)
  vector <- unique(fz_clean_live$track_name)
  vector
  song<- as.character(vector)
  song_string <- substr(song, 1,5)
  len<- length(song_string)
  len
  x <- list(1:len)
  a <- vector("list", 1)
  for (i in c(1:len)) {
    a[[i]] <- slope_graph_creater(fz_clean, vector[i])
    # assign(paste("quotient", i, sep = ""), a)
  }
  len <- length(a)
  plot <- grid.arrange(a[[1]], a[[2]] + theme(legend.position = "none", axis.text.y =  element_blank(), axis.ticks.y = element_blank()),
                       a[[3]] + theme(legend.position = "none", axis.text.y =  element_blank(), axis.ticks.y = element_blank()), a[[4]] + theme(legend.position = "none", axis.text.y =  element_blank(), axis.ticks.y = element_blank()),
                       a[[5]] + theme(legend.position = "none", axis.text.y =  element_blank(), axis.ticks.y = element_blank()), a[[6]] + theme(legend.position = "none", axis.text.y =  element_blank(), axis.ticks.y = element_blank()),
                       nrow =3, ncol = 2, top = textGrob(album, gp= gpar(fontsize = 20, font = 2)))
}

library(gridExtra)
library(grid)

frankzappa_livedata <- accumulatedlive %>%
  filter(artist_name == "Frank Zappa")
frankzappa_liveplots <- function_slope_for_each_song(frankzappa_livedata, "Roxy & Elsewhere")

```

### Citations

* [Spotify API](https://developer.spotify.com/documentation/web-api/libraries/)
* [Data Visualization](https://d3js.org/)
* [Data Cleaning](https://cran.r-project.org/web/packages/tidyverse/index.html)



