## Using a forge module - An NTP example

This page will give you a first hand example of configuring a NTP service ( as a server or client ) through Puppet. There's a pretty decent quick start guide [here](https://docs.puppet.com/pe/latest/quick_start_ntp.html) from Puppet itself. You may want to look at that first. If it serves your purpose, you need not read any further here. 

_Note_ :
_This doc is for a different audience than the quick start guide. It is for someone who has Puppet installed and has a basic idea on how Puppet works, has seen some basic puppet code. He/she knows what a class, module and resource in puppet is, but doesn't have much experience using it._

### NTP - the manual way

So, let's first setup NTP manually and see what exactly are we automating. If you know manual setup of NTP, you can skip to next section.

#### A little about NTP in Linux

So, NTP ( Network Time Protocol ) is way to ensure your machines are in sync w.r.t time. That's all ! 

The NTP server listens on port 123 and uses UDP. The NTP clients send requests to NTP server to sync its time with it. The config file used by both NTP server and client is _/etc/ntp.conf_ ( for RedHat and Ubuntu systems ). The NTP service ( called _ntpd_ ) acts as a server and/or a client depending on its configuration.

There are two packages that are most commonly talked about in NTP :

1. _ntp_ package, which is sets up the _ntpd_ service. 

    The _ntpd_ service runs in the background as a deamon and queries the configured NTP servers periodically to keep time in sync. It will also act as a server is it is configured to accept requests.
    
2. _ntpdate_ package, which is to manually sync time with a NTP server.
    The _ntpdate_ package gives a command of same name to query NTP servers.
    
    
 
