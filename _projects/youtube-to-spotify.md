---
name: Youtube to Spotify playlist
tags: ["Python", "Youtube API", "Spotify API", "AWS Lambda"]
github_link: "https://github.com/mefchristiansen/Youtube-To-Spotify-Playlist"
date: 2020-06-22
summary: "A Python script (which is AWS Lambda deployable) that converts your Youtube liked videos into a Spotify playlist."
intro: "A Python script that converts your Youtube liked videos into a Spotify playlist. This script is deployable to AWS Lambda where it will execute on a scheduled basis so that the Spotify playlist stays up to date as new videos are liked on Youtube."
---

# Intention

I spend a lot of time on Youtube, especially watching chess videos. I also like to listen to and discover a lot of new music when I'm browsing Youtube, and wished that it would easier for me to transfer all of the music that I find on Youtube into Spotify.

Inspired by similar projects, I wanted to write a script that converts all of the music that I like on Youtube into a Spotify playlist, rather than me having to manually find and add songs on Spotify. Although similar projects do a similar thing, they do not automatically update your Spotify playlist as you continue to like Youtube music videos. I wanted to write my own script and to expand upon it further by making it deployable to AWS Lambda, where it runs on a scheduled basis so as to keep the Spotify playlist up to date as you like new Youtube music videos.

# Execution / Design Decisions

## Clients / Authentication

In order to to be able to get your liked Youtube videos and to add tracks to a Spotify playlist of yours, this script interfaces with APIs.

To request liked Youtube videos, this script uses the [Youtube API](https://developers.google.com/youtube/v3), and [Google's API Python client](https://github.com/googleapis/google-api-python-client) to make it easier to interface with the API.

To add songs to your Spotify, this script uses the [Spotify Web API](https://developer.spotify.com/documentation/web-api/) and the [Spotipy](https://github.com/plamere/spotipy) Python library.

Since this script interfaces with personal user data on both Youtube and Spotify, it requires user authentication. Once the user logs in on both Youtube and Spotify (this is done in the `setup.py` script) an access and refresh token are returned. This access token gives you access to the API and permissions to interface with the user's data for a short period of time before it expires. Once it expires, you can use the refresh token to refresh the access token to continue to have access to the API.

## Youtube DL

In order to determine if a liked video was in fact a song and to parse out the tracks title and artist, I used [Youtube DL](https://github.com/ytdl-org/youtube-dl), a command-line program to download videos from YouTube, which is embedded into this script.

## AWS Lambda

Although there are similar projects that do a similar thing, I wanted to develop it further by enabling my program to automatically update your Spotify playlist as you continue to like Youtube music videos.

The way I decided to execute this was to make the script deployable to AWS Lambda. Lambda is a serverless computing platform that executes code only when needed (it's event-driven) and you only pay for the compute time that you use. I chose to use Lambda since its really easy to set up (no need to provision a server), and this script only needs to execute on a scheduled basis, which is very easily set up using CloudWatch Events.

Two limitations of Lambda that I had to consider when writing this script include the fact that you pay for the compute time that you use as well as its limited execution time (Lambda allows functions to run for up to 15 minutes).

I of course wanted to minimize the costs of running the script, and to do that I needed to minimize the script's exectution time. The way that the Youtube API orders your liked videos is from most to least recently liked. Therefore, in order to do this, I store the id of the most recently liked Youtube video every the script is executed. Then, the following time that the script runs, it will be aware of when to stop processing videos, and will only process newly liked videos, minimizing execution time. Using the id of the liked videos is however not ideal, as unliking and reliking videos can throw off the script. What would be ideal would be the timestamp for when a video was liked, but unfortunately the Youtube API does not return that metadata. Using the id for now works for the general use case, as unliking and reliking videos is rare, and would only cause the script to not process a handful of videos.

Furthermore, due to the maximum execution time of 15 minutes, if you have above 1000 liked videos on Youtube, the script will not be able to process all the liked videos. To ensure that the script is able to get through the backlog of liked videos, I recommend that the script is run locally first before being deployed to AWS. Then, Lambda should be able to get through all your newly liked videos in less than 15 minutes (unless of course you like more than 1000 videos in an hour).

Using Lambda, I was able to schedule my script to run on an hourly basis and keep my Spotify playlist up to date as I continue to like more music videos on Youtube.

### Helper Layer

This script is of course dependent on a lot of third party libraries to run; libraries that are not natively available on Lambda. Thus, to provide the Lambda script with all of the dependencies that it requires, I chose to use [Lambda layers](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html). A Lambda layer is zipped package containing libraries, a custom runtime, or other dependencies, that your Lambda function can use. This way you can separate your Lambda function from all its dependencies when uploading it to AWS. This keeps the Lambda function deployment package small and makes code updates easier, as your function is likely a lot more dynamic than the libraries it references.

I created a custom helper layer that contains all the necessary libraries for the script run. This way, my script can easily reference all the libraries that it needs to use, and, small changes to my function do not require me to upload all dependencies every time as part of the deployment package.

# Future Work

The main limitation of this project is that the results are not always perfect. Videos that are songs may be missed, the incorrect title or artist may parsed from the video, or the song may not exist on Spotify. This is a limitation of the Youtube API, as it does not provide the title or artist name meta data in its API. This is why I have use Youtube DL, but this is not perfect, and it will miss or incorrectly parse videos.

Thus, the main point of improvement that I've identified in this project is improving its ability to correctly identify Youtube music videos and add the correct corresponding Spotify track to the playlist. This problem would be very difficult to solve completely, as not every music video is formatted the same on Youtube, and so you can't use the same method to parse the title and artist. This could be solved if there was a standardized format, or if that metadata was provided by the API, but unfortunately this is not the case. I need to think more about how to improve the results, but it would require a more sophisticated parser that is able to parse the title and artist from a video using all metadata available (i.e. title, channel, description, tags, etc.).

Furthermore, I want to improve this project by implementing a method so that the script does not need to be run locally first. This can be done by keeping track of if the entire backlog of videos has been processed, and if not,  the id of the video that was last processed. Thus, when the script runs again, it can process all of the videos after the video that was last processed (in chronological order) until it gets to the end of the backlog, before processing newly liked videos. The limitations of using id in this scenario have been described above, but should be permissible for this use case.