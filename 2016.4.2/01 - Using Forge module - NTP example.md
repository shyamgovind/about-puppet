## Using a forge module - An NTP example

This page will give you a first hand example of configuring an NTP service ( as a server or client ) through Puppet. We will also try to understand why using a forge module sometimes makes sense. There's a pretty decent quick start guide [here](https://docs.puppet.com/pe/latest/quick_start_ntp.html) on NTP from Puppet itself. You may want to look at that first. If it serves your purpose, you need not read any further here. 

_Note_ :
_This doc is for a different audience than the quick start guide. It is for someone who has Puppet installed and has a basic idea on how Puppet works, has seen some basic puppet code. He/she knows what a class, module and resource in puppet is, but doesn't have much experience using it._

### NTP - the manual way

So, let's first setup NTP manually and see what exactly are we automating. If you know manual setup of NTP, you can skip to [next section](#ntp---the-puppet-way).

#### A little about NTP in Linux

So, NTP ( Network Time Protocol ) is way to ensure the clocks of your machines are in sync. That's all ! 

The NTP server listens on port 123 and uses UDP. The NTP clients send requests to NTP server to sync its time with it. The config file used by both NTP server and client is _/etc/ntp.conf_ ( for RedHat and Ubuntu systems ). The NTP service ( called _ntpd_ ) acts as a server and/or a client depending on its configuration.

There are two packages that are most commonly talked about in NTP :

1. _ntp_ package, which sets up the _ntpd_ service. 

    The _ntpd_ service runs in the background as a deamon and queries the configured NTP servers periodically to keep time in sync. It will also act as a server if it is configured to accept requests.
    
2. _ntpdate_ package, which is to manually sync time with a NTP server.

    The _ntpdate_ package gives a command of same name to query NTP servers.
    
#### Setting up NTP in RedHat based system.

Let's setup a NTP server and a NTP client. For both we need the same package _ntp_ and package _ntpdate_. Let's install it.

```
[On Server & client]
# rpm -qa | grep ntp
#

# yum install ntp ntpdate
#

# rpm -qa | grep ntp
ntpdate-4.2.6p5-25.el7.centos.1.x86_64
ntp-4.2.6p5-25.el7.centos.1.x86_64
#

```

Get the ipaddress for both Server and Client
```
[On server]
[root@codemaster ~]# facter ipaddress
192.168.138.131

[On client]
[root@centos7agent ~]# facter ipaddress
192.168.138.129
```

**A. Configuring the NTP server**
   
On the machine you want to make as the NTP server, edit the _/etc/ntp.conf_ as follows :

```
[root@codemaster ~]# vim /etc/ntp.conf

# Search for line "restrict" and add something like the following, to allow traffic from local IPs.
    
    restrict 127.0.0.1
    restrict ::1
    restrict 192.168.138.0 mask 255.255.255.0 nomodify notrap   # this is the line that makes it an NTP server.

# We are basically asking the "ntpd" service to start listening and responding to requests from a certain set of IPs.
# Explanation about IP values : 
# My client nodes are in the IP range 192.168.138.0-255. Hence, the above IP and mask. 
# You could give a broader range if you need to. like 192.168.0.0 & mask 255.255.0.0

```

Your local NTP server would typically connect to some other external NTP server like below, to keep its time in sync. So, the ntp.conf would also have some entry like this :

```
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
```
 
That's it. Start the _ntpd_ service to start the NTP server. 

![NTPd status](https://github.com/shyamgovind/puppet-cheat-sheets/blob/master/img/NTPd%20status.png)


**B. Configuring the NTP client**

Now let's do the same for all the NTP client nodes. Assuming we have already installed the _ntp_ and _ntpdate_ package, here's what we would configure in the _/etc/ntp.conf_.

Since we do not want these nodes to act as servers, the "restrict" line is left as it is. We change the "server" lines as follows :

```
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
server codemaster.shyam.net     # this is our NTP server configured previously i.e. 192.168.138.131
```

That's it. The node is ready as an NTP client. If you start the _nptd_ service, it will periodically check with the NTP server _codemaster.shyam.net ( 192.168.138.131 )_ to sync time.

You can also use _ntpdate_ to manually sync time with any NTP server.

![ntpdate](https://github.com/shyamgovind/puppet-cheat-sheets/blob/master/img/ntpdate%20command.png)

**C. Common issues**

- Error 1 : the NTP socket is in use, exiting

If you run the _ntpdate_ command without using -u option and while ntpd is running in the background, you'll see this error. It is because the _ntpd_ service is already using the port. -u makes it use any other port for querying.

![error 1](https://github.com/shyamgovind/puppet-cheat-sheets/blob/master/img/ntp%20error%201.png)

- Error 2 : no server suitable for synchronization found  ( Server dropped: strata too high )

Now there could be many reasons for this error. First step is to run ``` ntpdate -dv codemaster.shyam.net``` to know the error in more detail. The _"Server dropped: strata too high"_ is very common error. If you look closely you should see the _stratum 16_ entry ( see screenshot of error below ).

Meaning of strata is best explained [here](http://serverfault.com/questions/277375/ntpdate-d-server-dropped-strata-too-high)

>NTP increases the stratum for each level in the hierarchy - a NTP server pulling time from a "stratum 1" server would advertise itself as "stratum 2" to its clients.
>
>A stratum value of "16" is reserved for unsynchronized servers meaning that your internal NTP server at 192.168.92.82 thinks not to have a reliable timesource (i.e. not synchronizing to a higher-level stratum server).

Here's how this error would look like :

![ntp strata error](https://github.com/shyamgovind/puppet-cheat-sheets/blob/master/img/ntp%20strata%20error.png)

   **Solution**
   
   Since the issue is that your NTP server is not able to reach any other NTP server for syncing its time, all we need to do is make sure the NTP server ( _codemaster.shyam.net_ ) is able to reach another NTP server. Once it's time is synced, the NTP clients connecting to it will be able to sync with it.
    
![ntp strata solved](https://github.com/shyamgovind/puppet-cheat-sheets/blob/master/img/ntp%20strata%20solved.png)


### NTP - the puppet way

So, what we are trying to automate here through puppet is :
 - install ntp package
 - configure ntp ( server or client ) 
 - run the ntp service in the background.

We have two options here :

   1. Write our own small module something like this :

    ```
    class setup_ntp 
    ( $is_client = true,
    ){

    # decide if the setup is for NTP client or server
    if $is_client {
        $source_file = 'puppet:///modules/setup_ntp/ntp.conf.client'  
            # we'll keep a ntp.conf.client in files/ folder, to copy to /etc/ntp.conf and configure an NTP client 
      }
      else {
        $source_file = 'puppet:///modules/setup_ntp/ntp.conf.server'  
           # ntp.conf.server in files/ folder will be copied to /etc/ntp.conf to configure an NTP server.
      }

    # Install ntp package
    package { 'ntp' :
      ensure => installed,
    }

    # Put the right configuration
    file { '/etc/ntp.conf':
      ensure => file,
      source => $source_file,
      require => Package['ntp'],  # requires the package to be installed first, before trying to configure.
    }   

    # Start service and enable on boot
    service { 'ntpd':
      ensure => running,
      enable => true,                     # enable on boot
      subscribe => File['/etc/ntp.conf'], # instructs puppet to restart service "ntpd" if the file content changes.
    }


    }
    ```
    
    But the issue with this method is that our code is very basic & inflexible. It also doesn't handle all the different scenarios. What if you needed a different ntp.conf for a certain set of NTP clients ? What if you wanted to handle Solaris servers as well ? Or may be some AIX servers ? 


   2. Forge to the rescue. 
    
    Puppet forge as you might know, is a community repository of puppet modules. Let's look at ntp modules available there. Searching "ntp" on forge would give us this : https://forge.puppet.com/puppetlabs/ntp. _( Tip : Always use "[puppet supported](https://forge.puppet.com/supported)" modules if available. Next in line are "[puppet approved](https://forge.puppet.com/approved)" modules. )_ Using a forge module esp. ones like puppetlabs-ntp has a lot of benefits :
    
    ![forge ntp](https://github.com/shyamgovind/puppet-cheat-sheets/blob/master/img/ntp%20forge%20module.png)


You would notice, it has good documentation on how to use it too. So, let's install the module :
 
   ![installing module](https://github.com/shyamgovind/puppet-cheat-sheets/blob/master/img/installing%20forge%20ntp.png)

Now, our code is much simpler and would just involve calling the "ntp" class in this module :

```
[For NTP clients]
class { '::ntp':
  servers => [ 'codemaster.shyam.net'],
}


[For NTP server]
class { '::ntp':
  servers   => ['0.centos.pool.ntp.org', '1.centos.pool.ntp.org', '2.centos.pool.ntp.org', '3.centos.pool.ntp.org'],
  restrict  => [
    'default nomodify notrap nopeer noquery',
    '127.0.0.1',
    '::1',
    '192.168.138.0 mask 255.255.255.0 nomodify notrap'
     ],
}
```

That's it. You can call this class, like above, from PE console _or_ write it in another class _or_ classify in site.pp to apply globally. Best thing is, unlike our code this works for all the supported platforms - not just RHEL and Ubuntu.

_( Don't fret, if you don't know all the ways ! PE console is enough in most cases. Create Node groups for NTP servers and clients. Attach the "ntp" class to them. The "servers" & "restrict" go as parameters to the class "ntp". You just have to copy the rest as values to the parameter in the console, based on what you are configuring. )_

![passing parameters](https://github.com/shyamgovind/puppet-cheat-sheets/blob/master/img/passing%20parameters%20ntp.png)


And you are now managing NTP via Puppet ! Congratulations ! :)

