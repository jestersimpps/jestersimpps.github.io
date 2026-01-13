---
layout: post
title: "Decentralized Microservices With IPFS"
description: "Exploring how to combine IPFS distributed file system with microservices architecture to create robust, decentralized applications."
date: 2017-10-07
author: "Jester Simpps"
categories: [Development]
tags: [IPFS, microservices, decentralized, blockchain, distributed-systems]
image: /images/ipfs.jpg
---

## What is IPFS?

IPFS (the InterPlanetary File System) is a distributed file system that seeks to connect all computing devices with the same system of files. In some ways, this is similar to the original aims of the Web, but IPFS is actually more similar to a single bittorrent swarm exchanging git objects. (You can read more about its origins in the paper [IPFS - Content Addressed, Versioned, P2P File System.](https://github.com/ipfs/ipfs/blob/master/papers/ipfs-cap2pfs/ipfs-p2p-file-system.pdf?raw=true))

<iframe width="560" height="315" src="https://www.youtube.com/embed/8CMxDNuuAiQ" frameborder="0" allowfullscreen=""></iframe>

## What are microservices?

Microservices - also known as the microservice architecture - is an architectural style that structures an application as a collection of loosely coupled services, which implement business capabilities. The microservice architecture enables the continuous delivery/deployment of large, complex applications. It also enables an organization to evolve its technology stack.

![ipfs2](https://ipfs.io/ipfs/QmaCjhDr2378r9iPqEjUu4vXavmRK3JEtf5nKUxSrXpQBc)

## What makes the combination of IPFS and microservices interesting?

The nice thing about IPFS is it's immutable storage. IPFS works like a github repository. Just like commit hashes, IPFS hashes will always point to the same immutable files. This makes IPFS an interesting platform to develop microservices. Similar to dependency versioning in npm (node packaging manager), you know for sure that the code at a certain hash will always do the same thing. Secondly, these services are not hosted by a single source (centralized), but are hosted worldwide on several locations (distributed), which makes them more robust. They will remain available at all times.

![ipfs1](https://ipfs.io/ipfs/QmZpfhN5rucQC4kx7Gu5udS8FXxsxMddiDCvG8WFjG8SMv)

## Storing files on IPFS

Before creating an IPFS microservice, let's store a simple website on IPFS. You probably haven't noticed, but the image above was actually uploaded to the IPFS and downloaded using an IPFS gateway: [https://ipfs.io/ipfs/](https://ipfs.io/ipfs/). This gateway allows you to load decentralized files stored on IPFS in your web browser, which is pretty cool and opens up amazing possibilities regarding web apps.

### Uploading a static website to IPFS / IPNS

(*To follow this tutorial, please [install](https://ipfs.io/docs/install/) IPFS first.*)

Besides simple files, you can also store entire directories to IPFS.

```bash
$ ipfs add -r .
```

So, I uploaded a simple static website I created a while ago onto IPFS and it can be accessed using the directory hash `QmUyif9MLjMzpFeXvjLJiatCKsQJhykV1oZH7GPuXRcJjo` like so:

[https://ipfs.io/ipfs/QmUyif9MLjMzpFeXvjLJiatCKsQJhykV1oZH7GPuXRcJjo/en/](https://ipfs.io/ipfs/QmUyif9MLjMzpFeXvjLJiatCKsQJhykV1oZH7GPuXRcJjo/en/)

Now I have a simple static site hosted on IPFS. The problem is, however, when I update my site, the hash will change, and any links I have shared will continue pointing to the old version.

That's where IPNS comes in. It allows you to store a reference to an IPFS hash under the namespace of your IPNS hash or peerID.

```bash
$ ipfs name publish QmUyif9MLjMzpFeXvjLJiatCKsQJhykV1oZH7GPuXRcJjo
```

That will return your peerID and the hash you are publishing to it. A few seconds later my terminal spits back the following IPNS hash:

```bash
Published to QmaJLU8Jb2NmpHsgCGu3EqawjqFB23FfXHvt3kWted1PdQ:/ipfs/QmUyif9MLjMzpFeXvjLJiatCKsQJhykV1oZH7GPuXRcJjo
```

My website is now accessible through:

[https://ipfs.io/ipns/QmaJLU8Jb2NmpHsgCGu3EqawjqFB23FfXHvt3kWted1PdQ/en/](https://ipfs.io/ipns/QmaJLU8Jb2NmpHsgCGu3EqawjqFB23FfXHvt3kWted1PdQ/en/)

Notice the `ipns` in the url, instead of `ipfs`. Every time I update my website, IPNS will make sure anyone accessing the IPNS will get the hash of my latest site.

## Creating an IPFS Microservice

### IPFS and decentralized services

If we can upload websites, we can add javascript code to those websites that looks at the url and extracts additional IPFS hashes. Those can be used to access other files saved on IPFS. This allows us to create services or applications on IPFS that accept other IPFS files as inputs and do something with those.

A nice example of such a service is [this IPFS video player](https://ipfs.io/ipfs/QmVc6zuAneKJzicnJpfrqCH9gSy6bz54JhcypfJYhGUFQu/play#/ipfs/QmTKZgRNwDNZwHtJSjCp6r5FYefzpULfy37JvMt9DwvXse).

`https://ipfs.io/ipfs/QmVc6zuAneKJzicnJpfrqCH9gSy6bz54JhcypfJYhGUFQu/play#/ipfs/QmTKZgRNwDNZwHtJSjCp6r5FYefzpULfy37JvMt9DwvXse`

The entire code of the video player can be found [here](https://ipfs.io/ipfs/QmVc6zuAneKJzicnJpfrqCH9gSy6bz54JhcypfJYhGUFQu).

```javascript
var tf = $('#input');

function hash() {
  return window.location.hash.substring(1)
}

function render(type, path) {
  var video = $('<video>')

  video
    .attr("id", "videoid")
    .attr("width", "100%")
    .attr("height", "100%")
    .attr("controls", "controls")
    .attr("poster", path + "/poster.jpg")
    .attr("preload", "auto")
    .attr('data-setup', '{"example_option":true}')
    .addClass('video-js vjs-default-skin vjs-big-play-centered')

  if (type == "file") {
    $("<source>")
      .attr("src", path)
      .appendTo(video)
  } else {
    var formats = ['mp4', 'webm', 'ogv']
    formats.forEach(function (fmt) {
      $("<source>")
        .attr("type", "video/" + fmt)
        .attr("src", path + "/video." + fmt)
        .appendTo(video)
    })
  }

  $("div#video-div").innerHTML = ""
  $("div#video-div").append(video)

  videojs("videoid").ready(function(){
    this.play();
  });
}

var last = ""
function update(path) {
  if (last == path)
    return
  last = path

  $.get(path, function() {
    render("dir", path)
  }).fail(function() {
    render("file", path)
  })

  tf.val(path)
  window.location.hash = "#" + path
  console.log("updated to: " + path)
}

function isFile(path) {
    return path.split('/').pop().split('.').length > 1;
}

(function main() {
  tf.bind('keyup keypress blur change cut copy paste', function(event) {
    update(tf.val())
  })

  tf.bind('hashchange', function(event) {
    update(hash())
  })

  var text = hash()
  if (text.length < 1)
    text = ""
  update(text)
})()
```

If we take a look at the code, we can see that

`window.location.hash.substring(1)`

extracts the following string from the url: `/ipfs/QmTKZgRNwDNZwHtJSjCp6r5FYefzpULfy37JvMt9DwvXse`

This hash is then used to fetch the [input file](https://ipfs.io/ipfs/QmTKZgRNwDNZwHtJSjCp6r5FYefzpULfy37JvMt9DwvXse) for the video. If we would create another video file, add it to IPFS and the new hash to the video player url, the video player will act as a service to play that other file.

### Let's build a simple text to speech app

The following code is a simple webapp that reads out the url and speaks the text defined in the `say` query parameter.

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <title>text to speech render</title>
    <script src="lib/jquery.min.js"></script>
</head>

<body>
    <h1 id="text"></h1>
    <script>
        function speak(text) {
            var msg = new SpeechSynthesisUtterance();
            $('#text').text(text);
            msg.text = text;
            speechSynthesis.speak(msg);
        }

        (function main() {
            $(function() {
                if ('speechSynthesis' in window) {
                    var text = decodeURIComponent(window.location.search.split('say=')[1]);
                    speak(text);
                } else {
                    window.alert('no text to speech browser support.')
                }
            });
        })()
    </script>
</body>

</html>
```

It was uploaded to IPFS and can be accessed using the following link:

[https://ipfs.io/ipfs/QmPdWu3ko1qAwKJe83itC2UuAzRoKjnpQy8a1p4YYJDECr/?say=hello%20world](https://ipfs.io/ipfs/QmPdWu3ko1qAwKJe83itC2UuAzRoKjnpQy8a1p4YYJDECr/?say=hello%20world)
