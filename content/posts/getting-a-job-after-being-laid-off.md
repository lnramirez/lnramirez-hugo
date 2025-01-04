+++
date = "2017-05-25T14:00:00-04:00"
title = "Getting a job after being laid off is tough"

+++

- But before we start please tell me, why are you looking for a job?
- well, I was laid off this Monday, so I'm here looking (Wednesday)
- ...

That my friends is like the worst way to tell anyone you were recently laid off and are back in the market. After those words came out of my mouth I knew very well that nothing good could come up from that phone screening.

The interviewer, a senior engineer, started very strongly with good enthusiasm. He was cheerful and I had a good vibe. But I was just two days after my abroupt departure from my company and probably hadn't stop to think my next step. To be honest I've been looking for a job for the last 3 months when the company did another round of lay offs, but it's not the same see it coming that being in the situation.

It's not like I haven't been without a job looking for a position. Actually, two years ago I decided it was better for me and the company to part ways. I quit. Everytime someone asked me then, why did you leave? I'd respond *I want to find a place I really like and we had creative differences with my last company*, and recruiters and engineers were like shrugging their shoulders and moving on. Now is like first of all, a show stopper if you deliver that bit of information in bad timing, second everybody drops their energy and say *wow, that's bad, I'm sorry to hear that* and so on.

As a student of persuasion and pre-suasion I know this is bad. So the best I have come up with is start by saying the company history and how good it was and how quickly it didn't go so good for the company, myself and lots of talented people. I know that I need to put a lot of energy into trying to re-frame the situation, mostly that I feel positive about -which by the way is true- and that we should move on. With my A-B testing I think it's been working. Since I've been re-framing it better I get less of a panic reaction from the other side and the conversations for the most part go uneventful after saying I was laid off.

--

People take emotional decisions most of the time, I believe phone screenings or on site interviews are no different. People will know very early on the process if they would hire you or not, the facts, in this case how you solve an algorithm just work for post-rationalizing the decision taken.  I know, I've done my fair share of interviewing too.

In my case it went like this. The engineer on the other side asked me for a simple coding question, anagrams, nothing out of the ordinary. The funiest of all is I solved it exactly as it's described here [ardendertat](http://www.ardendertat.com/2011/11/17/programming-interview-questions-16-anagram-strings/).

First I attempted to sort both strings and evaluated for equality. Mr Dertat calls that an elegant solution. Pat in the back, cough, cough. Interviewer was not satisfied since I solved it in less than five minutes, so moving on. Once he wanted to pivot on how else to solve it I attempted to solve it with maps. For each string I would build a map, keys on the map represent characters and values for each key will be a count on repetitions of the character, once you build both maps check again for equality.

Here's what I came up with on my feet:

    def createMap(s : String, m : Map[Char, Int]) : Map[Char, Int] = {
      val h = s.headOption
      if (s.isEmpty) m
      else {
        val c = m.get(h.get)
        val min : Int = 1
        if (c.isEmpty) createMap(s.tail, m + ((h.get.toUpper, 1)))
        else  createMap(s.tail, m + ((h.get.toUpper, c.get + 1)))
      }
    }

It wasn't the most optimal because I could have continue improving it but was a good point to check up if this solution was on the venue he was looking for. Unfortunately my interviewer was having hard time to decipher my recursive solution. In functional world we love recursion, is immutable, tail call will ensure compiler optimizes it as a loop and everyone happy.

There are a couple of errors I made:

* I had totally forgotten how to update a map in scala, instead of using `+` I used `put` which I'd say should not matter that of a much
* get method on map returns you already an Option[T] but I used instead m.getOption, that doesn't exist
* that ugly if else can be worked out with map getOrElse but it wouldn't be in tailrec so that'd depend on your trade off
* He asked me how could I ignore blank spaces, tabs, carriage returns and I stated that I could use regexpes to solve the problem but then I would have two. Silence again. Tough crowd, I know.

It could be the case that I am once again into [cognitive disonance](../the-day-you-became-a-better-developer/) why I didn't get into the next round. Up to you to decide. I solved the problem in a decent time frame, I asked the interviewer if he liked my solution and he claimed he had trouble finding how would it work, tried to explain to him but he was having really bad time understanding me. I had primed him to be against anything I will do. Then he poked me with running time, which I believe I did it wrongly but I felt at the time he just wanted to find a fake because on why not to pass me to the next round.

I hanged up and knew I wouldn't go into the next round. I don't hold any grudge to my interviewer. I would have done the same if I myself have never been laid off and ran into someone delivering that information the way I did.

--

You might want to [hire me](https://www.linkedin.com/in/lnramirez) because I've been laid off and now you know it's tough.
