---
layout: post
title:  "Random Quote Signatures For OSX Mail"
date:   2014-03-27 12:19:00
header_image: vevo_mail_signature.png
---

I've been using OSX's Mail app for a few years now as my primary email application.  In the past, I've used it to track my personal Gmail account but these days I just use the web site for Gmail and use Mail to handle my work email and one or two other IMAP accounts that I have (the email I have for this site, my iCloud account, etc).  I've used a handful of other clients, but I'm not that big of an email junkie that I really care that deeply about setting up all sorts of personal tweaks and features for my email.  Mail handles just about everything I need and it comes with my computer - so I've stuck with it.

There is one feature that it lacks, though, and is the inspiration for this post. I don't even remember which mail app I had been using, but it allowed signatures to contain random quotes.  I know some folks will probably say that you shouldn't put random quotes in your email, but they can stop reading right about here and let the rest of us get on with our lives.  Mail allows you to add custom signatures, and even lets your randomize them, but you would have to manually input each quote as a seperate signature.  I have a file that I keep quotes that I like in and it is currently over 400 lines long.  Trying to maintain over 400 signature in Mail is not something I want to contemplate.

An alternative is to use some ready-made software.  For a long time I had been using Little Known Software's [SignatureProfile](http://littleknownsoftware.com/sigpro/), which allows for fancy signatures with tag replacements and a bunch of other nifty features.  If you're looking for the easiest possible way to add dynamic content to your signatures I highly recommend it.  For $12.00 you get a bunch of neat little utilities for your signatures.  While $12.00 isn't going to break my bank, SigPro has features I never used or needed and seemed like overkill.  I was sure I could figure out way to get dynamic signatures working without an external app - and I was right.  However, it wasn't as easy as I would've liked, so I feel like it's worth documenting.

I am going to detail how I went about solving this problem.  The particulars that I've used to solve it can be modified to match whatever you're most comfortable with.  I've toyed with the idea of updating it myself (mostly out of boredom or the need to tinker) but have kept this method for now since it works well enough.  My original solution involved trying to use AppleScript to update the signature so that I could create an app out of it using Automator, but I couldn't figure out how to get AppleScript to work well with HTML, so I gave up on that.  If you have a plain text email signature (or know how to get AppleScript to handle HTML) that's a viable alternative as well.

The first thing I did was set up a text file that contains all of my quotes.  Since I'm inputting them into an HTML signature, I ended up formatting them so that they could be directly inserted into the file without any editing.  Thus my quotes look something like this:

	"There lives more faith in honest doubt, believe me, than in half the creeds."<br>- Alfred Lord Tennyson

This setup is completely arbitrary and depends on the script you're using to insert the quotes/text.  An older version I had used pipes to separate the sections (I believe that's how SigPro asks you to format them).  This format has one quote per line, which makes it easier to find a quote.  Using a delimiter other than a line break means that you could include a paragraph of text (eg, poem or lyrics) without having to insert break tags into the quotes themselves.

Ok, now we have our quotes file.  But how do we go about getting it inserted into Mail, randomly?  Here's where it gets a bit tricky... The first thing you need to do is create the signature that you want to insert text into.  Make sure that you insert text into the signature exactly where you want the dynamic text/quote to go.  This will make sure that we have the signature saved in the format that you want it to eventually appear.

Next, you'll need to find the actual file where this signature is saved.  Look here:

	~/Library/Mobile Documents/com~apple~mail/Data/MailData/Signatures/

You can get to your Library folder either by using Terminal or in Finder by using Go -> Go To Folder... (ctrl-shift-G) and entering '~/Library' in the text box.  Once there you will see one or more files of the format "ubiquitous_{{HASH}}.mailsignature", where {{HASH}} is a series of numbers and letters separated by dashes. One of these is the signature we want.  Which one?  Well, your guess is as good as mine.  You can use finder to search for text that is unique to that signature (the quote you put on it) or just use a text editor to go through each one, looking for the right file. Once you've found and opened the right signature file we can begin to look though how to go about modifying it.  The method I'm giving here is a bit of a hack, but it's been working for me a while now so I have no reason to complain.

For the benefit of anyone who hasn't worked with this type of file before, here's a short description of what we're looking at. This file is a [MIME](http://en.wikipedia.org/wiki/MIME) encoded mail message. MIME lets us send binary or encoded data (files, images, etc) via plain text services such as email.  The top portion of the file (starting with "Message-Id") tells us what the file contains.  It also defines the "boundary" for the file.  You'll notice that there are multiple sections (starting with "--Apple-Mail=") that use the boundary.  This tells mail apps where each section of the file is and how to interpret it.  The first 'boundary' section is more than likely the body of your message in html.  Any images that you've attached will have their own section (base64 encoded).

You should be able to make out the body of your signature within the (obviously generated) html.  You'll notice that the lines all end with an equals sign ("="). The [MIME format](http://www.faqs.org/rfcs/rfc2045.html) requires that all of the lines be exactly 76 characters in length.  The equals are "soft breaks", meaning that the line continues onto the next line, ignoring the hard link break in the actual file.

So we're ready to modify this file to make it scriptable.  Find the quote that you inserted into your signature within the HTML gobbledygook.  It should be wrapped in a ridiculously long &lt;div&gt; that contains a bunch of style information.  Mine looks like this:

	<div =
	style=3D"color: rgb(0, 0, 0); font-family: Helvetica; font-size: 12px; =
	font-style: normal; font-variant: normal; font-weight: normal; =
	letter-spacing: normal; line-height: normal; orphans: auto; text-align: =
	start; text-indent: 0px; text-transform: none; white-space: normal; =
	widows: auto; word-spacing: 0px; -webkit-text-stroke-width: =
	0px;">

This is immediately followed by the quote text.  It's very difficult to figure out where the quote begins and ends using this generate html (which will change should we ever decide to tweak the format/layout of the signtaure) so we now need to insert our own delimiters to make it easy to replace the text.  To accomplish that I've used HTML comment blocks to wrap the quote text.  The advantage here is that the comment blocks are standard HTML and are not rendered to the end user, but they are easy to search for when replacing the text.  I've used <!--QUOTE-->{{quote is here}}<!--ENDQUOTE--> as my comment blocks around the quote, so my final signature looks something like this:

	<div =
	style=3D"color: rgb(0, 0, 0); font-family: Helvetica; font-size: 12px; =
	font-style: normal; font-variant: normal; font-weight: normal; =
	letter-spacing: normal; line-height: normal; orphans: auto; text-align: =
	start; text-indent: 0px; text-transform: none; white-space: normal; =
	widows: auto; word-spacing: 0px; -webkit-text-stroke-width: =
	0px;"><!--QUOTE-->=
	{{quote text}}
	<!--ENDQUOTE--></div>

I'm using {{quote text}} as a placeholder here - your should contain the actual text that you put in the signature.  Once that is saved you can go back into Mail and verify that your signature is working as expected.  Your quote text should still be showing properly, and you shouldn't see the comment tags we've just inserted - but they're there in the source of the signature.  Now we are finally ready to programatically change that text.  My primary development language is PHP, so I whipped up a solution using PHP and thus will show the script that I developed to do this, but it could just as easily be written in just about any other language or even a shell script.  Basically, what we need is a script to open the signature file, search for the comment tags that we put in and replace the text in between them with a random quote from our quotes file.

Here is how I accomplished this with PHP:

	$signature_file = "/Users/jamesalday/Library/Mobile Documents/com~apple~mail/Data/MailData/Signatures/ubiquitous_....mailsignature";
	$signature = file_get_contents($signature_file);

	$quotes = file("/Users/jamesalday/Documents/mail_sigs/sigs.txt");
	$key = array_rand($quotes);
	$myQuote = $quotes[$key];
	$new_signature = preg_replace('/\<!--QUOTE--\>=\n[^\<]*\s*(?:\<br>)?[^\<]*<!--ENDQUOTE-->/', "<!--QUOTE-->=\n" . $myQuote . '<!--ENDQUOTE-->', $signature);
	file_put_contents($signature_file, $new_signature);

Nothing too magical going on here.  We open the signature file and save it to a variable.  Then we open the signature file and save each line as an element in an array.  Then we choose a quote by choosing a random element from that array (this part might break down if the signature file were getting REALLY large, but it works fine for my couple hundred lines).  Then we use a regular expression to find and replace the quote text.  I'm being lazy here and just replacing the whole thing, including the comment tags instead of trying to find just the matched text.  Then we save the new replaced version back to the signature file.  I save this file alongside the signature file for convenience.  Once this is set up you should be able to run the script and see the quote text in your signature change each time.

So, we're done, right?  Well, not exactly.  It changes whenever we manually run the update script, but who wants to do that?  To be "random" we need it to change periodically.  In comes [launchd](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man8/launchd.8.html).  If you're familar with the Unix 'cron' command, launchd is Apple's replacement for that (and some other service-related apps).  We're going to use it to run our update script on a periodic basis - updating the quote in our mail signature once a minute.

Back to our Library - this time to the ~/Library/LaunchAgents directory.  It probably already exists, but you can create it if it doesn't.  In here we are going to create a [plist](https://developer.apple.com/library/mac/documentation/Darwin/Reference/Manpages/man5/plist.5.html) file that will run our script.  Plist files are basically just some XML that tells launchd what to do and where to find what it needs.  Mine is called 'com.jamesalday.Mail.random_sig_quote.plist'.  The name is fairly arbitrary and follows this format just to help identify it from other plists in the system.  Here is the contents of that file:

	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
	<plist version="1.0">
	<dict>
		<key>Label</key>
		<string>com.jamesalday.Mail.random_sig_quote</string>
		<key>Nice</key>
		<integer>20</integer>
		<key>ProgramArguments</key>
		<array>
			<string>/usr/bin/php</string>
			<string>/Users/jamesalday/Documents/mail_sigs/siggy.php</string>
		</array>
		<key>RunAtLoad</key>
		<true/>
		<key>StartInterval</key>
		<integer>120</integer>
	</dict>
	</plist>

You should be able to use this same file, just changing the name of the plist itself and the name/location of the update script.  Here's a breakdown of what this is doing... We set 'Nice' to 20 to tell the system that this isn't a very important call.  Other system commands that are high priority can take precedence over this one - so we don't cause a video to pause or a game to glitch just to update the email signature.  When the tell it to run the update script using PHP and tell it where to find the script.  RunAtLoad tells it to run the script as soon as we load this plist file. And finally we set StartInterval to 120 seconds so that we run the update once every two minutes.  I'm not writing all that much email and could probably bump this up to 5 or even 10 minutes without trouble, but two minutes seemed like a good compromise.

The final step to get this up and running is to register it with launchd and run it:

	launchctl load ~/Library/LaunchAgents/com.jamesalday.Mail.random_sig_quote.plist
	launchctl start com.jamesalday.Mail.random_sig_quote

If all went well, you should now see the quote in your signature change randomly every two minutes (or whatever interval you set in the plist).  If not, well, back to the drawing board.

This was a fairly "easy" fix to the problem, although it's a bit convoluted as well.  It'd be easier just to point Mail at a text file and tell it to load a random quote... but nothing's ever easy is it?  Anyway, it was fun figuring out how to get this working, and took a bit of trial and error to get it figured out, so I'm happy to share my results with the world.