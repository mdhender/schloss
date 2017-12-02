# Converting to a Hugo theme

This tutorial copies the theme from Apache's [Kafka](https://kafka.apache.org/) web site.

## Licenses

Please check the license for the site that you're going to copy from to make sure that you're allowed to.
If you look at the site for Kafka, you'll find a copyright and license in the footer:

    The contents of this website are © 2016 Apache Software Foundation under the terms of the Apache License v2.

# Preliminary Steps

I started by setting up a new Hugo site on my computer:

    $ hugo new site schloss
    $ cd schloss
    $ hugo new theme schloss

I created a LICENSE file (naturally, I used the [Apache License v2](https://www.apache.org/licenses/LICENSE-2.0.html)), along with a NOTICE.md to acknowledge the source.

Before starting, I cloned the Kafka site to my computer.
It's faster to work with a local copy, plus it reduces the bandwidth useage on the source.
They're nice enough to make their source available, so it's good to be kind in return.

I created a new Git repository on my computer (version control is a good thing).
I added my Kafka clone to the `.gitignore` file since it was already under version control.

Finally, I commited everything.