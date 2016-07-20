+++
date = "2012-06-01T00:00:00"
draft = "true"
title = "Iteration Guaymango release notes"
+++
Iteration Guaymango Rulz!

oh boy! I cannot believe how much work I did this iteration. 

### New features

#### Google code prettify

We added css to our code, thanks to markdown awesome html sintax now something like: 

    @RequestMapping(value="/update", method=RequestMethod.PUT, headers="Accept=application/json")
    @ResponseStatus(HttpStatus.OK)
    public void updateEntry(@RequestBody BlogEntry blogEntry) {
        blogEntry.setLastUpdateDate(new Date());
        blogEntry.setPrintableHtml(markdownProcessor.markdown(blogEntry.getArticle()));
        blogEntryRepository.save(blogEntry);
    }

is possible
 
#### Sitemesh 2 templating

Probably this ain't that cool to the bare eye, nonetheless is so good for developers. Before all pages had embedded css and boiler html code to look alike. Now every thing goes so smooth.

*About* page has only this code

    <html lang="en">
        <head>
            <title>About</title>
        </head>
        <body>
            <header>
                <h2>Bajoneando</h2>
            </header>
            <article>
                <strong>Bajoneando meaning</strong>
                <p>
                    Bajonear is a salvadorean-spanish verb that you use to refer
                    to the action of eating after hours, probably after a party or 
                    a long day at work. 
                    You can use it when you are famished and heading towards to
                    grab a bite, something like <em>I am going to bajonear</em></p>
                <p>
                    I am Bajoneando as you can see in the footer because I usually 
                    program at work only but nowadays I have been feeling like 
                    programming after hours with a cup of joe, nuts, dried fruit 
                    or any paleo snack that I can eat
                </p>
            </article>
        </body>
    </html>

yet produces a beautiful page go to  [About page](http://lnramirez.cloudfoundry.com/about/ "About page") and see it for yourself.

#### Article editor

Well not everybody is supposed to add, edit article but I have left it as is, like I said in the beginning _please don't be mean =)_ anyhow, we can add, update an entry and see a preview immediatly, no refresh =D thanks to the awesome dojo toolkit.

#### Pagination

Yup pagination was added as well, nothing too fancy just a previous and later entries links, I am hopping to write lots more entries everynow and then.  Right now we have 5 entries per page, sorted **DESC** on published Date

#### HTML5

I know, I know, I should not brag about it but my site is HTML5 complaint or at least I check up my generated html at [Markup Validation Service](http://validator.w3.org/ "Markup Validation Service") 