---
layout: post
title: Moving Gems from one version of Ruby Enterprise Edition to Another
permalink: /2008/12/28/moving-gems-from-one-version-of-ree-to-another.html
tags: ["ruby", "ree"]
comments: yes
---

As mentioned in my previous post I recently built a small internal micro app with merb.  As part of the process of deploying that app I needed/wanted to update to the latest version of Ruby Enterprise Edition (REE) and Passenger on my slice.  One of the issues I ran into while trying to update the REE version is that all my old gems where not installed in my fresh new version of REE.  There may be a better way to accomplish this task, but the approach I ended up using was to modify this capistrano file (<a href="http://github.com/jtimberman/ree-capfile/tree/master">http://github.com/jtimberman/ree-capfile/tree/master</a>) to install the gems in the old version of REE in the new version. 

<script src="http://gist.github.com/39066.js"></script>
