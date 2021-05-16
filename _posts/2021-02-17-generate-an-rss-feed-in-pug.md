---
layout: post
title: Generate an RSS feed in Pug
subtitle: Using the Pug HTML pre-processor to generate a rolling RSS feed.
date: "2021-02-17"
---

Pug is the rebranded continuation of Jade, a HTML template pre-processor simmilar to Haml or Slim. Recently I have been working on 
[a project](https://github.com/joshburnsxyz/podseed-server) that uses Pug. Below is the view template for the RSS feed that displays
podcast episodes that uses a data object passed in at render. The data itself isn't important here, what is important is how we use this
to generate an actual RSS feed that can be sent to iTunes for podcasting. The most important part here is the `doctype xml` on line 1.

```pug

doctype xml
rss(xmlns:itunes='http://www.itunes.com/dtds/podcast-1.0.dtd', version='2.0')
  channel
    title data.title
    link data.url
    language data.language
    itunes:subtitle data.subtitle
    itunes:author data.owner_name
    itunes:summary data.description
    description data.description
    itunes:owner
      itunes:name data.owner_name
      itunes:email data.owner_email
    itunes:explicit data.explicit
    itunes:image(href="#{data.iamge}")
      itunes:category(text="#{data.category}")
      if data.episodes.length
            lastBuildDate= new Date(posts[0].publishedAt).toUTCString()
        each episode in data.episodes
            item
                title episode.title
                itunes:summary episode.description
                description episode.description
                link
                | "#{data.url}/episodes/#{episode.link}"
                enclosure(url="#{data.url}/audio/#{episode.audio_file}", type="#{episode.audio_mime}", length="1024")
                pubdate episode.publish_date
                itunes:author episode.author
                itunes:duration episode.duration
                itunes:explicit episode.explicit
                guid episode.guid

```

Using the express router we simply render this template by passing a 
data object as a local (this becomes the `data` variable used to populate the fields). 

```js
// express router
var express = require('express');
var router = express.Router();

// data
var episodes_list = require("../episodes.json");

/* GET rss. */
router.get('/rss', function(req, res, next) {
  res.render('rss', { data: episodes_list });
});

```

The episodes.json file that actually holds the data is simply required and passed through. Because the data 
is passed through from a json file when the render is called, there is no need to restart the server for the changes to take effect.

