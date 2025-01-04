+++
date = "2012-07-03T00:00:00"

title = "On Map Reduce - Part 2"
+++
[On Map Reduce - Part 1](http://lnramirez.cloudfoundry.com/blog/4fedfebe4a14fb8d9f5e4e07/On%20Map%20Reduce%20-%20Part%201) I introduced the use of a map reduce to retrieve the latest entry on the blog. 

The document where we store blog entries looks like


    @Document
    public class BlogEntry implements Serializable {

        private String id;
        private String subject;
        private String article;
        private Date publishDate;
        private Date lastUpdateDate;
        private String printableHtml;
        //more code continues
    
    }

Spring part as I stated before is pretty straight forward, on my service I added getLastEntry method

    @Service
    public class BlogEntryService {

        public BlogEntry getLastEntry() {
            MapReduceOptions options = new MapReduceOptions();
            MapReduceResults<MappedBlogEntry> mapReduceResults = mongoTemplate.
                    mapReduce("blogEntry", "classpath:mongo/mapallblogs.js" , 
                    "classpath:mongo/reducelatestblog.js", MappedBlogEntry.class);
            if (mapReduceResults.iterator().hasNext()) {
                BlogEntry blogEntry = mapReduceResults.iterator().next().getValue();
                return blogEntry;
            } else {
                throw new RuntimeException("No entries for blogEntry");
            }
        }

        @Autowired
        private MongoTemplate mongoTemplate;

    }

In order to understand it we need to recall what our map and reduce functions looks like

mapallblog.js >

    function () {
        emit("blogEntry", this);
    }

on mapallblog it's emitting the whole object as it is with all it's properties.

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

then on reduce function we just select the one with the oldest publishDate and return it. 

But what mongo does behind the scenes is map it and pair it with a reduction, hence the result of this called will be an object 

    {
    id:"blogEntry",
    value:emittedObject
    }

we can map the result with another object

    public class MappedBlogEntry {

        private String  id;
        private BlogEntry value;
        //more code continues

    }

Going back to the actual spring call

            MapReduceResults<MappedBlogEntry> mapReduceResults = mongoTemplate.
                    mapReduce("blogEntry", "classpath:mongo/mapallblogs.js" , 
                    "classpath:mongo/reducelatestblog.js", MappedBlogEntry.class);
            if (mapReduceResults.iterator().hasNext()) {
                BlogEntry blogEntry = mapReduceResults.iterator().next().getValue();
                return blogEntry;
            } else {
                throw new RuntimeException("No entries for blogEntry");
            }

mongoTemplate.mapReduce fourth parameter is a class to which map the results this case MappedBlogEntry, and that's about it. Every time you access home screen a map reduce function is evaluated.