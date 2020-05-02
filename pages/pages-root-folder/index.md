---
#
# Use the widgets beneath and the content will be
# inserted automagically in the webpage. To make
# this work, you have to use › layout: frontpage
#
layout: frontpage
header:
  image_fullwidth: header_unsplash_12.jpg
widget1:
  title: "Blog & Portfolio"
  url: 'http://science4u.github.io/blog/'
  image: widget-1-302x182.jpg
  text: 'In progress <em>Science4U</em> take a look in latest posts.'
widget2:
  title: "Projects"
  url: 'http://science4u.github.io/Projects'
  text: '<em>Science4U</em> 1.<br/>2.<br/>3.<br/>4.'
  video: '<a href="#" data-reveal-id="videoModal"><img src="http://science4u.github.io/images/start-video-feeling-responsive-302x182.jpg" width="302" height="182" alt=""/></a>'
widget3:
  title: "About me"
  url: 'https://science4u.github.io/aboutme'
  image: widget-github-303x182.jpg
  text: '<em>Science4U</em> is free and licensed under a MIT License.'
#
# Use the call for action to show a button on the frontpage
#
# To make internal links, just use a permalink like this
# url: /getting-started/
#
# To style the button in different colors, use no value
# to use the main color or success, alert or secondary.
# To change colors see sass/_01_settings_colors.scss ----->>>> COLOUR
#
callforaction:
  url: https://tinyletter.com/feeling-responsive
  text: Inform me about new updates and features ›
  style: alert
permalink: /index.html
#
# This is a nasty hack to make the navigation highlight
# this page as active in the topbar navigation
#
homepage: true
---

<div id="videoModal" class="reveal-modal large" data-reveal="">
  <div class="flex-video widescreen vimeo" style="display: block;">
    <iframe width="1280" height="720" src="https://www.youtube.com/watch?v=BtN-goy9VOY" frameborder="0" allowfullscreen></iframe>
  </div>
  <a class="close-reveal-modal">&#215;</a>
</div>
