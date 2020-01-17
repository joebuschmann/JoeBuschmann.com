---
layout: post
title: Customizing a Ghost Theme
date: '2014-09-26 16:26:51'
tags:
- ghost-post
---

Customizing a Ghost blog theme can be as easy as taking the default Casper theme and tweaking it to your taste. I finally got around to creating my own theme to correct some niggling annoyances with Casper. I could have downloaded a prebuilt theme from the [Ghost marketplace](http://marketplace.ghost.org/), but my changes were minor so I opted to create my own. I started by checking out the [themes documentation](http://docs.ghost.org/themes/).

To test out my new theme, I [installed the Ghost platform](http://support.ghost.org/installing-ghost-windows/). Casper is located under `/content/themes` in the install directory. You can tweak it and test it out from there.

##### Adding comments

By default, Ghost doesn't come with comments but you can include a third party comment provider like [Disqus](https://disqus.com/) by adding a snippet of HTML to `post.hbs`. There are several write-ups on how to do this, but Christoph Voigt has [the best one](http://blog.christophvoigt.com/enable-comments-on-ghost-with-disqus) I've found. You can take it one step further and separate the Disqus snippet into a partial Handlebars file and include it by adding `{% raw %}{{> disqus-post}}{% endraw %}` where you want your comments to go.

<script src="https://gist.github.com/joebuschmann/a03ca5a702ea0f4aa90a.js"></script>

##### Social links

Next, I wanted to add links to Twitter, Github, and Facebook as well as to any static pages created in Ghost. An about page is a good example of a static page.

Initially, I added simple anchor elements with the URLs hard-coded on the page; however updating them became a chore once they were on several pages. A better solution would be to mark anchor elements as social links and dynamically load the URL from a config file. Ideally this would be done on the server, but since since my blog is hosted and I don't have control over the core code, I settled on having Javascript run on the client to load the URLs. If there are better ways of doing this, please let me know.

<script src="https://gist.github.com/joebuschmann/d39967e45ed6dc12f9aa.js"></script>

<script src="https://gist.github.com/joebuschmann/fe1dbe2ab3119f7c0f31.js"></script>

The first snippet is a JSON configuration file containing key/value pairs. The key identifies the kind of link, and the value is the URL. The second snippet is Javascript that scans a page looking for anchor elements with the `social-link` attribute whose value matches a key. When one is found, it sets the URL. For example, the HTML element `<a social-link="about">about me</a>` will have its href attribute set to the URL configured for the key "about".

I placed the script in `default.hbs` so I could add social links to any page. If a URL changes, it only needs to be updated in the configuration file, and all pages will get the new value.

##### Packaging shell script

If you're running the hosted solution (Ghost Pro) like I am, you'll need to zip up the theme files and upload them on the [Ghost blog management page](https://ghost.org/). Below is a simple shell script which creates a .zip file and renames the theme to Casper-JoeBuschmann.

<script src="https://gist.github.com/joebuschmann/0598171f19d893c66a24.js"></script>

For more information on customizing Ghost themes, check out [How to Make Ghost Themes](http://docs.ghost.org/themes/).