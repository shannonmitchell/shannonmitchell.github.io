########################
Troubleshooting Methods
########################

I have working on electronic and linux systems for about 20 years now.  I don't
and never will claim to be an expert on anything due to the field being in 
constant flux, but I have gotten "decent" at troubleshooting in general.  I have
always had trouble seeing things in my minds eye [#f1]_ which leads to overall
memory issues, but allows me to not get lost in the details. When you are lost
in the details, you are not taking the essential step back to look at the bigger
picture needed for troubleshooting a system. And when it comes to electonics or
technology in general, that big picture is the communication path.

The communications path at its source is a flow of electrons. When working on 
electronic systems in the army, that ment breaking out the oscope and placing a
probe into different test points along the path of electron flow to see if 
everything before the probe is working as intended.  The same can be done with 
most technology in general.  Instead of tracing the path of electrons with an 
oscope or voltmeter, we are are using many tools and logs to "probe" points
along the path.  This allows us to see if its working up to that point or not. 

The essense of troubleshooting most systems is knowing the communication path 
and ruling out sections of the path by "probing" a point along it.  So how do
you know where to probe?  I usually go by one of two methods depending on how
much I know the system's communication path. If you know the system, a
binary searchy [#f2]_ is a good way to quickly get to the root of the issue. If
you do not know the system, start from the beginning and follow the 
communication path.



**************************************************
Troubleshooting a Known System Binary Search Style
**************************************************

So lets say that you get a call and a client can't access their web page. In 
this scenario, you know the system and its communication paths. For the sake 
of this section, lets say it looks something like this. 

.. code-block:: none

  client browser <=> dc router <=> dc switch <=> haproxy load balancer <=> multiple nginx webheads <=> multiple php-fpm app servers <=> db

You have no idea where in that path the issue is occuring.  The path between
each component is a good probe point, but where do you start? You could start
from the beginning and waste a lot of time checking each point along the path.
The better approach is to pick a good point in the middle.  In this case in
between the switch and the haproxy load balancer will be a nice place to start.

.. code-block:: none

  client browser <=> dc router <=> dc switch <=> haproxy load balancer <=> multiple nginx webheads <=> multiple php-fpm app servers <=> db
                                                 ^^^^^^^^^^^^^^^^^^^^^

With a binary search, you start in the middle.  If the check is good you move
halfway between the last probe point and the end of the path and check there.
If the check was bad, move halfway between the start of the path and where
you just probed.  Even after the first probe, you have ruled out half of the 
path.  

So in this example, you could do a tcpdump on the incoming interface of the 
haproxy server.  You can have the client test again.  If you don't see the 
packets making it to the haproxy server, you can rule out the haproxy server, 
nginx webheads, php-fpm app servers and the db.  You now know that you will
need to contact whoever is in charge of the router and switch(usually "netsec"),
and have them check for packets on either side of the router.

.. code-block:: none

  client browser <=> dc router <=> dc switch <=> haproxy load balancer
                     ^^^^^^^^^                            


********************************************************
Troubleshooting an Unknown System by Following the Path
********************************************************

Many times when you are working support for larger companies that have many 
accounts, you have no clue what the communication path is.  I see lower level
techs get into a system during a call or ticket and just assume that its using
a server because the customer said it was there or the server was associated 
with a ticket.  Do NOT trust the customer and do NOT trust the ticket.  This 
can lead to a goose chase and many wasted hours.  Many times it can even lead
to breaking a good system while troubleshooting the wrong path. 

Start with getting get good information from the customer and take notes while 
following the communications path.  When I say good information, "I can't hit 
my betterdildos.com site" won't cut it.  Find out where they are connecting 
from and get them to put in the full url they are trying to hit.  Maybe they 
are typing in htttps://betterdildos.com and problem solved as they think the 
tripple ttt works better because they are OCD.

Lets say that its the same system as the binary section, but we don't know 
that yet. So what is the first thing that happens when you type in a url
in the browser?  The hostname needs translated into an ip address.  So 
the first step should be to do a dns query using something like dig to
get the ip address of the server. 

We get the ip address from a dns query and use internal tools to see what device
has that ip. In this instance its a public virutal ip attached to a router. We
now have the first leg of the communication path.

.. code-block:: none

  client browser <=> dc router

We don't have access to the router, so we call netsec.  They take 30 minutes to 
look up configs and see that its natted back to the internal ip address of a
server.  We log into the box and see that its a small server running an hapxory
software load balancer. We know that https means port 443 if you know your basic
port numbers.  We can use something like ss and netstat to see what application
is answering to port 443 on that ip address. You now know the next step in the 
path. 

.. code-block:: none

  client browser <=> dc router <=> switch <=> haproxy server(haproxy service port 443)

You may not know what haproxy is.  A good start for any service is to look in
/etc/<servicename> for it.  We get lucky and see an haproxy config file. After
digging through it, you see the defination for the frontend and see that it goes
to 2 nginx servers on the backend on port 80.  We log into the devices and with
an ss or netstat, we see nginx is listening on those ports. We now know the next 
paths that are taken. 

.. code-block:: none

  client browser <=> dc router <=> switch <=> haproxy server(haproxy service port 443) <=> (nginx1:80, nginx2:80)


We can now log into those servers and check the nginx config to see where the 
next set of hops is going to be.  In this case, we see that nginx has a config 
for betterdildos.com which has a backend balanced between 4 boxes on port 80. We
log into them and see that php-fpm is listening on port 80.  We now know the 
next hop.

.. code-block:: none

  client browser <=> dc router <=> switch <=> haproxy server(haproxy service port 443) <=> (nginx[1-2]:80) <=> (php-fpm[1-4]:80)

If you have been doing this a while, almost all applications utlize some sort of
database. You might be able to look for a php config or grep through the code
for a db config.  You can also use netstat for connections to things like port 
3306 for mysql connections.  We now know the last leg of the path(mostly). We 
really have no idea what custome code running on those php-fpm boxes are doing,
so it could be communicating out to several other places.  


.. code-block:: none

  client browser <=> dc router <=> switch <=> haproxy server(haproxy service port 443) <=> (nginx[1-2]:80) <=> (php-fpm[1-4]:80) <=> db


So you have a good idea of what the communication path is.  You can speed things
up by testing each of the path points while learning them.  This is good if you
have an angry customer on the line.  If its a minor issue where time can be taken,
it might pay to learn the system first then come up with a plan.  Either way, the
most important thing to do is take some notes.  You may find yourself in a serious
rabbit hole where something is going through 40+ hops.  Remembering something like
that is close to impossible.  You can reference the notes taken on each hop if 
needed.  This is also really good to put in tickets or notes on an account as it
can help the next admin to get up to speed on the environment much quicker.

It might looks something like this:

.. code-block:: none

  - customer has issues accessing http://betterdildos.com

  - dig output showing betterdildos.com going to ip 123.123.123.123

  - internal tool shows 123.123.123.123 goes to routerX

  - contacted netsec and 123.123.123.123 is natted to internal address 172.172.172.172

  - 172.172.172.172 is a linux box with id if 12345 with haproxy listening on 172.172.172.172:443

  - haproxy output showing config for 172.172.172.172:443 balancing to 10.0.0.1:80, 10.0.0.2:80
    Saw 500 errors in the haproxy logs during testing.

  - 10.0.0.1:80 and 10.0.0.1:80 are running nginx with betterdildos.com going to 10.0.0.3:80, 10.0.0.4:80, 10.0.0.5:80, 10.0.0.6:80
    Nginx servers up and running.  Seeing 500 erros in the logs from php-ftp services.

  - 10.0.0.3:80, 10.0.0.4:80, 10.0.0.5:80, 10.0.0.6:80 are running the customers code on php-fpm
    php-fpm logs showing database connectivity ussues. Checked netstat output and saw several connecitons to 10.0.0.100:3306 in a bad state

  - 10.0.0.100 looks like a mysql db. 
    After running some tests, it looks like we hit the max connection limit.


From this info, you can see that following the path took you exactly where you 
needed to be to find the issue.  Any admin coming in behind you can take this
and know exactly what is going on if further action is needed. It may seem like
this is lengthy and unneeded, but it will keep you from going on a goose chase
that can cause more problems than you started with.  It will also give you the
confidence to know exactly what the root of the issue is when talking to a 
customer or manager.  Many admins starting out will guess based on past 
experience and try some random troubleshooting steps until they stumble across 
the root cause or guess at it without any real proof.  

Also if you follow the path, you start seeing patterns.  Most systems no matter
what software they are running, follow some general guidelines.  Once you start
seeing what these patterns are, learning new systems can be a breeze.  Once you
know the big picture, getting the details for each probe point can be a man page
or google search away.


.. rubric:: Footnotes

.. [#f1] `Aphantasia <https://en.wikipedia.org/wiki/Aphantasia>`_ 
.. [#f2] `Binary Search <https://en.wikipedia.org/wiki/Binary_search_algorithm>`_
