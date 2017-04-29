This folder has all the useful info related to 2016.4.* PE release.

Note : 2016.4.2 is the first LTS release by Puppet.
 
The support cycles details are listed here : https://puppet.com/misc/puppet-enterprise-lifecycle



Puppet component versions for PE 2016.4 release ( ignoring patch versions i.e. "z" in x.y.z )

_P.S : Not all components but ones that matter in most cases._

For PE 2016.4 :

Main puppet components -
- Puppet        : 4.7   ( the main ruby application that does all the magic - Master. )
- Puppet Server : 2.6   ( the web server that hosts the above ruby app. A spin-off of Jetty web server ) 
- Puppet agent  : 1.7   ( the application that acts as an agent )
- Puppet DB     : 4.2   ( Database for puppet. A spin-off of Postgres. )
 
Others -
 
- Facter        : 3.4   ( a separate tool to collect and use system info. )
- Hiera         : 3.2   ( a tool to access and use data from text files. )
 
