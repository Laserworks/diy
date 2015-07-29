---
layout: post
title:  "Share Messages via Kindle on Fridge"
date:   2015-07-28 20:00:0
tags:
- kindle 
- display
- messaging
- raspberrypi
- fridge
---

Amazon Kindle is well suited to be _upcycled_ as a fridge messaging display due to its low power consumption. 

![Share Messages via Kindle on Fridge]({{site.baseurl}}/images/2015-07-29-share-messages-with-kindle-on-fridge/01.jpg "Share Messages via Kindle on Fridge")

The main idea is explained here and below are all the technical details:

![Share Messages via Kindle on Fridge]({{site.baseurl}}/images/2015-07-29-share-messages-with-kindle-on-fridge/kindle-emails.svg "Share Messages via Kindle on Fridge")

### Step 1: Send Email to myfridge@gmail.com

First you need to create a new email account for your fridge kindle on gmail.com, e.g., `myfridge@gmail.com`. Whoever sends an email to `myfridge@gmail.com` will get his email displayed on your fridge.

### Step 2: Email Arrives into myfridge@gmail.com Inbox

There is not much to explain here, all the hard work is done by the google team :)

### Step 3a: Webserver on Raspberry Pi is Checking New Emails

You can use any server you have, many people have a Raspberry Pi running on their home network. 
Our webserver app is written in Ruby.
First we need to gather last email from `myfridge@gmail.com` inbox.

We use [ruby-gmail](https://github.com/dcparker/ruby-gmail) gem which is designed to interact with gmail inbox via IMAP:

{% highlight ruby %}
require 'gmail'
{% endhighlight %}

we establish connection with our gmail account:

{% highlight ruby %}
# note that only 'myfridge' is provided instead of 'myfridge@gmail.com'
gmail = Gmail.new 'myfridge', 'some_password'
{% endhighlight %}

And gather the last email:

{% highlight ruby %}
last_email = gmail.inbox.emails.last
{% endhighlight %}


### Step 3b: Webserver on Raspberry Pi Renders Last Email as Webpage

Source code below is in this [repository](https://github.com/petervojtek/email-to-kindle-on-fridge).

We will use [sinatra ruby gem](http://www.sinatrarb.com/) to help our ruby script to act as webserver.
We want to achieve that when we visit in our web browser `http://rpi_ip_address:1212/email` we will see the last email.


{% highlight ruby %}
require 'gmail'
require 'sinatra'
require 'base64'

# we don't want the actual login and password to be stored in this source code file
gmail_login = ARGV[0]
gmail_password = ARGV[1]
$gmail = Gmail.new gmail_login, gmail_password

# we want to remember uid of the last email to know if newer email has arrived
$last_email_uid = nil

def fetch_last_email
  email = $gmail.inbox.emails.last
  if email.uid == $last_email_uid
    return # we already see the latest email
  end
   
  $last_email_uid = email.uid
  $sender = Mail::Encodings.value_decode email.sender.first.name
  $subject = Mail::Encodings.value_decode email.subject
  $received_at = email.envelope.date
  $image = nil
  if email.attachments.size > 0 
    $image = email.attachments.first.decoded
  end
end

set :port, 1212 # run the webserver on port 1212

get '/email' do # this html is rendered in your browser when you visit http://rpi_ip_address:1212/email
  fetch_last_email
  
  <<STRING
<html>
  <body>
    <!-- display who has sent the email and when -->
    <p><b>#{$sender}</b>, #{$received_at}</p> 

    <!-- email subject -->
    <h1>#{$subject}</h1> 

    <!-- first image attachment from email -->
    <img src="data:image/jpg;base64,#{Base64.encode64($image) if $image}" style="width:100%;"> 

    <!-- javascript will periodically check whether newer email has arrived -->
    <script> 
        lastEmailChangedCheck = function(){
          ajaxGetRequest('should_we_reload?rendered_email_uid=#{$last_email_uid}', function(xhr) {	
            if(xhr.responseText == 'yes')
                location.reload();
          });
          setTimeout(lastEmailChangedCheck, 10000) <!-- every 10 seconds -->
        }

        lastEmailChangedCheck();

    </script>
  </body>
</html>
STRING
end

# javascript in your browser is making ajax request to this sinatra action
get '/should_we_reload' do
  fetch_last_email
  (params['rendered_email_uid'].to_i == $last_email_uid) ? 'no' : 'yes'
end
{% endhighlight %}

Now we launch the webserver on our Raspberry Pi. the source code below is stored in file `email-to-kindle-webserver.rb`. For debugging purposes we can run it as following:

{% highlight ruby %}
$ ruby email-to-kindle-webserver.rb myfridge password
{% endhighlight %}

Once all tested, you rather want to start the webserver automatically (in case your RPi is rebooted). You can use [supervisord](http://supervisord.org/) to automatically start the webserver by putting following content into `/etc/supervisor/conf.d/email-to-kindle-webserver.conf`:


{% highlight text %}
[program:email-to-kindle-webserver]
command=ruby email-to-kindle.rb myfridge password
autostart=true
startsecs=1
startretries=1
user=your_rpi_login_name
directory=/path/to/email-to-kindle/repo
{% endhighlight %}

### Step 4: Display Last Email on Kindle

First we to connect kindle via wifi to the same network where Raspberry Pi is running.

Next we [disable](http://www.amazon.com/forum/kindle/ref=cm_cd_et_md_pl?_encoding=UTF8&cdForum=Fx1D7SY3BVSESG&cdMsgID=Mx2HLTBL11A2UBQ&cdMsgNo=11&cdPage=1&cdSort=oldest&cdThread=Tx3AL0H1N6IDN3S#Mx2HLTBL11A2UBQ) kindle's screensaver as kindle will automatically enter sleep mode after 10 minutes:

* Press the Home button to go the the Kindle home screen
* Press the keyboard button to display the virtual keyboard
* Type following string: `;debugOn` and press Enter button (not the Done button)
* Type following string: `~disableScreensaver` and press Enter.


Now we launch kindle's web browser by pressing Menu > Experimental > Web Browser.

You have to enter the url address `http://rpi_ip_address:1212/email`. I advise you to save the address immidiately as a bookmark to avoid entering the address in future which is painful on the kindle's virtual keyboard.

That's all. The webpage will detect new email arrival via ajax and javascript and will reload itself.


### Additional Notes

* I put this magnetic [Self Adhesive Magnetic Strip](http://www.ebay.com/itm/131485684970?_trksid=p2060353.m2749.l2649&ssPageName=STRK%3AMEBIDX%3AIT) on backside of kindle to keep it attached to the fridge.
* You may be tempted to use native [Gmail API](https://developers.google.com/gmail/api/) to check emails instead of using IMAP, but the official Gmail API is super overengineered and overcomplicated and you will probably spend few hours trying to achieve the same you can do in five minutes with IMAP.
* I received a suggestion to allow browsing the arrived emails instead of displaying newest email only. Unfortunately, the large next page/previous page buttons on Kindle 4 emit no signal to web browser and thus cannot be used to easily switch between email messages. I guess the touchscreen kindle's are better suited for this purpose.
* It seems is is not possible to enter fullscreen on the kindle's web browser.

### TODOs

* add more robustness to the gmail IMAP session so that when it is disconnected after few days of running, it will reconnect automatically.

