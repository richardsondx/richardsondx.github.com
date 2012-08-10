---
layout: post
title: "Adding Google Map to your App"
description: ""
category: 
tags: []
---
{% include JB/setup %}

You'll need the gem ` Geocoder ` in order to use google api location service. At least its the one I'm using here and I like it; its pretty simple to use. Check the documentation of the gem here [GEM GEOCODER](https://github.com/alexreisner/geocoder).

There is also a railscast on it: [Railscast #273]()

The official page of the gem is [www.rubygeocoder.com](http://www.rubygeocoder.com/)

I write this blogs for coder that already have a good understanding of ruby and rails. So I do not explain every single steps in details.

This is some of the sourcecode for an app I built called Wherely. User would ask question such as "Where can I go swimming in toronto"; and people would answer location that would then be displayed on a map.

Gemfile

		gem 'geocoder'

Schema

		Answer Model Migration
		  create_table "answers", :force => true do |t|
		    t.string   "location"
		    t.string   "longitude"
		    t.string   "latitude"
		    t.integer  "post_id"
		    t.datetime "created_at"
		    t.datetime "updated_at"
		  end
		
		
		Post Model Migration
		  create_table "posts", :force => true do |t|
		    t.string   "content"
		    t.datetime "created_at"
		    t.datetime "updated_at"
		  end
		
		
		User Model Migration
		  create_table "users", :force => true do |t|
		    t.string   "provider"
		    t.string   "uid"
		    t.string   "name"
		    t.datetime "created_at"
		    t.datetime "updated_at"
		  end
		end


views > posts

		<% title "Wherely ##{@post.id}" %>
		<p>
		  <strong>What I want to do:</strong>
		  <%= @post.content %>
		</p>
		<hr>
		<p>
		  <%= link_to "Edit", edit_post_path(@post) %> |
		  <%= link_to "Destroy", @post, :confirm => 'Are you sure?', :method 		=> :delete %> |
		  <%= link_to "Go Back", posts_path %>
		</p>
		<hr>
		<h2>Where you can do it:</h2>
		<%= form_for([@post, @post.answers.build]) do |f| %>
		  <div class="field">
		   Share a location:<br />
		    <%= f.text_field :location %>
		  </div>
		  <div class="actions">
		    <%= f.submit %>
		  </div>
		<% end %>
		<% @post.answers.each do |post| %>
		  <% unless post.id.nil? %>
		  <p>
		    <b>Map:</b>
		   <%= image_tag "http://maps.googleapis.		com/maps/api/staticmap?size=350x350&sensor=false&zoom=16&		markers=#{post.latitude}%2C#{post.longitude}" %>
		  </p>
		  <p>
		    <b>Answer:</b>
		    <%= post.location%>
		  </p>
		  <%= link_to " ↑ Destroy this answer #{post.id}", [@post, post], 		:confirm => 'Are you sure?', :method => :delete %>
		  <% end %>
		<% end %>

models > answer

		class Answer < ActiveRecord::Base
		  validates :location, :presence => true
		
		  attr_accessible :location, :longitude, :latitude, :post_id
		  belongs_to :posts
		
		  geocoded_by :location
		  after_validation :geocode
		end

models > post

		class Post < ActiveRecord::Base
		  validates :content, :presence => true, :length => { :minimum => 5 }
		  attr_accessible :content
		  has_many :answers
		end


controllers> answers_controller.rb

		#Answer is nested under Post model
		
		class AnswersController < ApplicationController
		  def index
		    @post = Post.find(params[:post_id])
		    @answers = @post.answers
		  end
		
		  def show
		    @post = Post.find(params[:post_id])
		    @answer = @post.answers.find(params[:id])
		  end
		
		  def new
		    @post = Post.find(params[:post_id])
		    @answer = @post.answers.new
		  end
		
		  def create
		    @post = Post.find(params[:post_id])
		    @answer = @post.answers.new(params[:answer])
		    if @answer.save
		      redirect_to [@post, @answer], :notice => "Successfully created 		answer."
		    else
		      render :action => 'new'
		    end
		  end
		
		  def edit
		    @post = Post.find(params[:post_id])
		    @answer = @post.answers.find(params[:id])
		  end
		
		  def update
		    @post = Post.find(params[:post_id])
		    @answer = @post.answers.find(params[:id])
		    if @answer.update_attributes(params[:answer])
		      redirect_to [@post, @answer], :notice  => "Successfully 		updated answer."
		    else
		      render :action => 'edit'
		    end
		  end
		
		  def destroy
		    @post = Post.find(params[:post_id])
		    @answer = @post.answers.find(params[:id])
		    @answer.destroy
		    redirect_to post_answers_url(@post), :notice => "Successfully 		destroyed answer."
		  end
		end


That's it! Let me know if you have any questions. It's pretty simple to use and I wanted to show how here.
