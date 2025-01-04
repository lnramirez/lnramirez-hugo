+++
date = "2013-10-27T00:00:00"
title = "From Cloudfoundry to Openshift"
+++
The first time I deployed lnramirez at Cloudfoundry I told myself it wouldn't last forever. I knew it. It happened. On jun 5th I got an email telling me Cloudfoundry V1 was going to be closed on jun 30th and I could migrate my applications to V2 a.k.a. GoPivotal which will be a paid service. Bummer.

I run my own blog as a hobbie and obviously I haven't been able to monetize it at all. There's no way I was going to pay it. After it was shutdown on june 30th I sent an email to GoPivotal team requesting they send me a backup of my MongoDb database, they were really nice and send me dump files. 

I had a couple of ideas where to migrate, needed a PaaS that would allow me to deploy lnramirez with MongoDb available. 

* Openshift
* Amazon Web Services
* Google App Engine

I knew I could deploy a J2EE web application at Google App Engine but MongoDb was out of the picture.

Amazon could be of use but doesn't come with out spending money, unless you run micro-instances which they come for free but only for a year. A co-worker told me I could create a new account every year but that seems a little bit stressful at least once a year.

Finally, Openshift would be my best solution but came with issues as well. In order to deploy a J2EE app you could use Openshift integration with Maven but I happened to use gradle for my building system. There must be a way to address all this but it would require certain work done. Once more, bummer.

After procrastinating for four months I told myself was enough. I was reading couple of links and out of the blue I bumped into an article explaining how to  [Run Gradle Builds on OpenShift](https://www.openshift.com/blogs/run-gradle-builds-on-openshift). Shekhar Gulati has become my personal savior, I won't bother explaining how I keep using gradle as build tool and being able to deploy in OpenShift, Shekhar does a pretty good job. 

But as usual something came up. When I was deploying in CloudFoundry you upload a WAR file. As easy as that. Unfortunately in OpenShift is a little bit more involved, you use git uploading your whole project, building it and eventually deploying it. Even though, thanks to Shekhar, I was closer to be able to deploy the application I couldn't due to a dependency to one of my own artifacts *bajoneandotags*. Beofre I hadn't had that problem because the WAR I was uploading was, well, packed already with all dependencies, now I was able to obtain dependencies through gradle but what about my artifacts? they are not mantained by anyone but me. 

I eventually gravitated towards [Marcello de Sale's Blog](http://marcellodesales.wordpress.com/2012/04/24/managing-and-building-version-controlled-maven-repos-using-git-gradle-and-nexus-server/) and after trial and error I was able to deployed my snapshots artifacts in my own github. Awesome! [https://github.com/lnramirez/lnramirez-mvn-repo](https://github.com/lnramirez/lnramirez-mvn-repo). 

Given that instead of using my local maven I only had to add reference to the newly created mvn repo

    maven {
      url "https://github.com/lnramirez/lnramirez-mvn-repo/raw/master/snapshots"
    }

and finally I was able to deploy once more lnramirez into the internet. Java compile once run everywhere stand for it self but build once deploy everywhere is still far from being reality. Nonetheless, I must say it surprised me that I was able to do it. 

Let's see how this adventure goes with Openshift. Red Hat is a well respected tech company that stands for being open and knows how to monetize money from open source or thing where no one sees an angle. Hopefully lnramirez has found a new haven.

P.S. can you tell I am using [Twitter Bootstrap?](http://getbootstrap.com/)