---
layout: post
title: "How I set up Paypal Adaptive Payment for Railsview.com"
description: ""
category: Ecommerce 
tags: [ecommerce]
---

Before you start, you need to create a developer account on [X.commerce an Ebay Company that own Paypal](https://www.x.com/) and have set up a test user in your paypal sandbox account. One for the merchant, and one for the seller. These account will be use to test this payment here. here railsview_paypal_account@gmail.com will be the sample email I will use to represent the merchant and author_paypal_account@gmail.com is the sample email I will use to represent the seller. This payment model is a chained payment which mean that there will be more than one recipient for the amount of the sale.

	class OrdersController < ApplicationController
	before_filter :assigns_gateway
	include OrderHelper
	
	include ActiveMerchant::Billing::Integrations
	
	def checkout
		# CHANGE THESE VARIABLE TO SET UP THE PAYMENT
		@theme = Theme.find(params[:theme])
		price = @theme.price
		# SET UP THE AUTHOR PROFIT PERCENTAGE HERE
		# ===========================
		authorProfitPercentage = 0.75
		# ===========================
		# DO NOT CHANGE THIS
		comission = 1 – authorProfitPercentage
		author_profit = price * authorProfitPercentage
		railsview_profit = price * comission
		# ===========================
		# The Author.Paypal Account Goes Here
		author_paypal_account = “author_paypal_account@gmail.com”
		# Marketplace.Paypal Account Goes Here
		railsview_paypal_account = “railsview_paypal_account@gmail.com”
		
		puts “COMISSION: #{comission}”
		puts “authorProfitPercentage #{authorProfitPercentage}”
		puts “AuthorProfit: #{author_profit}”
		puts “railsviewProfit: #{railsview_profit}”
		
		recipients = [{
			:email => author_paypal_account,
			# # Active this if its a split payement and both primary are false
			# :amount => author_profit,
			:amount => price,
			:primary => true},
			{:email => railsview_paypal_account,
			:amount => railsview_profit,
			:primary => false}
		]
		response = @gateway.setup_purchase(
			:return_url => “http://localhost:3000″,
			:cancel_url => “http://localhost:3000″,
			:ipn_notification_url => orders_notify_action_url,
			:receiver_list => recipients
		)
		# Notifications
		
		@gateway.set_payment_options(
			:display_options => {
				:business_name    => “YOUR_COMPANY_NAME”,
				:header_image_url => “http://cdn.yourbusiness.com/logo-for-paypal.png”
			},
			:pay_key => response["payKey"],
			:receiver_options => [
				{
					:description => "Your purchase of #{@theme.title} made by #{@theme.user.name}",
					:invoice_data => {
					:item => [
					{ :name => "Item #1", :item_count => 1, :item_price => price, :price => price},
					],
					:total_shipping => 0,
					:total_tax => 10
					},
					:receiver => { :email => “receiver1@example.com” }
					},
					{
					:description => “XYZ Processing fee”,
					:invoice_data => {
						:item => [{ :name => "Fee", :item_count => 1, :item_price => 10, :price => 10 }]
					},
					:receiver => { :email => “receiver2@example.com” }
				}
			]
		)
		
		# for redirecting the customer to the actual paypal site to finish the payment.
		redirect_to (@gateway.redirect_url_for(response["payKey"]))
	end
		
	def notify_action
		notify = ActiveMerchant::Billing::Integrations::PaypalAdaptivePayment::Notification.new(			request.raw_post)
		p “Notification object is #{notify}”
		if notify.acknowledge
		p “Transaction ID is #{notify.transaction_id}”
		p “Notification object is #{notify}”
		p “Notification status is #{notify.status}”
		end
		render :nothing => true
	end
	
	private

	def assigns_gateway
		@gateway = ActiveMerchant::Billing::PaypalAdaptivePayment.new(
		:login => “AUTHOR_PAYPAL_api1.gmail.com”,
		:password => “PAYPAL_PASSWORD”,
		:signature => “PAYPAL_SIGNATURE”,
		:appid => “PAYPAL_APP_ID”, # Get this from www.x.com
		:email => ‘author_paypal_account@gmail.com’
		)
	end
end

Ideally I would recommend to put all the sensitive information in a .yml file and add it to your .gitignore file this way when you push your project to github all the logins information remains private.	

{% include JB/setup %}
