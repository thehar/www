+++
tags = ["innerthoughts"]
categories = ["innerthoughts"]
date = "2017-04-15"
title = "Better, faster, stronger"

+++

HELLLOOOO

It all started with a random late night at Upwork preparing for a massive site "forklift" to AWS from our very outdated datacenter.  I knew vacation would be needed after said migration. So I booked and began my 15 day vacation to Iceland, Germany & Spain a few days after 48 hours of zero downtime.  On my flight to Germany, I decided to re-launch all of thehar.com into a static site and to `dogfood` it myself.

Lets get started.  First things first, DNS. Cloudflare offers a free service tier and will terminate SSL for you.  Sure there's `LetsEncrypt`, but this is more fun and you can make your site more robust in the future.  Lets make Cloudflare authoritative for the entire domain and proxy off www/non-www to S3.  I'd like to not worry about `uploading` or `removing` content and re-generating all the static content all the time, so lets use AWS Lambda to do our dirty work.  Lamba will do all of the things you tell it to in AWS without server(s) or will orchestrate EC2 compute nodes for you. The biggest thing here is that you pay for the _time_ spent reacting or processing.  You don't have to turn up completely new EC2 nodes or use Chef (like I normally would).

In order to get this working, we'll need to have an `input` bucket in S3 that you can put your non-generated assets into.  I suggest `input.domainname.com` just for ease of remembering what the hell these things are when you open S3's console.  Next you'll need to create two buckets in S3 to serve up `www.domainname.com` and `domainname.com`.  I prefer to keep things on the root domain and not serve up www traffic to everyone, although you can re-direct however you'd like.  One bucket needs to forward to the other in some shape or form S3's silliness.

## Testing Hugo locally:

I make the assumption that all of your local development is done on MacOS, 10.12.3.

### Dependencies
```
[Homebrew]
*  brew install awscli npm PyYAML make
*  brew cask install hugo

[Cloudflare]
* ROOT DNS
* www.ROOT DNS

[AWS]
* IAM role for Lamba functions
* Lamba function to add/remove
* S3 buckets
```

When you're ready to test out your changes and run a local web server, feel free to open Terminal.app and run:

`hugo server -wvD --port 1313`.

```
Options tagged off demonizing hugo above:
-w means to watch the directory and recreate as needed.
-v means to verbosely state what is going on in the web server.
-D means to include drafted posts you may have created.
-1313 means to serve everything on port 1313.
```

### Troubleshooting hugo sites
1. Purge the public/ directory.
2. Run the built in web server in watch mode.
3. Open your site in a browser.
4. Update the theme/pages.
5. Glance at your browser window to see changes.
6. Return to step 4.

### Contributing
1. Create a local branch to push to Github.
2. Create a new issue on https://www.github.com/thehar/www/issues/new
3. Push your branch to Github and cre
4. Create a pull request https://www.github.com/thehar/www/pr/new
