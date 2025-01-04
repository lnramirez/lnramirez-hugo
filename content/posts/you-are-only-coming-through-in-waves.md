+++
date = "2013-01-10T00:00:00"

title = "You are only coming through in waves"
+++
Happy new year! it's been such a long time since last post but life has been crazier than ever, what a year I had in 2012, parenthood, two jobs, my blog I could go on all day but moving on. 

I decided to finally upgrade my tools, gradle from b1.0 to 1.1, jdk6 to jdk7, these are my latest tools

    $ gradle -v
    
    ------------------------------------------------------------
    Gradle 1.1
    ------------------------------------------------------------
    
    Gradle build time: Tuesday, July 31, 2012 1:24:32 PM UTC
    Groovy: 1.8.6
    Ant: Apache Ant(TM) version 1.8.4 compiled on May 22 2012
    Ivy: 2.2.0
    JVM: 1.7.0_09 (Oracle Corporation 23.5-b02)
    OS: Mac OS X 10.7.5 x86_64

my steps were:

* update gradle using macport to ver 1.1
* update jdk to ver 1.7 
* build the app again
* update source version in build.gradle
* build the app again
* deploy it back to cloudfoundry+

I've been having more spare time lately so I hope I can introduce a couple of new feautures every now and then. 

\+ well this was not that easy peasy

    $vmc apps
    name            status    usage      runtime   url                             
    caldecott       stopped   1 x 64M    ruby18    caldecott-ae140.cloudfoundry.com
    lnramirez       running   1 x 512M   java      lnramirez.cloudfoundry.com  

as you can see my app had runtime set to java default ver which as of today is 1.6, and no it couldn't hot update it. Dirty but quick fix around: deploy it as a new app lnramirez-new selecting java7 as runtime

    $vmc apps
    name            status    usage      runtime   url                             
    caldecott       stopped   1 x 64M    ruby18    caldecott-ae140.cloudfoundry.com
    lnramirez       running   1 x 512M   java      lnramirez.cloudfoundry.com      
    lnramirez-new   running   1 x 512M   java7     lnramirez-new.cloudfoundry.com  

then map the new version to lnramirez url and remove the old one 

    $ vmc map lnramirez-new lnramirez.cloudfoundry.com
    Updating lnramirez-new... OK
    $ vmc unmap lnramirez 
    1: lnramirez.cloudfoundry.com
    Which URL?> 1         
    
    Updating lnramirez... OK

Voilà! everything good then probably I should remove my old lnramirez but I'll have it around so I can do a hot deployment next time.

probably I could have done using some CF commands but I always wanted to try mapping and unmapping, a couple of times -shame on me- I deploy a newer version and it doesn't work. So this way I know how to do a hot deployment with out affecting my users :)