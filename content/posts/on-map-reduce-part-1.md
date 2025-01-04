+++
date = "2012-06-29T00:00:00"

title = "On Map Reduce - Part 1"
+++
One of the hype things nowadays is Map Reduce, I won't talk about it, but I'll leave you this [useful link](http://lmddgtfy.co.cc/?q=map+reduce) that you'll definitely find interesting.

I don't have a particular necessity on using a map reduce for my big data, but I thought I could experiment with it. 

My map reduce is going to return me the latest blog entry based on publishDate. 

So first of all we need to map what are we reducing, in this case everything:


mapallblog.js >

    function () {
        emit("blogEntry", this);
    }

emit function takes to parameters, key and value. We'll use my collection's name just because, and we are emitting the current value on the second parameter. 

Later on, we need to define a reduce function. This one, will reduce all emission under the same key to a single object. We return the one with older publishDate. 

reducelatestblog.js >

    function (key, values) {
        var latest = values[0];
        for (var i=0;i<values.length;i++) {
            if (latest.publishDate < values[i].publishDate) {
                latest = values[i];
            }
        }
        return latest;
    }

And that's pretty much the magic under mongo map reduce function. I'll comeback to finish with the Spring side