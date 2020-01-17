---
layout: post
title: Switching to Ghost
date: '2013-12-29 22:01:55'
tags:
- ghost-post
---

I had been blogging with Wordpress for over two years, and while Wordpress has some great features like the statistics and comments engine, the core writing experience was never completely smooth for me. The online editor has a tendency to get spacing wrong and reformat blocks of text on a whim. Editing with Windows Live Writer is a improvement, but doing simple things like adding snippets of code or XML was always more difficult than necessary. I had to install a plug-in to get the formatting right.

Well, this week I switched Blog providers from Wordpress to Ghost. The driver behind the move was the fact that Ghost uses Markdown as its mark-up language. I have some experience using Markdown on Github and have come to love it. Delimiting code either inline or as a block is dead simple. Lists, block quotes, headers are also easy. And if Markdown doesn't give you what you want, you can fall back to inline HTML. To make writing even easier, Ghost provides a preview window next to the editor. It really is the best writing experience on the web.

I'd been looking for a Markdown-based blog engine for a while, and after seeing Ghost in action, the decision was easy.

That's not to say that Ghost is perfect. It's far from perfect. It is still a new platform, having launched in September, and is missing a few features one would expect from a blog engine.

The first and most glaring omission is the lack of comments. If you're hosting ghost, [you can tweak it to add Disqus](http://christophvoigt.com/enable-comments-on-ghost-with-disqus/) or another third party service. If you go with the hosted solution, I'm not aware of any way of adding comments. For me, this was disappointing but something I could live with for now.

Another missing feature is Ghost doesn't allow uploading media like images or videos. I worked around this by using a separate server for media hosting.

Finally, Ghost has weak support for custom domains. I was able to get joebuschmann.com pointed to my blog correctly, but because the hosted solution doesn't support A records, I had to switch DNS services from Network Solutions to CloudFlare. CloudFlare allows the configuration of CNAME records for a root domain whereas Network Solutions does not. In the end it was a good move. The CloudFlare site is less cluttered and easier to use. The Ghost team has [a good walkthrough for adding custom domains](https://ghost.org/blogs/domains/).

Despite these limitations, I'm very excited about Ghost. The excellent Markdown editor and preview pane make the other annoyances worthwhile. Now, time to uninstall Windows Live Writer.