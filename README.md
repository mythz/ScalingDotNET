# Making .NET a Star Performer

This page is in response to Robert Scoble's declaration that the [death spiral of MySpace can be attributed to their bet on the Microsoft technology stack](http://scobleizer.com/2011/03/24/myspaces-death-spiral-due-to-bets-on-los-angeles-and-microsoft/), specifically .NET. As a developer with an interest in developing high-performance scalable systems using C# I've learned a few things in my research on what makes systems perform and scale well which is what I'd like to share to the wider .NET world. In order to keep this document palatable and provide the most value I will try to stick to point form and link to various topics and articles with the intention for this to serve as a jump page into your own research. 

## Disclaimer
The opinions expressed here are my own formed through my own experience and research in this area, although I believe them to be true at the time of this writing treat them as 1 developers view point and as such I encourage you to do your own research. Where it provides an efficient option I'll be linking to my own OSS solutions developed in response to fill a missing piece for my high-perf solutions - for transparency these recommendations will be clearly marked with *Me, My, I* or something similar. This is not a definitive list by any means and strictly sticks to the small subset of **stuff I know**. As I want to maintain this as a  living and relevant document I invite any contributions (or corrections) to this page and GitHub site that other developers have found useful in their endeavours - please leave a pull-request, issue or wiki page to this effect. 

As a final cautionary note: performance and scaling is very much a series of trade-offs, where what may be a quick win providing instant value to some might be a pre-mature optimization that's not worth the effort for others. Keep this in mind and make your own judgement pertaining to your use-case.

## Why should we even care about performance?

My first interest in performance came to me nearly 8 years ago during my tenure as a Java GIS developer. Before Google maps the world of GIS was very different where the common workflow of a GIS user was to:

  1. Stop, customize a map query with selected layers/extents of interest 
  2. Submit your query and wait for your request to be rendered and a response returned from the server
  3. View your results before re-fining your query and starting the process again

Needless to say that the usability and utility of said software meant that no-one outside of work hours who weren't being paid to, was using GIS software. Then came Google with their pre-rendered and cached images of the world in their Google Earth (formerly keyhole) and Google Maps software and changed the game forever. The instant utility and performance of the software meant that it appealed to a much wider audience which turned GIS from a niche product to one that's apart of our daily lives. 

There are various research studies as to how the slightest change to performance / load times can have a dramatic effect on users perception and continued use of your software. One thing I will say is that a common trait shared by all top internet properties is that they load fast and provide instant utility. With that said I will close this section off with my favourite quotes on the topic:

[![@brada on speed](http://mono.servicestack.net/img/brada-speed.png)](http://twitter.com/#!/brada/status/42339108521115648)

-- [Brad Adams](http://twitter.com/#!/brada/status/42339108521115648) (Current Google employee, former Microsoft project manager and co-author of .NET Framework Design Guidelines)

[![@brada on speed](http://mono.servicestack.net/img/fredwilson-speed.png)](http://blog.mashape.com/post/44118215209/10-golden-principles-of-successful-web-applic)

-- [Fred Wilson’s 10 Golden Principles of Successful Web Apps](http://blog.mashape.com/post/44118215209/10-golden-principles-of-successful-web-applic)


## What to look for at a high-level

Some general guiding principals I use when I'm coding along include:

	Develop for the end in mind	
	
I also like to coin this the **Google approach** where in all their strategic properties (i.e. Search, Gmail, Maps, Chrome, etc), they research deeply into what ultimate result will deliver the best customer UX and work towards this end. It's sort of like YAGNI-EU i.e. YAGNI for the end user. On this note, if you need any more advice that Single Page Applications (SPA's) and not Silverlight are the way to go, the fact that Google has decided to pursue it should at least warrant investigation.
	
	Avoid premature optimizations where un-necessary and focus on Macro-level optimizations

Many developers generally preach that you should never prematurely optimize, however I'm more of the mind that since you're already writing code, think about how often this piece of code gets run and optimize accordingly. In practice this means I never optimize one-off code yet spend considerable effort speeding up [my serializers](http://www.servicestack.net/mythz_blog/?p=344) where any improvement has the potential to improve all my services. You don't know when your head is going be back in the space you're currently in so a decent effort at efficient code (without hacks) is not a bad idea.

With that said, if you're developing an ajax app, something like [Page Speed](http://code.google.com/speed/page-speed/) that tests the complete end-user experience is invaluable to visualize where the areas of optimization will yield the most value. 

By Macro optimizations I mean, making use of a CDN, taking advantage of ETags to save bandwidth or even better using the Expires and Cache-Control HTTP Headers to eliminate un-necessary requests entirely.

### Have benchmarks in-place before optimizing code
Although this should go without saying, it can get lost on eager devs looking for a quick win. It's effectively impossible to assume every optimization you make will end up improving total performance. The only way to know for sure if you are optimizing the right way is to have automated benchmarks available to test on every iteration.

### Numbers everyone should know
This **[list is so important](http://everythingisdata.wordpress.com/2009/10/17/numbers-everyone-should-know/)** that they should be in the forefront of developers minds (if it matters in what they do of course) **before** they develop software: 

Operation                           | Time (nsec)
---------------------------------   | -----------
L1 cache reference                  | 0.5
Branch mispredict                   | 5
L2 cache reference                  | 7
Mutex lock/unlock                   | 25
Main memory reference               | 100
Compress 1KB bytes with Zippy       | 3,000
Send 2K bytes over 1 Gbps network   | 20,000
Read 1MB sequentially from memory   | 250,000
Roundtrip within same datacenter    | 500,000
Disk seek                           | 10,000,000
Read 1MB sequentially from disk     | 20,000,000
Send packet CA -> Netherlands -> CA | 150,000,000

### Caching
The other most important facet and one that runs at the heart of all high-performance systems is Caching, where even the most inefficient systems can be masqueraded by good caching strategies. The level of caching which provides the most value is what I like to call **front-line caching** where you cache the outer most layer (saving the most CPU) in the most optimal output format. E.g. if you're developing web services, you wan't to cache your gzipped/deflated JSON or XML output. The most optimal place to store your cache is in-memory although if you have load-balances servers (as many popular systems do) you will want to consider the leading caching servers in this area capable of some [impressive numbers](http://antirez.com/post/update-on-memcached-redis-benchmark.html):

  * [Memcached](http://memcached.org) - The original and industry standard
  * [Redis](http://redis.io) - The hot new entry into this space, like a Memcached on steroids used by a [growing number of companies](http://redis.io/topics/whos-using-redis) including .NET's own [StackOverflow](http://highscalability.com/blog/2011/3/3/stack-overflow-architecture-update-now-at-95-million-page-vi.html)
  * [AppFabric](http://www.microsoft.com/windowsazure/AppFabric/Overview/default.aspx) - Worth a mention, since its Microsoft's entry into this area, but they're earlier recommendations for SQL as a distributed cache and their weak fine-grained caching options inherent in ASP.NET caching provider leaves me un-impressed. 
  
Since caching should be treated and thought about as an important first-class concept, I like to keep in mind the cacheability and use-cases of my services and pages when designing the level of granuality of my API. Since I like fine-grained control of caching, I prefer to use [my own pluggable caching apis with fine-grained cache control](https://github.com/ServiceStack/ServiceStack/wiki/Caching). 

Most .NET developers will likely just make do with **Time based caching** as that's the default behaviour in ASP.NET caching provider API's and OutputCaching directives. My preferred option is to cache on **Validity** where I would invalidate caches manually (e.g. when a user modified his profile, clear his cache) which means you always get the latest version and you never need to hit the database again to rehydrate the cache if it hasn't changed.

### Compression
Related to the subject of Caching is Compression since they usually operate on the **outer most layer** i.e. the final Output which in a lot of cases you should consider compressing if its not already (i.e. .jpg, .png, etc). The .NET framework comes with its own GzipStream and DeflateStream classes to do this for you. (Note: prior to 3.5 .NET had a weak impl of DeflateStream so I used [DotNetZip](http://dotnetzip.codeplex.com/) instead).

#### Enabling Web Server Caching and Compression
And speaking of the outermost tier it's hard to go any further than the web server itself, so here's a useful article on [configuring caching & compression in IIS](http://weblogs.asp.net/anilkasalanati/archive/2010/12/29/enabling-http-caching-and-compression-in-iis-7-for-asp-net-websites.aspx) Whilst html5boilerplate delivers equal love to [Web.Config](http://html5boilerplate.com/docs/#web.config), [nginx.conf](http://html5boilerplate.com/docs/#nginx.conf) and [Apache](http://html5boilerplate.com/docs/#.htaccess).

## Inefficiencies inherent in software design

There are a few major things that can really degrade performance in software design. Each topic deserves volumes but I will keep it short and invite you to do your own reading.

### 1. Blocking IO

Blocking IO is the most expensive thing you can do in general software development. When you make an IO call and the call doesn't complete until the response is returned like most .NET IO APIs (i.e. the API doesn't provide a callback or event handler) that's a blocking call and as a result the calling thread is blocked and can't be reused for other tasks while its waiting for the IO response to return. On a single-threaded server this blocks the server from processing all other requests which is why we have ThreadPool web servers like IIS and Apache so your web server can keep serving pages while one of your requests is waiting to hear back from your SQL Server cache :)

The problem with this design is that ThreadPool servers consume more CPU and memory resources then their non-blocking cousins. They do however allow the programmer to write more intuitive sequential code without the use of callbacks/event handlers which is why they're still the default in general purpose web development. Since it simplifies development, my advice is to stick to IIS for general web development, as a good caching strategy will provide your instantaneous response times. 

However there are times when it does make sense to use a non-blocking server:

  * You're approaching the [C10K Problem](http://www.kegel.com/c10k.html)
  * You have to handle multiple concurrent connections (i.e. support Web Sockets/Comet chat servers)
  * You have identified a hotspot that could benefit from a non-blocking solution  

In general async programming **is hard** but there are frameworks that simplify the burden as much as possible. The [AsyncEnumerator](http://msdn.microsoft.com/en-us/magazine/cc721613.aspx) class allows you to perform asynchronous operations using a synchronous programming model by leveraging C#’s iterator language feature. The class is part of the Jeffrey Richter's [Power Threading](http://wintellect.com/PowerThreading.aspx) library and is completely free to use. The [Rx framework](http://msdn.microsoft.com/en-us/devlabs/ee794896) was the first to tackle the problem by allowing you to compose async operations in a reverse composable and chainable LINQ pattern. Although the most anticipated new feature to help programming in this domain is the arrival of [C# 5.0's async/await](http://msdn.microsoft.com/en-us/vstudio/async.aspx) features which promises to simplify it even further by allowing you to write composable async logic sequentially thanks to some compiler magic.

Whether or not this technology produces some non-blocking servers from Microsoft is not clear, however there are already efforts that already provide async, non-blocking frameworks, the 2 most notable ones for .NET are:

  * [Manos de Mono](http://jacksonh.tumblr.com/post/1159500924/manos-de-mono-the-manifesto)
  * [Kayak HTTP Server](http://kayakhttp.com/)

#### The node.js revolution
For devs who are unaware of the [node.js](http://nodejs.org) revolution happening right now on Unix platforms, it primarily has to do with being able to build high-performance, non-blocking network servers trivially with simple JavaScript. Which once coupled with one of the fastest JavaScript runtimes in V8 (thanks largely to the arms race amongst the major browser vendors to produce the fastest js implementation) provides developers un-precedented power to create runtimes previously prohibitively impossible to develop - but for an elite few C/C++ wizards.

For those interested for a glimpse into this new world, here are some enlightening videos I recommend watching:
 
  * [Ryan Dahl: Introduction to NodeJS](http://www.yuiblog.com/blog/2010/05/20/video-dahl/)
  * [Douglas Crockford: on JavaScript - Loopage](http://developer.yahoo.com/yui/theater/video.php?v=crockford-loopage)

Now the best time to make use of a non-blocking server is when you can use it for free and by that I mean when you don't have to develop against it and be forced into a world of asynchrony. But if you need only to make use of its services, you won't find anything better than this distinguished list below to serve your needs:

**Distributed in-memory caching**

  * [Redis](http://redis.io)
  * [Memcached](http://memcached.org)
  
**HTTP Server, Reverse Proxy, Load Balancer, etc**  

  * [Nginx](http://wiki.nginx.org)

### 2. IO Latency

On a similar topic to Blocking IO, is IO latency where the only thing slower than a synchronous IO call - is multiple synchronous IO calls so best to avoid them as much as possible, and something that should never make it into any production systems is what's commonly referred to as **N+1** calls where a separate IO is initiated for each item in an unbounded collection.

I have never liked how C# method calls have historically been used to invoke Remote Procedure calls, in my opinion this is deceptively bad practice as a remote method call is millions of times slower than a C# method call and for this very reason developers should be encouraged to batch all remote calls where ever possible. Unfortunately WCFs use of a C# [ServiceContract] interface to define your web services effectively encourages the opposite which is what has led to the proliferation of chatty client-specific APIs. [Microsoft even recognizes this fact](http://msdn.microsoft.com/en-us/library/ff649585.aspx) but unfortunately this important recommendation is lost in the sea of documentation in MSDN and since WCF's api encourages the opposite it is effectively ignored by most developers.

For those interested I've spent the last couple of years developing a [replacement web service framework to WCF](http://www.servicestack.net) that's built around [Martin Fowlers DTO pattern](http://martinfowler.com/eaaCatalog/dataTransferObject.html) tuned for best-practices and speed.

If for whatever reason you're unable to batch your calls then the best way to minimize the total time taken to complete multiple IO calls is to run them concurrently which would allow you to complete the task in close to the cost of a single call, rather than the sum of all calls. In terms of resource usage this is still fairly inefficient so this is one area where you should consider your caching options.

### 3. Polling 

I will leave this short because it largely doesn't affect the general .NET populace but for those that do it's still a very important topic and requires further investigation. Effectively polling/pull-based systems are generally less efficient then their event-driven, push-based counterparts. Not only will it result in more IO requests and resources to achieve the same result, but it can also elongate the total time to complete a multi-hop request, quite substantially. If you're connecting disperate systems together within the same network then you should be evaluating whether the use of message queues will ultimately provide a superior solution. One book that I recommend reading on this topic which explains the benefits of a messaging approach is [Enterprise Integration Patterns](http://www.eaipatterns.com/eaipatterns.html). Unfortunately even though the use of messaging is a common practice in the Java world, it is often an overlooked discipline in .NET. I generally attribute this to the limited capabilities of Microsoft's MSMQ and the resulting weak focus and attention it gets next to Microsoft's premier WCF/ASP.NET technologies. However as this remains an important technology I'll provide links:

  * [RabbitMQ](http://www.rabbitmq.com/) - Open source, popular and the highly regarded AMQP implementation
  * [NServiceBus](http://www.nservicebus.com/) - Provides many of the missing add-on features on top of MSMQ. Now turned commercial but has a free licence.
  * [MQ in WCF](http://msdn.microsoft.com/en-us/library/ms731089.aspx) - The MQ endpoint in WCF

Other worthy mentions are the very expensive but long-time leaders in this space from [Tibco](http://www.tibco.com/company/news/releases/2003/press587.jsp) and [IBM](http://www.redbooks.ibm.com/abstracts/sg247012.html). I'll also note the other popular Open source option [ActiveMQ](http://activemq.apache.org/) but due to the issues resulting from the immature .NET clients I've personally witnessed at 2 major companies - use with caution.

Outside of the network boundaries of the corporate intranet, polling over HTTP largely exists out of necessity since its the only option to provide real-time notifications to all browsers over firewalls with JavaScript turned on. A new standard that looks to replace this inefficiency with a bi-directional protocol over HTTP is Web Sockets which should be considered if you plan on providing real-time notifications on the Internet. Due to the potential high-number of concurrent connections the best solution would be to use a non-blocking solution where connections are cheap.

### 4. NoSQL solutions

This topic is too broad to be able to cover in any meaningful way in a short passage so I'm not even going to try. Since NoSQL solutions has more to do with scaling than performance I may cover the .NET story in a future article, that is, if I ever finish writing this one :). Suffice to say that nearly every popular site operating at **Internet Scale** (ref: Web 2.0 buzzwords) is not using an RDBMS. This is likely not going to be you but if your RDBMS is proving to be the bottleneck you may want to research the technology. Here are some starting points:

  * [NoSQL on WikiPedia](http://en.wikipedia.org/wiki/NoSQL) - Definition of NoSQL on WikiPedia
  * [nosql-database.org](http://nosql-database.org/) - An entire site devoted to the technology
  * [highscalability.com](http://highscalability.com) - Resources on scaling in the real-world w/ large % of NoSQL usages
  * [NoSQL at Twitter](http://www.slideshare.net/kevinweil/nosql-at-twitter-nosql-eu-2010) - Great and entertaining intro on how NoSQL is used at Twitter.

**(Full Text) Search**

Although the NoSQL world can offer performance, scalability and productivity improvements, you needn't have to leave the comfort zone of your RDBMS as an [agressive caching strategy](http://highscalability.com/blog/2011/3/3/stack-overflow-architecture-update-now-at-95-million-page-vi.html) and [good servers](http://highscalability.com/plentyoffish-architecture) can support event the largest loads and remember even Facebook started on MySQL. 

Having said that, if your application/website needs to search on any level you're likely to hit full-text search limits built-into your RDBMS, so getting to know a dedicated search engine provider and implementing it sooner rather than later might save some sever heartaches.

  * [Lucene.NET](http://lucene.apache.org/lucene.net/) - .NET port of [Lucene](http://lucene.apache.org/), a brilliant opensource java full text search engine library. StackOverflow might be the most famous [Lucene.NET implementers](http://blog.stackoverflow.com/2011/01/stack-overflow-search-now-81-less-crappy/) after getting fed up with MSSQL's limited full text capabilities.
  * [Sphinx](http://sphinxsearch.com/) - Sphinx is an open source full text search server, designed from the ground up with performance, relevance (aka search quality), and integration simplicity in mind. [.NET client available](http://code.google.com/p/sphinx-dotnet-client/)
  * [Solr](http://lucene.apache.org/solr/) - Where Lucene is the engine, Solr is the server. Solr extends the raw power of Lucene with an XMLRPC based web interface adding many stemmers, tokenizers and analyzers. [SolrNet is a mature client](https://github.com/mausch/SolrNet) for .NET well worth checking out.
  * [ElasticSearch](http://www.elasticsearch.org/) - ElasticSearch is a REST based search engine also based on Lucene. It competes with Solr in the same area but being relative new means it's feels more polished in certain area's where in some it may still has to play catch up to Solr. Speed of development is astonishing though. Currently has two not very mature clients [ElasticSearch.NET](https://github.com/medcl/ElasticSearch.Net) and [NEST](https://github.com/Mpdreamz/NEST).
  * [Riak Search](http://wiki.basho.com/Riak-Search.html) - Deserves a mention.

It's important to note that, although for some time the Lucene.NET project was not actively developed, since July 2011 development is restarted and the features and overall progress are quite on par with the original Java version.

## General Tips and Strategies

This section is reserved for general performance tips and strategies that I hope to maintain with the help of other contributors. I'll start it off with some general adhoc advice:

### Prefer ASP.NET MVC over ASP.NET Web Forms

ASP.NET Web Forms behind the scenes is actually a mature and performant technology and it's compositional development model is still a good fit for developing internal LOB applications. My issue with it is that I consider it to be an **anti-web** technology, where its thick view-state and post-backs for everything prohibits the inherent caching benefits in HTTP and encourages poor web development practices. I'm not denying that you can fight against the grain and make ASP.NET apps perform well, but I am recommending the move to the shinier and newer web frameworks from Microsoft in [asp.net/mvc](http://www.asp.net/mvc) and [asp.net/webmatrix](http://www.asp.net/webmatrix). 

Whilst on ASP.NET if you are making heavy use of Sessions you're likely to run into this [degrading performance issue](http://stackoverflow.com/questions/3629709/i-just-discovered-why-all-asp-net-websites-are-slow-and-i-am-trying-to-work-out) - check the link for potential fixes. I dodge this issue by avoiding ASP.NET sessions completely by making use of my [caching providers](https://github.com/ServiceStack/ServiceStack/wiki/Caching).

#### OWIN
Note worty: a new vibrant effort is underway in **[OWIN](https://github.com/owin/gate/wiki)** driven by many devs in pro web devs in the .NET web community to create a new HTTP abstractions inspired from Ruby's Rack and Python's Paste frameworks - so [watch this space](https://github.com/owin/gate/wiki)!

### Blobbing in RDBMS's

This is probably a contentious recommendation since it goes against the RDBMS normalization teachings in academia. However from the real-world school of hard knocks comes blobbing, which is not only most of the time a performance win, but its friction-free [schema-less traits](https://github.com/ServiceStack/ServiceStack.Redis/wiki/MigrationsUsingSchemalessNoSql) can also make it a productivity win too -  so look for it in your local ORM provider. Caution: since blobbing effectively inhibits its participation in server-side SQL querying you should use it only to hold non-aggregate root meta-data that you don't intend to query on.

TODO: Fill living part of the document with contributions from .NET community <- hint, hint! ;)

----------

## Finale

I think I'll wrap it up here, I hope this had covered the main areas that I think would yield the greatest value to backend .NET developers. I had originally intended for this piece to contain both performance and scalability tips (which although related, are not too similar), but looking back know this was never going to happen. Based on the interest, contributions and feedback I may consider doing a similar piece on Scalability. 

### Contributions Welcome!

If you are an interested .NET developer and want to contribute to this page I invite you to fork this project and send me pull requests to this effect, otherwise feel free to send tweets to **[@demisbellot](http://twitter.com/demisbellot)** on twitter.
