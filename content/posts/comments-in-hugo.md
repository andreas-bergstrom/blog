---
title: "Comments in Hugo"
date: 2019-03-06T10:52:23+01:00
draft: false
toc: false
images:
tags: 
  - hugo
---

This site is powered by Hugo, a static site generator for Hugo. It sits in the same category as Jekyll (Ruby), Pelican (Python), Jigsaw (PHP), Windersmith (Node) and Wyam (.Net Core), if you're coming from any of those platforms.

I love the concept of handling content as raw markdown and keeping it version controlled with Git. But one thing I struggled to find a solution to was how to "comment out" chunks of content that shold not be rendered to HTML by Hugo, similarly to how you can comment out regular code. Using HTML-comments is not an option as that would expose my comments in the HTML-code.

It is a tricky topic to google-search as you will find a lot related to adding Disqus and other ways to implement comments in Hugo. But from what I could find most answers on forums and Stack Overflow says that since Hugo is written in Go, I should be able to use regular Go template comments like this:

{{< highlight go >}}
{{/* a comment */}}
{{< / highlight >}}

However this seems to interfere with the templating language used by Hugo as this will still output the tags and any inner content, just as raw text (see below).

What did work was to create a custom shortcode in **layouts/shortcodes/todo.html** (or whatever shortcade name you prefer) and populate it with:

{{< highlight go >}}
{{ if .Inner }}{{ end }}
{{< / highlight >}}

This is a seemingly useless shortcode that does not do anything with any content or parameters that is passed to it. Which is exactly what we want! Now we can use it in any markdown content to simulate comments:

{{< highlight html >}}
{{</* todo */>}}
Any text added here will not be rendered
- Which is great for adding todos and other notes to your posts
{{</* / todo */>}}
{{< / highlight >}}

You can read more about custom shortcodes in the [Hugo docs](https://gohugo.io/templates/shortcode-templates/#create-custom-shortcodes).

As a sidenote: if you need to display Hugo shortcodes in raw format as above, you need to use {{</*/* */*/>}} instead of {{</* */>}}:

{{< highlight go >}}
{{</*/* todo */*/>}}
{{</*/* / todo */*/>}}
{{< / highlight >}}

This way the text will be rendered raw instead of processed.