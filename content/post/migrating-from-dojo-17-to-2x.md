+++
date = "2013-02-07T00:00:00"
draft = "true"
title = "Migrating from Dojo 1.7 to 2.x"
+++
Throughout superbowl weekend I started migrating to a newer version of Dojo, especifically from Dojo 1.7.2 to 1.8.3. One resource that help me a lot through the processs was [Dojo 1.x to 2.0 migration guide](http://dojotoolkit.org/reference-guide/1.8/releasenotes/migration-2.0.html) I must say that I built my own Dojo version, it's supposed to optimize it to your needs but to tell you the truth I just did it for the sake of doing it. 

The [Creating Builds](http://dojotoolkit.org/documentation/tutorials/1.8/build/) tutorial is more than helpful but brings a lot of information, in summary you need to create two build files and run it. I leave you the two files (links to gist) that might be good enough to just build it. 

* [package.json](https://gist.github.com/lnramirez/4733370) 
* [lnramirez.profile.js](https://gist.github.com/lnramirez/f0739ca02fb136d063c6)

Now, when it comes to the changes I had to do to the code I had to remove anything using the good *dojo.somemodule* for *require("somemodule")*, I will explain with code:

In my 1.7 code I had

    var xhrArgs = {
        url: "${pageContext.request.contextPath}/blog/single/" + _id,
        handleAs: "json",
        load: function(data) {return data;},
        error: function(error) {return error;}
    }
    var deferred = dojo.xhrGet(xhrArgs);

as you can see I am using *dojo.xhrGet* in 2.X versions although there's suposed backward compatibility on using dojo. seems like my build file did something phony and totally refrained from liking it. Anyhow at some point it will change so no harm done you had to change it slightly 

    require(["dojo/json","dojo/request/xhr",json,xhr) {
        xhr.get("${pageContext.request.contextPath}/blog/single/" + _id,{handleAs: "json"})
            .then(function(blogentry) {
            }
    }

And that's about it.

Only thing really frustrated me was the use of script. If you go to the about [link](about/) you will see my github and twitter accounts, there's an actual call to each of those sites to retrive my repositories and followers, and tweets and followers respectively. So accordingly to migration link **dojo.xhr* and dojo.io.* have been replaced with dojo/request** but when I tried using them it was impossible.

the correct way to form the url is: 

* https://api.github.com/users/lnramirez?callback=dojo.io.script.jsonp_dojoIoScript1._jsonpCallback

but when I was using dojo/request module the url was getting tagled 

* https://api.github.com/users/lnramirez*[?&]*callback=dojo.io.script.jsonp_dojoIoScript1._jsonpCallback 

and also before *[?&]* characters there was a CR, so beats me what was going on. My wild guess a bug. 

In order to make it work I had to use my old code just on the new fashion

    require("dojo/io/script",script) {
    script.get({
        url: "https://api.github.com/users/lnramirez",
                jsonp: "callback"
        }).then(function(data_) {

and works like a charm.