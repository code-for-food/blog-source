---
layout: post
title: "[Rails] Get Youtube download link"
date: 2014-11-19 08:39:56 +0700
source_name: viddl-rb
source_site: https://github.com/rb2k/viddl-rb
comments: true
categories: [youtube, rails]
Author: Hau Nguyen 
---
###Installation:
```
	gem install viddl-rb

```
###Usage:
Download a video: viddl-rb http://www.youtube.com/watch?v=QH2-TGUlwu4
<br>
Viddl-rb supports the following command line options:
```
-e, --extract-audio              Extract the audio track of the download to a separate file
-a, --abort-on-failure           Abort if one of the videos in a list fails to download
-u, --url-only                   Just display the download url without downloading the file.
-t, --title-only                 Just display the title without downloading the file.
-f, --filter REGEX               Filters a video playlist according to the regex (Youtube only right now)
-s, --save-dir DIRECTORY         Specifies the directory where videos should be saved
-d, --downloader TOOL            Specifies the tool to download with. Supports 'wget', 'curl', 'aria2c' and 'net-http'
-q, --quality QUALITY            Specifies the video format and resolution in the following way: width:height:format (e.g. 1280:720:mp4)
                                 The width, height and format may be omitted with a *.
                                 For example, to match any quality with a height of 720 pixels in any format specify --quality *:720:*
-h, --help                       Displays the help screen
```
###Library Usage:
```
require 'viddl-rb'

download_urls = ViddlRb.get_urls("http://www.youtube.com/watch?v=QH2-TGUlwu4")
download_urls.first     # => "http://o-o.preferred.arn06s04.v3.lscac ..."
```
The ViddlRb module has the following module public methods:

####get_urls_names(url)
 -- Returns an array of one or more hashes that has the keys :url which points to the download url and :name which points to the name (which is a filename safe version of the video title with a file extension). Returns nil if the url is not recognized by any plugins.

####get_urls_exts(url) 
-- Same as get_urls_names but with just the file extension (for example ".mp4") instead of the full filename, and the :name key is replaced with :ext. Returns nil if the url is not recognized by any plugins.

####get_urls(url) 
-- Returns an array of download urls for the specified video url. Returns nil if the url is not recognized by any plugins.

####get_names(url) 
-- Returns an array of filenames for the specified video url. Returns nil if the url is not recognized by any plugins.

####io=(io_object) 
-- By default all plugin output to stdout will be suppressed when the library is used. If you are interested in the output of a plugin, you can set an IO object that will receive all plugin output using this method.

###Requirements:
>* curl/wget/aria2c or the progress bar gem
>* Nokogiri
>* ffmpeg if you want to extract audio tracks from the videos
