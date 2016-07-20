+++
date = "2014-08-08T00:00:00"
draft = "true"
title = "Securing your application"
+++
Back in june I access bajoneando only to find that I had a new entry on my blog, which I didn't remember doing it. When I checked the data it was empty, just had the date 6/2/2014. It occurred to me that somebody might have checked my password at github (it was hardcoded) and tried to add something. So I removed it and continue with my life. After a while it happened again. And well it just happened yesterday. *Sigh*

I decided I should invest some time in really finding out what the problem was. 

Well, it just so happened that I didn't secure methods, I had some security in order to prevent users from navigating to some pages but, alas, I forgot to secure POST, PUT, DELETE methods I didn't want to be accessed. 

This is a snippet of my spring security before today's change

    <global-method-security pre-post-annotations="enabled"/>
    
    <http auto-config="true" use-expressions="true">
        <intercept-url pattern="/images/download/**" access="permitAll" />
        <intercept-url pattern="/images/**" access="hasRole('ROLE_ADMIN')" />
        <intercept-url pattern="/resources/**" access="permitAll" />
        <logout logout-success-url="/" />
    </http>

basically I thought I didn't want people uploading images since anyone could do it. Users could go almost anywhere but based on their roles they would not be able to add a new entry. At the http level everything was opened. 

For example, the method to add a new post is:

    @RequestMapping(method=RequestMethod.POST)
    public String addEntry(@ModelAttribute(value="blogEntry" value="/blog") BlogEntry blogEntry) 

so basically you could use postman app and generate a post to http://www.bajoneando.com/blog/ url and insert a new record, which I think is what was going on. More likely a web crawler doing it. I tried it and boom! I added a new record just like the ones I've been deleting lately. 

In order to secure those methods I updated my spring security as so:

    <http auto-config="true" use-expressions="true">
        <intercept-url pattern="/visit/add/**" method="POST" access="permitAll" />
        <intercept-url pattern="/**" method="POST" access="hasRole('ROLE_ADMIN')" />
        <intercept-url pattern="/**" method="PUT" access="hasRole('ROLE_ADMIN')" />
        <intercept-url pattern="/**" method="DELETE" access="hasRole('ROLE_ADMIN')" />
        <intercept-url pattern="/images/download/**" access="permitAll" />
        <intercept-url pattern="/images/**" access="hasRole('ROLE_ADMIN')" />
        <intercept-url pattern="/resources/**" access="permitAll" />
        <logout logout-success-url="/" />
    </http>

Most of my post, put, delete methods you need to have ROLE_ADMIN, except for add a new visit (which I've been tracking) for any user accessing the site through a browser with javascript enabled. Works like a charm. I expect no more ghosts posts in bajoneando.

By the way, while securing my app, I found out that url patterns are sensitive to order, which now that I think so, makes sense. For example I had before:

        <intercept-url pattern="/**" method="POST" access="hasRole('ROLE_ADMIN')" />
        <intercept-url pattern="/visit/add/**" method="POST" access="permitAll" />
        
Any where you go in the app posts a new visit add, but with this configuration it would deny me access. So I switch  to:

        <intercept-url pattern="/visit/add/**" method="POST" access="permitAll" />
        <intercept-url pattern="/**" method="POST" access="hasRole('ROLE_ADMIN')" />

and works fine. One thing I don't like of my current security is that for some reason instead of responding with a 403 http status, responds with a 200 and sends the login html screen as response. Let's see if I want to be so polite with my crawler and let it know it can access those methods. But I don't think so. 

p.s. I think dojo is so 2009, cool kids nowadays use angular, ember or the likes. I have my intentions behind learning react and clojurescript :) that would be awesome. Let's see if it can see the light some day.