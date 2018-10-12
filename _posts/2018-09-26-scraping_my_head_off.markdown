---
layout: post
title:      "Scraping my head off "
date:       2018-09-26 11:14:37 -0400
permalink:  scraping_my_head_off
---


When html is pain. How can html be pain? It's so simple right?? CSS selectors? Baah it's written right there what they do.. how can this be something convolute?

Well, lemme tell you about the process of scraping my favourite news website and how scraping data doesn't always go well...


# Planning the Gem
My "wishlist" was the first step. I wanted to grab the headlines shown on the main page - the website has 12 headlines containing varied monday to friday stories (going back 2.5 weeks, 12 weekdays).
Then I wanted to just select that weekday's headlines - For ex there are 3 tuesdays to choose from. Finally open the selected Tuesday and show all headlines and all stories in the right order.

Simple enough right?

This is the main website:

https://www.wired.co.uk/topic/wired-awake 

![](https://res.cloudinary.com/melzreal/image/upload/v1537964530/Capture_z1dtid.jpg)




The purpose of the project being to build a gem with a cli that allows the user to interact with then select the stories as required. This meant the first step was to create a gem, so using bundler:

bundle gem wired_awaker


And magically all my files appear!

Off the bat a first problem appeared. "wired_awaker.gemspec"  spec.summary cannot be "TODO", something of the like.. after modfying the file and saving, that's one problem down.

On to the next.

The first step was creating the executable within wired_awaker/bin/wired_awaker
This will contain a method I haven't made yet, and require 2 dependencies:

*
require "bundler/setup"
require "wiredawaker"


WiredAwaker::CLI.new.fetcher*


Then, within the lib folder, where the code for the programme to be executed will to, create a cli that will be called by my executable file.
*
lib/wiredawaker/cli.rb

class WiredAwaker::CLI

 def fetcher
     puts "testing cli"
end
end*

To ensure that my CLI can be "seen" by the rest of my programme, I need to state the different dependencies on both /lib/wired_awaker.rb   and /bin/wired_awaker



To check if the current structure is working, chmod+x on  /bin/wired_awaker and from the terminal try ./bin/wired_awaker

*testing cli* should be returned. Ok so far so good...

Now let's create a "stories" class that will scrape the website and create story objects on the go. Sure I could separate my scraper from my story objects but for now let's start by doing it all in one class to see how they can interact together.

lib/wired_awaker/stories.rb 

And thus is the structure:

![](https://res.cloudinary.com/melzreal/image/upload/v1537966378/structure_y6eyif.jpg)


I'm going to fast forward here since its about day 12 of working on this and I have one key word to take from this:

overcomplication

OVERCOMPLICATION

Well, over complicating is what I did. It all went well with scraping the first page and doing the initial CLI with a case statement. That was working that very first day.

But boy oh boy did things get tricky after..

First I though, let's give the option to select "all tuesdays" or "all thursdays" and from there select the particular tuesday or thursday one wants. So my stories class looked like this:


*class WiredAwaker::Stories
  attr_accessor :title, :story_body, :url
  @@all = []

  def initialize
    @@all << self
  end

  def self.scrape_stories
    doco = Nokogiri::HTML(open("https://www.wired.co.uk/topic/wired-awake"))
    doco.css("li article").map{ |stories|
      s = self.new
      s.title = stories.css("a").text
    }
  end

  def self.scrape_weekday(weekday)
    doco = Nokogiri::HTML(open("https://www.wired.co.uk/topic/wired-awake"))
    selected_weekday = Date::DAYNAMES[weekday]
    @selected_stories = []
    @selected_urls = []

    doco.css("li article").map { |stories|
      s = self.new
      s.title = stories.css("a").text
      s.url = stories.css("a").attribute("href").value
    }

    self.all.map { |h|
      if h.title.include?(selected_weekday)
        @selected_stories << h.title unless @selected_stories.include?(h.title)
        @selected_urls << h.url unless @selected_urls.include?(h.url)
      end
    }

    @selected_stories.each.with_index(1) do |h, i|
      puts "#{i}. #{h}"
    end

  end
*



I essentially shoved the selected "thursdays" or "tuesdays" into an array of "selected stories" to then display it.

My CLI then could do:


*class WiredAwaker::CLI

  def fetcher
    show_main
    choose_weekday
    byebye
  end

  def show_main
    puts "Here are the headlines for the last 12 days of wired awake stories: \n "
    @stories = WiredAwaker::Stories.scrape_stories
    @stories.each.with_index(1) do |story, i|
      puts "#{i}. #{story}"
    end
  end

  def choose_weekday
    @awaker = WiredAwaker::Stories
    weekday = nil
    puts "Type 1-5 or the weekday you want top news for - Ex. type Tuesday or 2 \n"
  
    while weekday != "exit"
      #puts "\n Would you like to read more?"
      weekday = gets.chomp.downcase
      case weekday
      when "1","monday"
        puts "Select which Monday to read more about"
        puts "You can also type exit / Select different weekday: \n "
        @awaker.scrape_weekday(1)
      when "2","tuesday"
        puts "Select which Tuesday to read more about"
        puts "You can also type exit / Select different weekday: \n "
        @awaker.scrape_weekday(2) 
      when "3","wednesday"
        puts "Select which Wednesday to read more about"
        puts "You can also type exit / Select different weekday: \n "
        @awaker.scrape_weekday(3)
      when "4","thursday"
        puts "Select which Thursday to read more about"
        puts "You can also type exit / Select different weekday: \n "
        @awaker.scrape_weekday(4)
      when "5","friday"
        puts "Select which Friday to read more about"
        puts "You can also type exit / Select different weekday: \n "
        @awaker.scrape_weekday(5)
      when "exit"
        weekday = "exit"
      else
        puts " \n Sorry didn't catch that - Please enter a weekday by name / number or exit. \n"
      end
    end
  end

  def byebye
    puts "Good to see you!"
  end
end*

And show off what I wanted.. but yes its not really that Object oriented right if I'm grabbing object titles and shoving them into an array to display?

Giving it a big MEH I tried to work off that method, and I'm sorry to say it took me days to find the right selectors. Whenever I scraped the page I'd only get one big fat array instead of individual titles and stories. After much tinkering I found I could use the XPath to get the stories, but had to use a completely different selector for the titles.

That meant I couldn't just create stories with titles and bodies in one go, I had to separate the selectors then bring them all together with truncate. At this point in time my method to get into those stories looked like so:

* def self.scrape_url(selected)
    conv = selected.to_i-1
    url_completer = "https://www.wired.co.uk" + "#{@selected_urls[conv]}"
    doco_three = Nokogiri::HTML(open("#{url_completer}"))
    titles =[]
    stories = []

      doco_three.search("h2.bb-h2")[0..-3].collect{ |s|
        unless s.text.include?("Popular on WIRED")
         titles << s.text
        end
      }

      doco_three.search("/html/body/article/div/div/div/p")[2..-5].collect{ |s|
       stories << s.text
     }

      [titles,stories].transpose.each do |title, story|
        mm = self.new
        mm.title = title
        mm.story_body = story
        puts "#{mm.title} #{mm.story}"
      end

  
  end*
	
	And that seemed to work when calling it off the console. But I always had to call the method scrape_weekday  first to make sure the array which contained the url for the selected day worked.
	
	After all that suffering to find the right selectors I realised I couldn't call the methods chained from my cli class... I was.. OVERCOMPLICATING! Gosh, the requirements were to just go one level deep, why show all the tuesdays? So in exasperation, I've simplified and now stopped using my "scrape_weekday" method completely, and instead called on the urls that were stored in the story object, and gave the option of the user to select from 1-12. 
	
	Tadaa! Here is the final code for my story class (yes, it needs refactoring and dumping that second stories method away but still)
	
	
	*class WiredAwaker::Stories
  attr_accessor :title, :story_body, :url
  @@all = []

  def initialize
    @@all << self
  end

  def self.scrape_stories
    doco = Nokogiri::HTML(open("https://www.wired.co.uk/topic/wired-awake"))
    doco.css("li article").map{ |stories|
      s = self.new
      s.url = stories.css("a").attribute("href").value
      s.title = stories.css("a").text

    }
  end

  def self.scrape_weekday(weekday)
    doco = Nokogiri::HTML(open("https://www.wired.co.uk/topic/wired-awake"))

    doco.css("li article").map{ |stories|
      s = self.new
      s.title = stories.css("a").text
      s.url = stories.css("a").attribute("href").value
    }

    selected_weekday = Date::DAYNAMES[weekday]
    @selected_stories = []
    @selected_urls = []

  self.all.map { |h|
        if h.title.include?(selected_weekday)
        @selected_stories << h.title unless @selected_stories.include?(h.title)
        @selected_urls << h.url unless @selected_urls.include?(h.url)
      end
    }

    @selected_stories.each.with_index(1) do |h, i|
      puts "#{i}. #{h}"
    end

  end

  def self.scrape_url(selected)
    self.scrape_stories
    conv = selected.to_i-1
    urls_collection = self.all.collect{|s| s.url }
    chosen_day =  urls_collection[conv]

    @url_completer = "https://www.wired.co.uk" + "#{chosen_day}"
    doco_three = Nokogiri::HTML(open("#{@url_completer}"))
    titles = []
    stories = []

      doco_three.search("h2.bb-h2")[0..-3].collect{ |s|
        unless s.text.include?("Popular on WIRED")
         titles << s.text
        end
      }

      doco_three.search("/html/body/article/div/div/div/p")[2..-5].collect{ |s|
       stories << s.text
      }

      [titles,stories].transpose.each do |title, story|
        mm = self.new
        mm.title = title
        mm.story_body = story
        puts "#{mm.title} #{mm.story_body}"
      end

  end

  def self.all
    @@all
  end

end
*


And here is my, almost stupid, but yet functioning, CLI:


*class WiredAwaker::CLI

  def fetcher
    show_main
    choose_weekday
    byebye
  end

  def show_main
    puts "Here are the headlines for the last 12 days of wired awake stories: \n "
    @stories = WiredAwaker::Stories.scrape_stories
    @stories.each.with_index(1) do |story, i|
      puts "#{i}. #{story}"
    end
  end

  def choose_weekday

    weekday = nil
    @awaker = WiredAwaker::Stories
    while weekday != "exit"
      puts "Type 1-12 for the weekday you want to see top news for \n"
      weekday = gets.chomp
        case weekday
        when "1"
            @awaker.scrape_url(1)
        when "2"
            @awaker.scrape_url(2)
        when "3"
            @awaker.scrape_url(3)
        when "4"
            @awaker.scrape_url(4)
        when "5"
            @awaker.scrape_url(5)
        when "6"
            @awaker.scrape_url(6)
        when "7"
            @awaker.scrape_url(7)
        when "8"
            @awaker.scrape_url(8)
        when "9"
            @awaker.scrape_url(9)
        when "10"
            @awaker.scrape_url(10)
        when "11"
            @awaker.scrape_url(11)
        when "12"
            @awaker.scrape_url(12)
        when "exit"
            weekday = "exit"
        else
            puts " \n Sorry didn't catch that - Please enter a number from 1-12 or exit. \n"
        end
    end

  end

  def byebye
    puts "Good to see you!"
  end
end
*


So happy this is done. The wired awake website had some downtime earlier today and I had a small heart attack at the thought that I just wasted a week and something working on a scraper that could now be completely broken!

But oh well.. deep deep breaths *

At least its working !

See the github repo here:

https://github.com/melzreal/wired_awaker



Cheeriosss

Mel





