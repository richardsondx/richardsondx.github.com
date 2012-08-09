---
layout: post
title: "Rake Task Omniauth Identity + carrierwave"
description: ""
category: Rake
tags: []
---
I often use `rake db:populate` in development mode to populate my database with random data. It’s a useful too to test your website and see how it would look like when you have a lot of users. It’s also extremely useful for testing paginations. On Railsview.com I build a script that auto generate random user using the Omniauth Identity gem and then generate random user’s theme that each have an image using the carrierwave gem.

	namespace :db do
		require ‘Faker’
		require ‘omniauth’
		desc “Fill database with sample data”
		task :populate => :environment do
			Rake::Task['db:reset'].invoke
		make_users
		end
	end
	
	def make_users
	
	20.times do |n|
		name = Faker::Name.name
		email = “test-#{n+1}@test.com”
		password = “password”
		
		# CREATE USERS: Populate the database with random users
		Identity.create! do |user|
			user.name = name
			user.email = email
			user.password_digest = BCrypt::Password.create(password)
		end
	
		hash = OmniAuth::AuthHash.new(
			:info => {
			:name => name,
			:email => email,
			}
		)

		user = User.create_from_omniauth(hash)
		
		# CREATE THEMES: Populate the database with random themes that each users own.
		theme = user.themes.new
		@uploader = ImageUploader.new(theme, :image)
		
		2.times do |a|
			@uploader.cache!(File.open(“app/assets/images/sample_themes/sample#{rand(1..10)}.png”))
			theme = user.themes.create!(
				:title => “#{a}Capadona#{n+1}”,
				:image => @uploader,
				:price => rand(1..100),
				:description => “lorem ipsum efeeffaf”
			)
		end
	end
end

{% include JB/setup %}
