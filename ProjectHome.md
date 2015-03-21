# Zeta Twitter #

## Introduction ##

In this article I will introduce you a small console application written in C# and .NET 3.5 that enables you to automatically follow Twitter users based on keywords they use in their tweets.

If you don't know Twitter, you may look at their website [Twitter.com](http://twitter.com) or read the [Wikipedia](http://en.wikipedia.org/wiki/Twitter) article about Twitter.

## Background ##

The motivation for developing this tool was to enable some of our company's Twitter accounts to automatically follow other Twitter users based on the keywords they post.

E.g. we maintain a [Twitter account](http://twitter.com/zetaproducer_en) for our CMS "Zeta Producer Desktop". Now I wanted to do [a search for "cms" on Twitter](http://search.twitter.com/search?q=cms) and follow all users (or the first n ones) that were displayed in the search result.

Even though a similar service exists with [Twollow](http://twollow.com), they charge for their services. Since I wanted a free version (although I do have way less features than they have implemented), I started developing Zeta Twitter.

## Technical implementation ##

The application is a .NET 3.5 console application that is intended to be used as a scheduled task in the Windows Task Scheduler. You would e.g. configure the task to run every 30 minutes.

![http://i3.codeplex.com/Project/Download/FileDownload.aspx?ProjectName=zetatwitter&DownloadId=73231&is=image.png](http://i3.codeplex.com/Project/Download/FileDownload.aspx?ProjectName=zetatwitter&DownloadId=73231&is=image.png)

I am using the free library [tweet#](http://tweetsharp.codeplex.com/) to communicate with the [Twitter API](http://apiwiki.twitter.com/).

You can add multiple Twitter accounts and for each account multiple keywords to check. The whole configuration is done within a separate configuration file called "_ZetaTwitterConfiguration.xml_". This file must be stored in the same folder as the executable "_ZetaTwitter.exe_".

Currently all attributes are mandatory. Adjust the file in the downloaded archive with the XML/text editor of your choice.

Example configuration file:

```
<?xml version="1.0" encoding="utf-8" ?>

<zetaTwitter version="1.0">
	<accounts>
		<account userName="mytwittername" password="mypassword" active="true">
			<autoFollowKeywords>
				<keyword search="webcam" />
				<keyword search="magerquark" />
				<keyword search="#mj" />
			</autoFollowKeywords>
		</account>

		<account userName="sometest" password="somepassword" active="false">
			<autoFollowKeywords>
				<keyword search="cms" />
				<keyword search="content management system" />
			</autoFollowKeywords>
		</account>
	</accounts>
</zetaTwitter>
```

## Internal function ##

This chapter outlines the highlights of the code. Again the warning that none of the code is rocket science.

### Configuration ###

The configuration is being stored in an XML file (as stated above) and being read into two classes by simple `XmlDocument` method calls. I do read it manually, not with any XML to class mapping.

The classes are:

  * `Configuration` - The root class which has an array of `Account` objects each for each `<account>` node in the configuration file.
  * `Account` - The representation of one account. Has an array of `string` objects for the keywords specified in the configuration file.

### Processing ###

The processing is done the `Program` class. Using the `IsAlreadyRunning` property (which is implemented with a mutex) to quit itself if it is already running. So it is a single-instance console application.

Reason for that is that depending on the configuration file size, one run may take rather long and I wanted to avoid that the Windows Task Scheduler fire multiple instances. Twitter only allows a limited number of requests per IP address per day, so you better play nice here.

The Main method consists of two nested loops, the outer iterates through all `Account` objects of the configuration, the inner through all keywords of the currently processed account.

Using tweet# is interesting: They heavily rely on .NET 3.5 [extension methods](http://en.wikipedia.org/wiki/Extension_method) to implement most of the functionality. The [basic steps are outlined in the online documentation](http://tweetsharp.codeplex.com/s) of tweet#:

  1. Create a request of what you want to get and specify your account login credentials.
  1. Retrieve the reply from Twitter and further process the reply.

**Example for step 1:**

```
var twitter = 
    FluentTwitter.CreateRequest()
        .AuthenticateAs( account.UserName, account.Password )
        .Search()
        .Query()
        .Containing( keyword )
        .AsJson();
        
response = twitter.Request();
```

Here in this example, I authenticate an account with user name and password, and then query for tweets with the given keyword. The result is then returned a string containing the [JSON](http://en.wikipedia.org/wiki/JSON) formatted result.

**Example for step 2:**

```
var searchResult = response.AsSearchResult();
```

The response is transformed into a search result collection. Here you have the ability to e.g. iterate through the results and further process them:

```
foreach ( var status in searchResult.Statuses )
{
    var userName = status.FromUserScreenName;

    // ... do something with the user name ... 

}
```

Depending on the kind of query, different object types are being returned. Again, please see the [tweet# reference Wiki ](http://tweetsharp.codeplex.com/) for full options.

### Notes ###

Currently all output is written to console window only. Ususally next steps would be to use a logging framework like [LOG4NET](http://logging.apache.org/log4net/index.html) instead to log to various locations like e.g. e-mail or log files to get notified when something goes wrong.

## Current state of the tool ##

The tool was a quick development of approximately 2 hours. It currently does exactly what I wanted it to do. Proably It does not even have any of the features that you want it to have.

Here you have at least two options:

  1. Download the source code and enhance it the way you want it.
  1. Tell me (preferable in the [Issues section](http://code.google.com/p/zetatwitter/issues/list) which features are missing that I should add.

Of course option 2. would be helpful to me when enhancing.

As usual, I will enhance, extend and correct the tool over the next weeks and months. Keep the feedback coming!

## History ##

  * **2009-11-06** - Updated to latest tweet# library.
  * **2009-06-26** - First version.