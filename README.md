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

# [Setting up the styles](https://github.com/mdhender/schloss/releases/tag/setup-styles)

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
    Watching for changes in ~/schloss/{data,content,layouts,themes,static}
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

    adding created directory to watchlist ~/schloss/static/images

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

# Create the Homepage Template

You probably know that the homepage can be built from either `layouts/index.html` or `layouts/_default/list.html`
I recommend using the first because it aligns nicely with the `content/_index.md` file that we'll be putting our homepage's content in.

To start, we're going to move the `static/index.html` file into the theme.

    $ mv static/index.html themes/schloss/layouts/index.html

Nothing seems to change in the browser because Hugo's smart enough to see that the file's moved:

    Static file changes detected
    2017-12-02 16:17:22.407 -0700
    File no longer exists in static dir, removing /index.html

    Change detected, rebuilding site
    2017-12-02 16:17 -0700
    Template changed "~/schloss/themes/schloss/layouts/index.html": REMOVE
    Template changed "~/schloss/themes/schloss/layouts/index.html": CREATE

A neat thing about the Kafka page was that it includes chunk files (e.g., `#include virtual="includes/_header.htm"`).
We're going to slice up that index file into similar chunks and even similar names.

## Create the Header Partial

A partial is just a template that lives in the `layouts/partials/` directory.
The advantage of a partial over other templates is that Hugo automatically searches for them.
(That may make more sense in a future life.)

Start by copying over the existing chunk:

    $ cp kafka-site/includes/_header.htm themes/schloss/layouts/partials/header.html

Note that I used `.html` for the extension. Hugo likes that.

Now change the index file to use that partial. In short, we're changing this:

    <!--#include virtual="includes/_header.htm" -->

to this:

    {{ partial "header.html" . }}

After we save the change, Hugo will automatically update the browser.
After glancing at the page and verifying that it still looks as expected, we'll change the next chunk.

    $ cp kafka-site/includes/_top.htm themes/schloss/layouts/partials/top.html

And update the index file from:

    <!--#include virtual="includes/_top.htm" -->

to:

    {{ partial "top.html" . }}

Then check the browswer.
It's important (even though it is very tedious) to check the browser after each update.
If you don't, you'll eventually find yourself staring at a broken page and wondering what went wrong.
Small steps actually means faster progress in the long term.

After we do the same for the footer chunk, the `themes/schloss/layouts/partials/` directory looks like:

    $ ls -l themes/schloss/layouts/partials/
    total 32
    -rw-r--r--  1 mdhender  staff  3874 Dec  2 16:39 footer.html
    -rw-r--r--  1 mdhender  staff  2671 Dec  2 16:29 header.html
    -rw-r--r--  1 mdhender  staff  3792 Dec  2 16:36 nav.html
    -rw-r--r--  1 mdhender  staff   144 Dec  2 16:34 top.html

Let's do a copy+paste to move the content to the `content/_index.md` file.

When we're done, the `themes/schloss/layouts/index.html` template looks like:

    {{ partial "header.html" . }}
    {{ partial "top.html" . }}
    <div class="content">
        {{ partial "nav.html" . }}
        {{ .Content }}
    <script>
    // Show selected style on nav item
    $(function() { $('.b-nav__home').addClass('selected'); });
    </script>
    {{ partial "footer.html" . }}

I'll be the first to admit that it looks a little odd because of the way the partials are split up.
We'll fix that later.
For now, though, congratulations are in order: you've made a working homepage template!

# Create the Article Template

As mentioned earlier, I'm going to use "article" as the archetype for pages in this theme.
(I'm likely abusing the word archetype here. It's a handy way for me to thing about pages.)

## Create the archetype

Hugo uses the archetype file when you create new content.
You don't really need to create an archetype file, but it's classy if you do.
I created `themes/schloss/archetypes/article.md` with the following content:

    +++
    title = {{ .Name | title }}
    date = {{ .Date }}
    draft = true
    +++

You'd then be able to run a command like `hugo new article/lorem.md` to create a new article.

    $ hugo new article/lorem.md
    ~/schloss/content/article/lorem.md created
    $ cat content/article/lorem.md 
    ---
    title: "Lorem"
    date: 2017-12-02T17:16:41-07:00
    draft: true
    ---

The key takeaway is that the name of the directory under `content/` is the name of the archetype.

## Create the template

Hugo offers a lot of choices for single page templates.
I like to create a template with the same name as the archetype.
That makes it easier for me to understand how the pieces fit together.

To start with, I'm going to copy one of the Kafka pages to a template.
I'm doing this so that I can incrementally add the pieces needed to make it work with Hugo.

    $ mkdir themes/schloss/layouts/article
    $ cp kafka-site/intro.html themes/schloss/layouts/article/single.html

In the command above, `article` is the archetype (sometimes just called the type or the TYPE) and single.html is the name that Hugo looks for to build single pages.

If you look at `intro.html`, you should notice that the layout looks very familiar:

    <!--#include virtual="includes/_header.htm" -->
    <!--#include virtual="includes/_top.htm" -->
    <div class="content">
    <!--#include virtual="includes/_nav.htm" -->
    <div class="right">
        <h1>Introduction</h1>
        <!--#include virtual="10/introduction.html" -->
    <!--#include virtual="includes/_footer.htm" -->
    <script>
    // Show selected style on nav item
    $(function() { $('.b-nav__intro').addClass('selected'); });
    </script>

We'll change the include chunks to partials, just like we did for the homepage template:

    {{ partial "header.html" . }}
    {{ partial "top.html" . }}
    <div class="content">
        {{ partial "nav.html" . }}
    <div class="right">
        <h1>Introduction</h1>
        <!--#include virtual="10/introduction.html" -->
    {{ partial "footer.html" . }}
    <script>
    // Show selected style on nav item
    $(function() { $('.b-nav__intro').addClass('selected'); });
    </script>

I want to point out that the original uses a Javascript snippet to show which navigation item is selected.
Hugo has a way to handle that, which we'll cover later. For now, just let it remain hard-coded.

To test this out, we'll need to create an article with some content:

    $ hugo new article/intro.md
    ~/schloss/content/article/intro.md created

    $ vi content/article/intro.md

    $ cat content/article/intro.md
    ---
    title: "Introduction"
    date: 2017-12-02T17:29:45-07:00
    draft: false
    ---

    # Introduction
    Curabitur sit amet metus sem.
    Suspendisse sollicitudin nec elit eget condimentum. Fusce facilisis lorem in nibh fringilla, vel malesuada ex euismod.

    Nulla ac purus fringilla, egestas dui nec, tristique dolor.
    Nam interdum porttitor sollicitudin. Vivamus et gravida nunc.
    Integer iaculis, nisi a rhoncus interdum, justo orci fringilla augue, vehicula lobortis erat orci in tellus.
    Mauris facilisis varius semper.
    Nullam scelerisque purus at venenatis tristique.
    Vestibulum in enim quam.
    Morbi vitae ligula imperdiet, egestas magna sed, egestas dui.
    Sed a nibh non sem fermentum pulvinar eu ut purus.

Things to note:
1.  Hugo creates new articles with `draft` set to true. You must set it to false to publish it (ok, that's a bit of a lie).
1.  Hugo sets the `title` to the name of the file. I changed it.
1.  I added some "lorem ipsum" text to test with.
1.  Hugo created the page at `http://localhost:1313/article/intro/`.

Take a look at that page. You should see a page with the expected styling, big heading of "Introduction," and no content.
You see the header because it's hard-coded into the article template.
Let's fix that template to show the article's content.

Open the `themes/schloss/layouts/article/single.html` file and look for the line with `<!--#include virtual="10/introduction.html" -->`.
That's where the original page loaded the content.
With Hugo, you load content using `{{.Content}}`, so let's replace that line:

    {{ partial "header.html" . }}
    {{ partial "top.html" . }}
    <div class="content">
        {{ partial "nav.html" . }}
    <div class="right">
            <h1>Introduction</h1>
        {{.Content}}
    {{ partial "footer.html" . }}
    <script>
    // Show selected style on nav item
    $(function() { $('.b-nav__intro').addClass('selected'); });
    </script>

Now the page looks right, except that "Introduction" is there twice.
That's because we have it in both the single page template and the article.

If you look at the original site, headings are treated inconsistently.
Sometimes they're in the main HTML file, sometimes in a child.
The site's not that big, so it's not a problem there.
I'd like to be a bit more general, so I'm going to delete the heading from the template and keep it in the content file.

After making the change, `themes/schloss/layouts/article/single.html` contains:

    {{ partial "header.html" . }}
    {{ partial "top.html" . }}
    <div class="content">
        {{ partial "nav.html" . }}
    <div class="right">
        {{.Content}}
    {{ partial "footer.html" . }}
    <script>
    // Show selected style on nav item
    $(function() { $('.b-nav__intro').addClass('selected'); });
    </script>

And that means that you've created a single page template.

# Linking to Articles from the Homepage

This theme has a navigation bar on the left with the name of all the articles.
To test this out, we'll start with two of the original pages, Perfomance and Contact.
(FWIW, I picked them because they're small.)

    $ hugo new article/contact.md
    ~/schloss/content/article/contact.md created
    macpro:schloss mdhender$ hugo new article/performance.md
    ~/schloss/content/article/performance.md created

I edited those files and changed them to Markdown format.
Then I compared the way that each looked when Hugo served them to the originals.
They were identical except for the menu (which we'll address later).

There was one other difference. The URL for this included the path:

*   http://localhost:1313/article/performance/
*   http://localhost:1313/article/contact/

The original didn't have `/article/` in it.
If we were converting a site to Hugo and SEO was important, we'd have to use slugs to fix the path.
Since this is a theme tutorial, I'm not going to worry about it.

The simplest thing that could possible work for the link is to update the homepage template and hard code the path.
That's an easy change to the `themes/schloss/layouts/partials/nav.html` file:

      <a class="b-nav__performance nav__item" href="/performance">performance</a>
      <a class="b-nav__contact nav__item" href="/contact">contact us</a>

To:

      <a class="b-nav__performance nav__item" href="/article/performance/">performance</a>
      <a class="b-nav__contact nav__item" href="/article/contact/">contact us</a>

A quick check of the browser confirms that the links work.
But it's really not a good solution.
It'd be better to have the menu built by Hugo from the actual list of articles.

Hugo has good support for menus, so let's add menu information to each article.
The entry will define the menu group, the name to use, and the relative weight for ordering.

For example, the front matter for `content/article/intro.md` will look like:

    ---
    title: "Introduction"
    date: 2017-12-02T17:29:45-07:00
    draft: false
    menu:
        articles:
            name: "introduction"
            weight: 2
    ---

After updating the articles, we'll change the `themes/schloss/layouts/partials/nav.html` file to use the menu group we created:

    <nav class="b-sticky-nav">
    <div class="nav-scroller">
        <div class="nav__inner">
        {{ range .Site.Menus.articles }}
        <a class="b-nav__home nav__item" href="{{.URL}}">{{.Name}}</a>
        {{ end }}
        <a class="btn" href="/downloads">download</a>
        <div class="social-links">
            <a class="twitter" href="https://twitter.com/apachekafka" target="_blank">@apachekafka</a>
        </div>
        </div>
    </div>
    <div class="navindicator">
        {{ range .Site.Menus.articles }}
        <div class="b-nav__home navindicator__item"></div>
        {{ end }}
    </div>
    </nav>

We use `.Site.Menus.articles` because we used `menu: articles:` in the front matter.
The `range` command that loops through all of the pages added to that menu group.

I hard coded "b-nav__intro" for the class on on all the items, so our selector won't work.
As usual, we'll worry about that later.

Another quick look at the browser shows that the menu looks pretty much like we want it to.

## Add a default menu group to the page archetype

We can add the menu group to our archetype file so that we don't have to remember to add it each time we create content.
All we have to do is add it to `themes/schloss/archetypes/article.md`.

    ---
    title: "{{ replace .TranslationBaseName "-" " " | title }}"
    date: {{ .Date }}
    draft: true
    menu:
        articles:
            name: "{{ replace .TranslationBaseName "-" " " | lower }}"
            weight: 999
    ---

Actually, that's not quite true.
When Hugo created our site, it created a file, `archetypes/default.md`.
As long as that file exists, we can't use an archetype from our theme.
Go ahead and delete it, then create a new article:

    $ hugo new article/project.md
    ~/schloss/content/article/project.md created

    $ cat content/article/project.md 
    ---
    title: "Project"
    date: 2017-12-02T21:39:00-07:00
    draft: true
    menu:
        articles:
            name: "project"
            weight: 999
    ---

Congratulations, we now have the homepage showing menu items that link to our articles.