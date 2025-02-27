+++
title = "Shortcodes cheatsheet"
date = 2025-01-05
render = false
in_search_index = false
[taxonomies]
categories = ["meta"]
tags = ["demo", "random"]
+++

This is some meta-technical page I use to store cool shortcodes I haven't found use for in my existing blog posts (yet)

<!-- more -->

Some cool features I have there:

# GIF-Suport

It's fake! I don't know why they named MPEG-4 as "gif" in docs

{{ gif(sources=["assets/cirno.mp4"], width=50)}}

# Fancy Notes

{{ note(body="
**Note:** This author is really good with generating LaTeX code with LLMs 🤭

$$ \sum\_{i=1}^{n} i = \frac{n(n+1)}{2} $$
")}}

# Some cool YouTube video

{{ youtube(id="49bwzZX5_iw", width=80) }}

# Audio File Embedding

> You've made it this far!
> > toot toot
{{ audio(source="assets/audio.mp3")}}

