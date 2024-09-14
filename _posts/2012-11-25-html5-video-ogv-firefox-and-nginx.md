---
layout: post
pageId: 1
title: HTML5 video, ogv, firefox and nginx
date: 2012-11-25 16:50:10
comments: true
categories:
  - nginx
  - html video
  - ogv
tags:
  - nginx
  - html video
  - ogv
author: Luis Mendes
redirect_from:
  - /news/html5-video-ogv-firefox-and-nginx
---
If you are having problems serving `ogv` video content on Firefox there are some steps that you should follow.

## Serving ogg files with the correct MIME type

First you have to ensure that you are serving the correct `mime type` for `ogv` files, it should be `video/ogg` you can test it using [web sniffer](http://web-sniffer.net/) (see the return value in the __Content-Type:__ tag, you should see `video/ogg`)

If not try adding the following `mime types` to your `nginx` (eg: /etc/nginx/mime.types`) and restart it.

```bash
video/ogg   ogm;
video/ogg   ogv;
video/ogg   ogg;
```

## Handle HTTP 1.1 byte range requests and serving `X-Content-Duration` headers

Firefox or better yet the Gecko engine that powers Firefox / Thunderbird / SeaMonkey and others uses a sequence of HTTP 1.1 byte-range requests to retrieve the media from the seek target position in order to determine the duration of the media, it does this because ogv files don&#39;t contain the duration of the animation, so in order to determinate the duration of the file it retrieves a 8k chuck in the beginning of the file to determinate if the file is a valid ogg file, than based on the <i>Content-Length</i> header sent from the server it asks for the last chuck as you can see here, and finally it resumes the download from the first chuck as you can see in this image.

<img alt="print screen partial content" src="/assets/images/firefox_partial_content_ogv.png" />

In order to reduce the amount of requests made to your web server you can send the <code>X-Content-Duration</code> header with the duration in seconds of the media. If your video has a duration of 1 minute and 32.6 seconds you should server the value as 92.6

`X-Content-Duration: 92.6` Here is example how you can serve a media file using `nginx`

```bash
location /media/video.ogv {
  add_header X-Content-Duration "92.6";
}
```

If you&#39;re curious to see in greater detail why Gecko makes some of the HTTP requests it does, read <a href="https://bugzilla.mozilla.org/show_bug.cgi?id=502894#c1" target="_blank">comment 1 of bug 502894</a>.

## Consider using <code>preload</code> attribute

The HTML5 <code>video</code> and <code>audio</code> tags provide the <code>preload</code> attribute which tell Gecko to attempt to download the entire media when the page loads. Without the <code>preload</code> attribute Gecko only downloads enough of the media to display the first video frame and determinate the media&#39;s duration.

`preload` if <i>off</i> by default so to enable just add the attribute to the video tag and set it&#39;s value to <i>auto</i>.

```html
<video preload="auto">
  <source src="media/video.ogv" type="video/ogg" />
</video>
```
