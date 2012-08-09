---
layout: post
title: "Installing Rmagick ImageMagick on Mac OS 10.7 Lion can be a PITA"
description: ""
category: 
tags: []
---
{% include JB/setup %}

Rmagick is an interface between the Ruby programming language and ImageMagick/GraphicsMagick.

In orde rot install Rmagick YOU’LL NEED ImageMagick pre-instaleld on your machine. Most tutorial will tell you that you just need to type “brew install ImageMagick” then “gem install rmagick” but this option tend to result in an error or bugs. I tried a few method and the one that ended up being my favorite one was cloning the Imagemagick package on my machine and installing it locally via ssh. Thanks to a shell script I was able to Install ImageMagick on Snow Leopard! Here are the directions: 

 

First you need to download the script. If you have git installed…

` cd ~/src git clone git://github.com/masterkain/ImageMagick-sl.git cd ImageMagick-sl sh install_im.sh `

At one point, it runs a command using sudo, so it will ask for your password. After the script has finished, ImageMagick will be installed. Now, to install the RMagick gem…

` sudo gem install rmagick  `

That’s it! 

 

All the credit goes to Andrew on StackOverflow. Thanks to his post I was able to find a working solution. 