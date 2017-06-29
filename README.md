# google calendar hours widget
Let's start by saying this: I'm not sure if you'd really call this a widget, but my library team understands that term, so that's what I'm calling it. I suppose it could be a plug-in? 

Also: I wouldn't normally put both HMTL and JS in the same HTML file, but that's the easiest way to add it to a libguide, so if you're not a library, sorry about that.

## what does it do?
This widget takes information from a single google calendar and spits it out into a table. I'm positive it can be tinkered with to do other things, but I use it to display "Today's Hours" on my library's website. In theory, it can display lots of locations as long as they're all on the same calendar. In practice, it displays three (Downtown Indy, Lawrence, and Greencastle). 

The hitch in this plan is that the events in google calendar **must** be input a specific way. If a single part of it is not done correctly, the information will not display or will only be partially displayed.

## what doesn't it do?
It does not scrape information off of any old calendar and spit it out how you want it. It uses the google calendar API and authentication keys and will only work on calendars you actually have access to.

## what do I need to do this myself?
The script is dependent on a few things:

- google calendar editing permissions
- google developer console
- lodash.js
- moments.js
- bootstrap

The first four are essential. Maybe. The lodash bit might be unnecessary if I get time to rewrite it using ES2015, but this was faster for now. Bootstrap just happens to be the framework that our CMS uses, so I use bootstrap components in the table. Feel free to remove them.

Lodash and moments are linked to using the CDN, but feel free to download and use locally if you need a faster load time. 

## main steps *or* table of contents
1.  [google calendar for address](#calendar)
2.  [google developers console API manager for API key](#api)
3.  [doctor the code](#code) ⇒ paste in your calendar address and API key in those variables/`const`
4.  [update calendar events with description and start/end times](#events)
5.  drop code into libguide box HTML editor

## how does this work?
If you're a libguides user, it's as simple as creating a new box and adding this (DOCTORED) code to the rich text/HTML editor. The doctoring part is where it could get complicated. I hope to explain everything as best as I can in the following documentation. 

*Keep in mind that I'm not a javascript ninja. I just tinker until what I have in mind matches what's showing up on the screen. This may not work the way you want it to. I'll tinker with it some more if you can't get it to work, but I most certainly don't have the time to fix it for you. Put on your thinking cap and go google, my friend!*

## documentation (a.k.a. doctoring the important bits)

### google calendar <a name="calendar"></a>
Get yourself to your google calendar's settings page. Hopefully you have something that looks like this:
![google calendar address](https://libapps.s3.amazonaws.com/accounts/41961/images/google_cal_address.jpg)

That bit after that ends in @group.calendar.google.com is your Calendar ID. You will need this for your request URL, so hold on to it. 

### google developer console <a name="api"></a>
Head to the [Google Developers Console API Manager](https://console.developers.google.com/). You'll probably need to sign in if you're not already signed in to Google via your browser.

On the left side of the screen, click on the key icon with **Credentials** next to it. 

You're going to need a new credential, so click the blue **Create credentials** button and select API key. You will be given a very long string of random characters and numbers. This is the API key you will need to access google's API. It will also be added it your request URL, so copy that down somewhere, too.

This key belongs to your username now, so any time you come back to the Developer Console API Manager, it will be there. Until you delete it. I've created different keys for different Google API projects I've worked on, so if you decide to try this code using different calendars, you should get a new API key as well.

### custom code <a name="code"></a>
Copy the code from the [Hours Widget github repo](https://github.com/carylwyatt/google-cal-hours-widget/hours.html). Or clone it. However you work, you do you!

#### HTML
Under the "Very Few Lines of HTML" heading, you can see how I've chosen to display this information on our website. Basically, there's a third-level heading with "Today's Hours" followed by an empty paragraph tag for today's date. Neither of these are necessary, but the empty paragraph will populate with the current date like this: **Thursday, June 29, 2017**.

Following that is the skeleton of the table where the hours will go. When I first created this code, I used an unordered list and the hours info was spit out in list items instead of table rows. It's easily changed, you just have to know a little HTML to pull it off. If you decide to change it, just make sure to copy over the unique element IDs because that's how the javascript knows where to inject to calendar info.

The link under the table is not important, but it links to our page that displays all the hours.

#### JavaScript
This is where the magic happens!

You'll see variables set for the calendar address (`const calAddress`) and the API key (`const keyAPI`). Paste your calendar and key into their respective variables and ensure there are quotes around them (strings require quotes!).

Then next variable is `const googleCal` which takes the calendar address, the API key, and today's date (see below if you're not in the Eastern time zone) and makes a lovely and quite long URL. You can copy this URL and paste it into your browser, replacing all the variables (i.e. `${keyAPI}`) with their string literals and today's date (in `YYYY-MM-DD` format) to double check if your URL is pulling the calendar info like you think it should. 

Here's what part of the JSON from my calendar looks like:
![google calendar JSON](https://libapps.s3.amazonaws.com/accounts/41961/images/google_cal_JSON.jpg)

I have a [JSON view enabler extension](https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc?hl=en) installed in my browser, which makes it easier to read, but there should still be this basic type of information. If you can recognize at least the date and summary or description of an item, you can verify that it's pulling the info it should be. 

### google calendar events <a name="events"></a>
If your calendar hasn't shown up yet, it's probably because your events haven't been set up correctly. This still happens to my calendar regularly because the people putting events in don't know what needs to be filled in and I didn't catch it in time.

The way the javascript works, it literally tells the browser to cycle through all the individual events that showed up in that JSON file earlier. Each event is its own object and each object has several properties. The javascript loop basically does this:

#### For each event:

- if it has start and end time properties (`(library.start.dateTime && library.end.dateTime) === true`), spit out the event description in one cell of the row and the start and end times in another cell of the same row and move on to the next event in the list;
- OR if the event doesn't have a start and end time, spit out the event description in one cell of the row and the second cell of the row should say "Closed" and move on to the next event in the list;
	- (my reasoning for this, which may be faulty, is that if it doesn't have an open/close time, it's closed all day-- at least, our closed libraries have the "all-day event" box checked, meaning no start/end times)
- OR if the event doesn't have a description, skip it all together and move on to the next event in the list

...until all the events on the list have been checked and it stops.

Using Friday, June 30, 2017 of my library's calendar as an example, here's what will happen: 

Downtown    8 am - 9 pm 
Lawrence      9 am - 5 pm 
Greencastle  Closed

Downtown is actually the description of the event. If you look at the JSON image above, you'll see that the event summary says **GRC OPEN** but the description says **Greencastle**. The JavaScript displays the description, because that's what I told it to do. GRC OPEN is easier for us to read on the Google Calendar, but it doesn't make sense to any library users, so I decided to add the campus name to the description and display that instead. Feel free to change that in the javascript if you find it unnecessarily complicated (lines 103, 107, 114).

If, for whatever reason, you have an event without a description, the script will skip it (as seen in the third bullet point of the list of if statements above). This is because people at my library like to know when breaks are coming up, but I don't want them to show up on the public-facing calendar. No one on the library website cares that it's the first day of classes, they just want to know what time the library closes today. You can delete that part of the if statement if you don't want that to happen (lines 105-109).

#### TL;DR:

- event summary can be whatever will make it easy for you to read
- event description is what will actually show up on the calendar
	- if the description is left blank, the event won't show up at all
- if you want the open and close times to show up, make sure you enter them into the event
- if you are closed, don't enter a start/end time
	- instead check the "all-day event" box for the event
	- this will cause the script to list "closed" on the hours widget
