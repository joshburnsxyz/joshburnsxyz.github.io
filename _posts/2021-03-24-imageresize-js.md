---
layout: post
title: Resizing Images without Imagemagick
subtitle: Not an april fools joke
date: "2021-03-24"
---

Recently while working on a client project I came across the need to resize a bulk of images to the same parameters. Rather than spending
hours opening them each in GIMP and spending hours repeating myself, I set off to write a script to iterate over the folder and resize each
image. To accomplish this I used the [sharp](example.com) Node.js library. Sharp provides a nice API to simply define input, action, and output.

The iteration logic ws accomplished using Node.js `fs` module. The script in its entirety is below, I've also added some comments to illustrate my
intent on certain design choices. The script is also available on gitlab [here](https://gitlab.com/joshburnsxyz/imageresize.js).

```js

// Sharp is the only 3rd-party dependency
// everything else is in-built.
const sharp = require('sharp');
const process = require('process');
const fs = require('fs');
const path = require('path');

// Define options as constants to make it easier
// to repurpose later.
const RESIZE_WIDTH = 500;
const RESIZE_HEIGHT = 500;

// argv[2] relates to the first argument after the script is passed to `node`.
// usage: node ./imageresize.js <DIRECTORY>
const DIRECTORY = process.argv[2];

fs.readdir(DIRECTORY, function(err, files){
    // if .readdir throws an error for whatever reason, simply kill
    // the process to avoid any unwanted behaviour
    if (err) {
        console.error("Could not list directory " + DIRECTORY + " ERROR: " + err);
        process.exit(1);
    }

    // This is the main iteration loop.  
    files.forEach(function(file, index){
        var filePath = path.join(DIRECTORY, file);

        // the output will simply be 'myphoto.jpg_resized' this is
        // more or less the lazy option, but it was never intended for anything
        // more than automating a one-off task anyway.
        var outputPath = path.join(DIRECTORY, file + '_resized');

        fs.stat(filePath, function(err, stat){
            if (err) {
                console.error("Could not stat file " + file + " ERROR: " + err);
                return;
            }

            if (stat.isFile()) {
                console.log("resizing '%s'", filePath);

                sharp(filePath)
                    .resize(RESIZE_WIDTH, RESIZE_HEIGHT)
                    .toFile(outputPath, function(err){

                        // If sharp can't recognize the file as a photo
                        // i.e Thumbs.db or a text-file just skip it
                        if (err) {
                            console.error(err);
                            return;
                        }
                    });

            }
        });
    });
});


```

