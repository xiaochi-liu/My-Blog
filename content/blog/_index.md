---
title: A Blog That Works
description: ""
author: Xiaochi Liu

show_author_byline: true
show_post_date: true
show_post_thumbnail: true
thumbnail_left: true

# for listing page layout (list, list-sidebar, list-grid)
layout: list

# for list-sidebar layout
sidebar:
  title: A Sidebar for Your Thoughts
  description: What should I write here
  author: Xiaochi Liu
  show_sidebar_adunit: false
  text_link_label: Subscribe via RSS
  text_link_url: /index.xml

# set up common front matter for all pages inside blog/
cascade:
  author: Xiaochi Liu
  show_author_byline: true
  show_comments: true
  show_post_date: true
  layout: single-sidebar # single or single-sidebar
  sidebar_left: false
  sidebar:
    text_contents_label: On this page
    show_sidebar_adunit: false # show ad container
    # text_series_label: In this series
    text_link_label: ""
    text_link_url: ""
---

** No content below YAML for the blog _index. This file provides front matter for the listing page layout and sidebar content. It is also a branch bundle, and all settings under `cascade` provide front matter for all pages inside blog/. You may still override any of these by changing them in a page's front matter.**
