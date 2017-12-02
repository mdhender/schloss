# Converting to a Hugo theme

This tutorial copies the theme from Apache's [Kafka](https://kafka.apache.org/) web site.

## Licenses

Please check the license for the site that you're going to copy from to make sure that you're allowed to.
If you look at the site for Kafka, you'll find a copyright and license in the footer:

    The contents of this website are © 2016 Apache Software Foundation under the terms of the Apache License v2.

# [Preliminary Steps](https://github.com/mdhender/schloss/releases/tag/preliminary-steps)

I started by setting up a new Hugo site on my computer:

    $ hugo new site schloss
    $ cd schloss
    $ hugo new theme schloss

I created a LICENSE file (naturally, I used the [Apache License v2](https://www.apache.org/licenses/LICENSE-2.0.html)), along with a NOTICE.md to acknowledge the source.

Next, I cloned the Kafka site to my computer.
(I put the clone in my site's folder, in a sub-directory named `kafka-site`.)
It's faster to work with a local copy, plus it reduces the bandwidth useage on the source.
(They're nice enough to make their source available, so it's good to be kind in return.)

I created a new Git repository on my computer (version control is a good thing).
I added my Kafka clone to the `.gitignore` file since it was already under version control.

Finally, I commited everything.

# Take a quick survey

Before you start building the theme in Hugo, take a moment to look at the original and determine how many templates you'll need.

Generally speaking, the home (or landing) page will have its own template.

The remainder of the pages on the Kafka site all have a similar style, so they'll share another template.

## Archetypes

Hugo allows you to create `archetype` files to simplify creating content.
I'm going to call my main archetype an article since the Kafka site doesn't feel like a blog.

# Setting up the styles

The nice thing about cloning an existing theme is that all the hard work of creating and testing the styles has been done for you.
You want to make sure that you've gotten all the CSS and script files that you need.

## Set up your config file

Things will be simpler if you tell Hugo to use your new theme as the default for the site.
For this theme, that meant adding a line to my `config.toml` file:

    theme = "schloss"

That tells Hugo to look in `themes/schloss` for files before looking anywhere else.

## Copy in the CSS files

All of the CSS files should be copied the theme's style folder.

Luckily, there was only one folder with CSS, so I was able to run a simple copy command:

    $ cp kafka-site/css/* themes/schloss/static/css/

## Test the styles

To test, I copied the original `index.html` file to my site's `static/` folder:

    $ cp kafka-site/index.html static/

Items in the site's `static/` folder are used as-is when Hugo serves up a site.
Note that your site has a static folder and so does your theme.
The rule of thumb is that Hugo copies files from your theme first, then from your site.
That means that the files in your site's folder take precedence.

I ran Hugo to serve up the site:

    $ hugo serve
    Started building sites ...

    Built site for language en:
    0 draft content
    0 future content
    0 expired content
    0 regular pages created
    6 other pages created
    0 non-page files copied
    0 paginator pages created
    0 categories created
    0 tags created
    total in 7 ms
    Watching for changes in /Volumes/ssda/mdhender/Software/go/src/github.com/mdhender/schloss/{data,content,layouts,themes,static}
    Serving pages from memory
    Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
    Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
    Press Ctrl+C to stop

Then opened up the site in my browser.
All the text was there, but none of the styling.
(Honestly, that's what I expected to see since you usually need to change links when cloning a theme.)

The usual cause for missing styling is forgetting to copy in CSS or script files.
I used Chrome's developer tools to check.
Missing files and scripts show up on that tab with an HTTP Status of 404 (well, anything but 200), so I opened up the Network tab and refreshed the page.
There was one network error for an image file (`kafka_diagram.png`), so I copied that over to my site:

    $ mkdir -p static/images
    $ cp kafka-site/images/kafka_diagram.png static/images/

One of the nice features with Hugo is the live refresh.
Hugo picked up that I added a new directory with a new image in it.

    adding created directory to watchlist /Volumes/ssda/mdhender/Software/go/src/github.com/mdhender/schloss/static/images

    Static file changes detected
    2017-12-02 14:55:36.699 -0700
    Syncing images to /

I now had the picture on the page, but the styling was still missing.

Going back to the Network tab in the browser, I confirmed that there were no more issues with missing files.
That meant that the next thing to check was the links inside of the `index.html` file that I'd copied over.

A quick look at `static/index.html` showed the problem: the file didn't contain an HTML header.
Instead, it contained directives for the Apache web server to dynamically include template files:

    <!--#include virtual="includes/_header.htm" -->
    <!--#include virtual="includes/_top.htm" -->

Templates are something that Hugo excels at, but my goal here is to confirm that I've got all the files and scripts that I need.
To do that, I copied the contents of all of those files into my `static/index.html` file and then went back to the browser to compare.

After I did that, the page looked as I expected it to.
There were still some missing images, but it was quick work to copy them in to the `static/images/` folder.

I double checked the Network tab and noticed that there several Javascript libraries that needed to be copied, too.
I copied them to the theme folder:

    $ cp kafka-site/js/jquery.min.js themes/schloss/static/js/

Note that I placed the scripts in the theme's directory while I placed the images in the site's directory.
The scripts must go in the theme's directory because they actually belong to the theme.
I didn't do that with the images because I planned on replacing them with placeholders.
It doesn't make sense for my theme to use the Apache and Kafka logos (it might confuse visitors).

Instead, I used a drawing program to create images with the same name, dimensions, and resolution as the originals.
I put those files in to the `themes/schloss/images/` folder.

    $ mkdir themes/schloss/static/images
    $ mv static/images/* themes/schloss/static/images/

Later on I'll put better images in there. For now, though, the placeholders will work just fine.
They'll be a good reminder that users need to add their own to their site's directory to override them.
