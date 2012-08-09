---
layout: post
title: "How to build your own gems"
description: ""
category: 
tags: []
---
{% include JB/setup %}

Gems are a pretty cool way to share ruby code. I have a lot of fun building my own gems. Here some details on what I did and a useful link to learn how to do the same..

From the [RubyGem Guide](http://guides.rubygems.org/make-your-own-gem/):

Here the most important details you’ll need to make your gem:

First, start by creating an account on RubyGems.org - http://rubygems.org/sign_up

Let’s assume that your gem name is “my_totokudo”

Create your gem folder as follow:

  	$ my_totokudo
	
  	$ \lib\my_totokudo.rb
	
  	$ my_totokudo.gemspec

` my_totokudo.rb ` is in the subfolder “lib” of your folder “my_totokudo”

Your ` my_totokudo.rb ` contains the code of your gem and the gemspec usually contain the specification of your gems, here something you can copy and use. Make sure to edit it as per your need:

	Gem::Specification.new do |s|
	  s.name        = 'my_string_extend'
	  s.version     = '0.0.1'
	  s.date        = '2012-02-13'
	  s.summary     = "Short summary"
	  s.description = "Description of your gem."
	  s.authors     = ["Your Name"]
	  s.email       = ["Your@Email.here"]
	  s.homepage    = "http://twitter.com/richardsondx"
	  s.files       = ["lib/my_totokudo.rb"]
	end

Before you install and push your gem you need to make sure that the name of your gem isn’t in use already:

` $ gem list -r my_totokudo `
Then you want to make sure that you have the last version of Rubygems installed:
` $ gem update --system `
Once you have all the specification of your gem and the .gemspec file ready you can build your gem. In the folder my_totokudo where the .gemspec is located Type:
` $ gem build my_totokudo.gemspec `

Then install it locally to test it if needed:

` $ gem install ./my_string_extend-0.0.1.gem `

to test it you'll have to prompt the testing command. Depending of your code and gem, what you may need to test it may varies. To launch the prompt type: 

` $ irb --simple-prompt `

Then require your gem:

` >> require 'my_totokamo' `

Do whatever test you need to do here. To exit the prompt type "quit".
Publishing your gem is very simple. You just have to type:

` $ gem push my_totokamo-0.0.1.gem `

And follow the instruction on the screen.  

If you build exciting Gems please share it with me by sending a mention and a link to your gem on twitter @Richardsondx

