---
layout: post
title: "How I set up Paypal Adaptive Payment for Railsview.com"
description: ""
category: Ecommerce 
tags: [ecommerce]
---

Before you start, you need to create a developer account on [X.commerce an Ebay Company that own Paypal](https://www.x.com/) and have set up a test user in your paypal sandbox account. One for the merchant, and one for the seller. These account will be use to test this payment here. here railsview_paypal_account@gmail.com will be the sample email I will use to represent the merchant and author_paypal_account@gmail.com is the sample email I will use to represent the seller. This payment model is a chained payment which mean that there will be more than one recipient for the amount of the sale.

		class OrdersController < ApplicationController
		  include OrderHelper
		  include ActiveMerchant::Billing::Integrations   
		
		  before_filter :get_buyer_info, :get_theme, :initializer
		  skip_before_filter  :verify_authenticity_token, :only => :notify_action
		
		  def initializer
		      app_paypal_email("APP_PAYPAL_EMAIL_HERE") # example_biz_api1.gmail.com"
		      setup_seller_profit(75)
		      setup_receivers("PRIMARY_RECEIVER_HERE", "SECONDARY_RECEIVER_HERE")
		      get_buyer_info
		      get_theme
		      assigns_gateway
		  end
		
		  def checkout
		    initializer
		  
		    recipients = [{:email => @seller,
		                   # # Active this if its a split payement and both primary are false
		                   # :amount => @author_profit,
		                   :amount => @theme.price,
		                   :primary => true},
		                  {:email => @railsview,
		                   :amount => @railsview_profit,
		                   :primary => false}
		                   ]
		    response = @gateway.setup_purchase(
		                :return_url => "http://localhost:3000",
		                :cancel_url => "http://localhost:3000",
		                :ipn_notification_url => orders_notify_action_url(
		                  :theme => @theme, 
		                  :current_user => @current_user
		                  ),
		                :receiver_list => recipients
		                )
		  
		    setup_billing_information
		
		    redirect_to (@gateway.redirect_url_for(response["payKey"]))
		  end
		
		  def notify_action
		    notify = ActiveMerchant::Billing::Integrations::PaypalAdaptivePayment::Notification.new(		request.raw_post)
		    p "Notification object is #{notify}"
		    if notify.acknowledge
		      get_notification
		    end
		
		    def get_notification
		      p "Transaction ID is #{notify.transaction_id}"
		      p "Notification object is #{notify}"
		      p "Notification item_id is #{notify.item_id}"
		      p "Notification amount is #{notify.amount}"
		      p "Notification status is #{notify.status}"
		      p "Notification invoice is #{notify.invoice}"
		
		      @notification = OrderNotification.create(
		      :status => notify.status,
		      :txn_id => notify.transaction_id,
		      :amount => notify.amount,
		      :item_id => "@theme.user.name/Theme_#{@theme.id}",
		      :invoice => notify.invoice
		      )
		
		      p "**NOTICE: Istant Payment Notifications details were succesfuly added to @notifiation"
		
		      if @notification.status == "COMPLETED"
		        is_completed
		      else
		        puts "The transaction wasn't completed"
		      end
		    end
		
		    def is_completed
		
		        # Add details to the purchase table
		          @current_user = User.find(params[:current_user])
		          @purchased = Purchase.create!(
		          :author => @theme.user.name,
		          :buyer => @current_user.name,
		          :theme => "#{@theme.title}# id:#{@theme.id}",
		          :price => @theme.price,
		          :author_profit => @author_profit,
		          :railsview_profit => @railsview_profit
		        )
		
		        print_purchase_details(@author_profit)
		        increment_theme_purchase(@theme.id)
		    end
		
		    render :nothing => true
		  end
		
		  def print_purchase_details(author_profit)
		    puts "AUTHOR PROFIT: #{author_profit}"
		    puts "The details were added to the Purchase table:"
		    puts @purchased.to_yaml
		  end
		
		  def setup_billing_information
		    @gateway.set_payment_options(
		      :display_options => {
		        :business_name    => "Railsview.com || #{@theme.title}",
		        :header_image_url => "http://cdn.yourbusiness.com/logo-for-paypal.png"
		      },
		    :pay_key => response["payKey"],
		    :receiver_options => [
		      {
		        :description => "Your purchase of #{@theme.title} made by #{@theme.user.name}",
		        :invoice_data => {
		          :item => [
		            { :name => "#{@theme.title} - #{@theme.id}", :item_count => 1, :item_price => @theme	.	price, :price => @theme.price},
		          ],
		          :total_shipping => 0,
		          :total_tax => 10
		        },
		        :receiver => { :email => @seller }
		      },
		      {
		        :description => "XYZ Processing fee",
		        :invoice_data => {
		          :item => [{ :name => "Fee", :item_count => 1, :item_price => 10, :price => 10 }]
		        },
		        :receiver => { :email => @railsview }
		      }
		    ]
		    )
		  end
		
		  private
		
		    def assigns_gateway
		      @gateway = ActiveMerchant::Billing::PaypalAdaptivePayment.new(
		        :login => @app_paypal,
		        :password => "=PASSWORD",
		        :signature => "APPLICATION_SIGNATURE",
		        :appid => "APPLICATION_ID",
		        # :email => "#{current_user.paypal}"
		        :email => @seller
		      )
		    end
		
		    def get_buyer_info
		      @current_user = User.find(params[:current_user])  
		    end
		
		    def get_theme
		      @theme = Theme.find(params[:theme])
		    end
		
		    def setup_seller_profit(profit_in_percentage)
		      seller_profit = profit_in_percentage/100
		      comission = 1 - seller_profit
		      @railsview_profit =  @theme.price * comission
		      @author_profit = @theme.price * seller_profit
		    end
		  
		    def print_profit_rate
		      puts "COMISSION: #{comission}"
		      puts "authorProfitPercentage #{seller_profit}"
		      puts "AuthorProfit: #{@author_profit.to_f}"
		      puts "railsviewProfit: #{@railsview_profit}"
		    end
		
		    def setup_receivers(primary_receiver, secondary_receiver)
		      @seller = primary_receiver
		      @railsview = secondary_receiver
		    end
		
		    def app_paypal_email(email)
		      @app_paypal = email
		    end
		
		    def increment_theme_purchase(theme_id)
		      Theme.increment_counter(:purchased, theme_id)
		    end
		
		end

Ideally I would recommend to put all the sensitive information in a .yml file and add it to your .gitignore file this way when you push your project to github all the logins information remains private.	

{% include JB/setup %}
