+++
date = "2016-08-05T15:50:18-04:00"
draft = true
title = "On using Hugo and Monger"

+++

The web app was hostend on Openshift before, my data in mongo. I used RockMongo to get a dump of my data. It was a nice .gz file that as luck will have it, I need to read at least five sites before I actually know how to extract it. Yup, if my life depended on my abilities to extract files I'd be in the strees by now. With the file handy and have done all the running mongo in docker I needed to upload the actual data.

    $ mongo localhost:1111
    MongoDB shell version: 3.2.0
    connecting to: localhost:1111/test
    > use anotherdb
    switched to db anotherdb
    > load('mongo-lnramirez-20160718-172713.js')
    true
    > show collections
    blogEntry
    images.chunks
    images.files
    visit

So there I was decided to use Hugo for my blog, all collections uploaded to my local mongo db and all it was required was reading mongo and generate a mongo equivalent.

Mongo is a non-structured repository meaning structures are not rigid but they all are json. My json objects didn't change that much over time.

    > db.blogEntry.find({subject: "Iteration Guaymango release notes"})
    { "_id" : ObjectId("4fc81041be6c1e4069b1046a"), "_class" : "com.bajoneando.lnramirez.blog.BlogEntry", "subject" : "Iteration Guaymango release notes", "article" : "Iteration Guaymango Rulz!\n\noh boy! I cannot believe how much work I did this iteration. \n\n### New features\n\n#### Google code prettify\n\nWe added css to our code, thanks to markdown awesome html sintax now something like: \n\n    @RequestMapping(value=\"/update\", method=RequestMethod.PUT, headers=\"Accept=application/json\")\n    @ResponseStatus(HttpStatus.OK)\n    public void updateEntry(@RequestBody BlogEntry blogEntry) {\n        blogEntry.setLastUpdateDate(new Date());\n        blogEntry.setPrintableHtml(markdownProcessor.markdown(blogEntry.getArticle()));\n        blogEntryRepository.save(blogEntry);\n    }\n\nis possible\n \n#### Sitemesh 2 templating\n\nProbably this ain't that cool to the bare eye, nonetheless is so good for developers. Before all pages had embedded css and boiler html code to look alike. Now every thing goes so smooth.\n\n*About* page has only this code\n\n    <html lang=\"en\">\n        <head>\n            <title>About</title>\n        </head>\n        <body>\n            <header>\n                <h2>Bajoneando</h2>\n            </header>\n            <article>\n                <strong>Bajoneando meaning</strong>\n                <p>\n                    Bajonear is a salvadorean-spanish verb that you use to refer\n                    to the action of eating after hours, probably after a party or \n                    a long day at work. \n                    You can use it when you are famished and heading towards to\n                    grab a bite, something like <em>I am going to bajonear</em></p>\n                <p>\n                    I am Bajoneando as you can see in the footer because I usually \n                    program at work only but nowadays I have been feeling like \n                    programming after hours with a cup of joe, nuts, dried fruit \n                    or any paleo snack that I can eat\n                </p>\n            </article>\n        </body>\n    </html>\n\nyet produces a beautiful page go to  [About page](http://lnramirez.cloudfoundry.com/about/ \"About page\") and see it for yourself.\n\n#### Article editor\n\nWell not everybody is supposed to add, edit article but I have left it as is, like I said in the beginning _please don't be mean =)_ anyhow, we can add, update an entry and see a preview immediatly, no refresh =D thanks to the awesome dojo toolkit.\n\n#### Pagination\n\nYup pagination was added as well, nothing too fancy just a previous and later entries links, I am hopping to write lots more entries everynow and then.  Right now we have 5 entries per page, sorted **DESC** on published Date\n\n#### HTML5\n\nI know, I know, I should not brag about it but my site is HTML5 complaint or at least I check up my generated html at [Markup Validation Service](http://validator.w3.org/ \"Markup Validation Service\") ", "publishDate" : ISODate("2012-06-01T00:00:00Z"), "lastUpdateDate" : ISODate("2012-06-04T13:24:20.582Z")}"

So as I have explained before I was storing already markdown-ish and Hugo format is very simple too, you create a file with an iphenized name to be more url friendly, in the file, at top date, title and some other options

    +++
    date = "2016-08-05T15:50:18-04:00"
    draft = true
    title = "on using hugo and monger"

    +++

    I use RockMongo to get a dump of my mongo data. It was a nice .gz file that as usual I need to read at least five sites before I act

and yes, this is actually recursive ;)

my go to place to look what library should I by using to this or that is http://clojurewerkz.org/ looking up on how to connect to mongo seems like Monger is your best option http://clojuremongodb.info/

I use leinegen project, adding dependencies to monger

      :dependencies [[org.clojure/clojure "1.8.0"]
                     [com.novemberain/monger "3.0.2"]
                     [selmer "1.0.7"]
                     [clj-time "0.12.0"]]

Importing

      (:require [monger.core :as mg]
                [monger.collection :as mc]
                [selmer.parser :refer [render-file]]
                [clj-time.format :as f]
                [clj-time.coerce :as c]

All the action needes is get all the blog entries and write them as md what Hugo expects

    (defn gen-files
      []
      (map #(write-md %) (all-blog-entries)))

Mongo magic happens pretty quick, please don't blink

    (defn all-blog-entries
      []
      ;; connect with authentication
      (let [conn (mg/connect {:port 1111})
            db (mg/get-db conn "test")
            coll "blogEntry"]
        (mc/find-maps db coll {})))

Rest is writting the file Hugo expects, and I wanted to pause here because I was going to use a template generator, again per clojurewerkz I chose selmer but something funky happened because of the characters were generating garbage so I had to rely on good all concatenating, not my brightest moment I know

    (defn hyphenize
      [s]
      (->
          (replace s #"(::|\?|\.|-)" "")
          (replace #"([A-Z]+)([A-Z][a-z])" "$1-$2")
          (replace #"([a-z\d])([A-Z])" "$1-$2")
          (replace #"\s+" "-")
          (replace #"[áa]" "a")
          (replace #"[ée]" "e")
          (replace #"[íi]" "i")
          (replace #"[óo]" "o")
          (replace #"[úu]" "u")
          (replace #"_" "-")
          (lower-case)))

    (defn extract-date
      [d]
      (f/unparse (:date-hour-minute-second f/formatters) (c/from-date d)))

    (defn gen-hugo
      [subject pub-date text]
      (str "+++\ndate = \"" pub-date "\"\ndraft = \"true\"\ntitle = \""subject"\"\n+++\n" text))

    (defn write-md
      [entry]
      (let [subject (:subject entry)
            file-name (hyphenize subject)
            text (:article entry)
            pub-date (extract-date (:publishDate entry))]
        (spit (str file-name ".md") (gen-hugo subject pub-date text))))

    (defn gen-files
      []
      (map #(write-md %) (all-blog-entries)))

With that, all my files were generated, named correctly (for the most part) and didn't have to do anything else. So here we are starting another chapter.
