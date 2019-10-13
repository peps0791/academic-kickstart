---
title: Analyze My Music
date: 2017-08-14
math: true
diagram: true
markup: mmark
image:
  placement: 3
  caption: 'Image credit: [**John Moeses Bauan**](https://unsplash.com/photos/OGZtQF8iC0g)'
---



## What is it?

Music is probably one of the best conversation openers out there.

*What's your song of the day?*<br>
*If you were stuck on an island and you could you have one album with you, what would that be?*<br>
*What kind of music do you listen to ?*<br>

The last question is usually the all-encompassing question that can sum up one's musical preferences. But the answers are not always unambigious.

*Depends on the mood.*<br>
*I can listen to all types of music.*<br>

Whilst there are people whose musical choices are eclectic in its truest sense, most people, even while listening to different genres and different types of music, have an inclination towards a certain type of songs, which have an underlying common set of attributes. This application aims to analyze musical preferences and find any hidden patterns.

## So, what's the information source?

Various music streaming servics such as Spotify, LastFM, Deezer expose their APIs for fetching song information such as attributes, tags, artist, album information etc. We will use the information provided by these APIs for this project.

## Technology Stack

[NodeJS](https://nodejs.org/en/) - Server side<br>
[EJS](http://www.embeddedjs.com) - Front end<br>
[D3JS](https://d3js.org) - Visualization<br>

## Work flow

![Developer Work Flow](/assets/images/analyze-music/devdiagram.png)
Developer WorkFlow 

## So far...

As expected, visualization does aid in finding patterns and the task of comparing and analyzing music becomes much more easier with the help of histograms.

![Results Visualization](/assets/images/analyze-music/results1.png)
Results Visualization

![Results Visualization](/assets/images/analyze-music/results2.png)
Results Visualization

![Results Visualization](/assets/images/analyze-music/results3.png)
Results Visualization

![Results Visualization](/assets/images/analyze-music/results4.png)
Results Visualization

![Genre Tags](/assets/images/analyze-music/results5.png)
Genre Tags



## But...

This implementation leaves a lot to be desired.

1. For starters, this implementation requires the user to manually enter the details of songs and artists in a text file for uploading. Howevver, it would be much more easier to be able to just export playlists from streaming services such as spotify, and google play music in a text file and use that for analysis.
There is an [open source project](https://github.com/watsonbox/exportify) on Github that does the job of exporting spotify playlists to a text file. A wrapper could be written for this library and integrated in our project.

2. Any kind of analysis ends with comments and remarks. In our case, we can recommend new artists and songs based on our analysis using Spotify API.

3. It would be so much better if the entire results section (graphs and  recommendations) can be exported to a PDF document for distribution.

## Conclusion

For now the project can hold on its own, but its far from over for now. There are some interesting ways in which it can be enhanced. Hope to work on the playlist export story on the coming weekend. 
Until next update!

P.S - Here's the link to the [Github Repo](https://github.com/peps0791/analyze-my-music).

