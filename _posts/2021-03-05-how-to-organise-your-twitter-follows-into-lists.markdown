---
layout: post
title: How to organise your Twitter follows into lists
description: Improve your Twitter experience by organising your feed by topic
date: 2021-03-05 19:20:46 +0000
published: true
categories: programming social-media data
tags: ruby
---

Twitter is a double edged sword: on one hand it gives you access to bleeding edge discussion on the most varied topics, on the other it's a hate-fuelled attention slot machine where the signal to noise ratio often makes the whole experience a waste of time at best, and positively harmful at worst.

One of the big causes of this is Twitter's own algorithm. Its workings aren't public, but whatever they are, they determine what you see and consume. One way to get around both these problems is by organising your Twitter into topic lists. Lists show you tweets in chronological order so aren't as vulnerable to the negative aspects of algorithmic curation. The problem is, we often have far too many follows for it to be practical to do this manually, but this is where we can leverage Twitter's API.

The first thing we want to do is to extract all of our followed users so we can begin organising them into the lists we want to create. The best place to do this is a Google Sheet, because we can easily assign lists to each user, and then use the Google Sheets API in combination with Twitter's to get the list of users we want to organise and put them in their designated list.

To organise our followed users into lists we need to do the following:
1. Create a Twitter app
2. Create a Google app
3. Organise users into lists
4. Run the scripts to assign users to those lists.

---
  1. Create a Twitter app

This can be done at [https://developer.twitter.com/en/portal/dashboard](https://developer.twitter.com/en/portal/dashboard). Make sure to keep the access credentials and set to read & write.
{:start="2"}
  2. Create a Google app

Go to the [Google developer console](https://console.cloud.google.com/) and create a new app. Activate the Google Sheets API and download the json file with your credentials.
{:start="3"}
  3. Organise users into lists

This is where the fun starts. We want to create a spreadsheet with the following structure so that we can organise our followed users into lists:

| ID | Name | List | Description
|-------|--------|---------|---------|
| 5367537 | john bob |  | Economics professor at an Ivy League
| 942847 | jazz |  | Being a footballer in the 80s took its toll
| 1235342790 | granny  |  | Building 12 philosophy startups. ex-@memeco

So the first thing we need to do is export this data from our twitter account into a csv file.

{% highlight ruby%}

@client =
  Twitter::REST::Client.new do |config|
    config.consumer_key = 'your consumer key'
    config.consumer_secret = 'your consumer secret'
    config.access_token = 'your access token'
    config.bearer_token = 'your bearer token'
    config.access_token_secret = 'your access token secret'
  end

def export_users(user)
  File.open('List.csv', 'w+') do |file|
    @client
      .friend_ids(user)
      .each do |f|
        file <<
          "#{f}, #{@client.user(f).name}, #{@client.user(f).description.gsub("\n", ' ')}\n"
    rescue Twitter::Error::TooManyRequests => e
      # NOTE: Your process could go to sleep for up to 15 minutes but if you
      # retry any sooner, it will almost certainly fail with the same exception.
      puts 'retrying..'
      sleep e.rate_limit.reset_in + 1
      retry
      end
  end
end

export_users('my_username')

{% endhighlight %}

Then we want to upload that CSV to a Google Sheet and fill in the list name we want each user to belong to.

{:start="4"}
4. Once we've done that we can retreive that table and work with it.

{% highlight ruby%}

# frozen_string_literal: true

require "google/apis/sheets_v4"
require "googleauth"
require "googleauth/stores/file_token_store"
require "fileutils"

OOB_URI = "urn:ietf:wg:oauth:2.0:oob"
APPLICATION_NAME = "Google Sheets API Ruby Quickstart"
CREDENTIALS_PATH = "lib/credentials.json"
# The file token.yaml stores the user's access and refresh tokens, and is
# created automatically when the authorization flow completes for the first
# time.
TOKEN_PATH = "token.yaml"
SCOPE = Google::Apis::SheetsV4::AUTH_SPREADSHEETS_READONLY

##
# Ensure valid credentials, either by restoring from the saved credentials
# files or intitiating an OAuth2 authorization. If authorization is required,
# the user's default browser will be launched to approve the request.
#
# @return [Google::Auth::UserRefreshCredentials] OAuth2 credentials
def authorize
  client_id = Google::Auth::ClientId.from_file CREDENTIALS_PATH
  token_store = Google::Auth::Stores::FileTokenStore.new file: TOKEN_PATH
  authorizer = Google::Auth::UserAuthorizer.new client_id, SCOPE, token_store
  user_id = "default"
  credentials = authorizer.get_credentials user_id
  if credentials.nil?
    url = authorizer.get_authorization_url base_url: OOB_URI
    puts "Open the following URL in the browser and enter the " \
         "resulting code after authorization:\n" + url
    code = gets
    credentials = authorizer.get_and_store_credentials_from_code(
      user_id: user_id, code: code, base_url: OOB_URI
    )
  end
  credentials
end

# Initialize the API
service = Google::Apis::SheetsV4::SheetsService.new
service.client_options.application_name = APPLICATION_NAME
service.authorization = authorize

# use your spreadsheet
spreadsheet_id = "id_goes_here"
range = "Sheet2!A:C"
response = service.get_spreadsheet_values spreadsheet_id, range
response_array = []

response.values.drop(1).each do |row|
  # Print columns A and C, which correspond to indices 0 and 2.
  unless row[2].nil?
    response_array << (row[0]).to_i
    response_array << (row[2]).to_s
  end
end

response_array.drop(0)

$hash = Hash[*response_array]

{% endhighlight %}

Finally, we can use the Twitter API to put each user in their designated list.

{% highlight ruby%}


require "twitter"
require_relative "sheets"

@client =
  Twitter::REST::Client.new do |config|
    config.consumer_key = 'your consumer key'
    config.consumer_secret = 'your consumer secret'
    config.access_token = 'your access token'
    config.bearer_token = 'your bearer token'
    config.access_token_secret = 'your access token secret'
  end

all_lists = @client.lists
$hash.each do |user_id, list_name|
  user_id_name = @client.user(user_id).name

  if all_lists.any? { |list| list.name == list_name } # if list exists
    lists = all_lists.select { |list| list.name == list_name } # get lists with the hash list name
    lists.each do |list| # for each list
      if @client.list_members("duarteosrm", list.slug).any? { |member| member.name == user_id_name }
        puts "#{user_id_name} already in #{list.name} list"
      else # unless user exists in list
        @client.add_list_member("duarteosrm", list.slug, user_id) # add user
        puts "added #{user_id_name} to existing list, #{list.name}"
      end
    end
  else # else if list does not exist
    @client.create_list(list_name) # create list
    puts "list created: #{list_name}"
    lists = @client.lists.select { |list| list.name == list_name } # get lists with the hash list name
    lists.each do |list| # for each list
      @client.add_list_member("duarteosrm", list.slug, user_id) # add user
      puts "added #{user_id_name} to new list, #{list.name}"
    end
  end
rescue Twitter::Error::TooManyRequests => e
  # NOTE: Your process could go to sleep for up to 15 minutes but if you
  # retry any sooner, it will almost certainly fail with the same exception.
  puts "retrying.."
  sleep e.rate_limit.reset_in + 1
  retry
rescue Twitter::Error::NotFound
  puts "user not found"
  next
rescue Twitter::Error::Forbidden
  puts "cannot add to lists"
  next
end
{% endhighlight %}

Bonus: I wanted to unfollow all of the users I had put in lists that didn't follow me back, so I created the following script to do so:

{% highlight ruby%}


require "twitter"
require_relative "sheets"

@client =
  Twitter::REST::Client.new do |config|
    config.consumer_key = 'your consumer key'
    config.consumer_secret = 'your consumer secret'
    config.access_token = 'your access token'
    config.bearer_token = 'your bearer token'
    config.access_token_secret = 'your access token secret'
  end

begin
  id = @client.user.id
  followers = @client.follower_ids(id)
rescue Twitter::Error::TooManyRequests => e
  # NOTE: Your process could go to sleep for up to 15 minutes but if you
  # retry any sooner, it will almost certainly fail with the same exception.
  puts "retrying.."
  sleep e.rate_limit.reset_in + 1
  retry
end

$hash.each do |user_id, list_name|
  @client.unfollow(user_id) unless followers.any? { |follower| follower == user_id }
  puts "unfollowed #{@client.user(user_id).name} unless they follow back"

rescue Twitter::Error::TooManyRequests => e
  # NOTE: Your process could go to sleep for up to 15 minutes but if you
  # retry any sooner, it will almost certainly fail with the same exception.
  puts "retrying.."
  sleep e.rate_limit.reset_in + 1
  retry
rescue Twitter::Error::NotFound
  puts "user not found"
  next
rescue Twitter::Error::Forbidden
  puts "cannot add to lists"
  next
end

{% endhighlight %}