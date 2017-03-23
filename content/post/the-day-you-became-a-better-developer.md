+++
date = "2017-03-03T14:00:00"
draft = "true"
title = "The day you became a better developer"
+++

I didn't pass the phone interview. I was sad, frustrated and went full cognitive dissonance. I thought the interviewer was a jerk, he had his facts wrong and I did an amazing interview so they were going to call me. It didn't happen.

After a couple of days of reiterating how it went, blow by blow, I realized that definitely I had made a fool of myself. It was basic object oriented programming questions and algorithms:

* what's the difference between overloading vs overriding. *I answered it with concepts exchanged.*
* how can you transfor this list ["2", "3", "4"] into this [30, 41, 52] *. *I spent 15 minutes spinning my wheels.*
* if you have 10 million numbers but you could only load 3 million in memory how could you find top 10 numbers?. *I didn't know how to even start.*
* ok, how could you sort a list?. *A day before I read sometimes they ask that so I answered, you use quicksort then he asked me to implement it. I totally forgot.*

I felt like a total failure and a charlatan. How can you say you are a developer for the last 10 years and can't even answer one of these questions?

Truth be told, most developers go winging it, why am I using this? what does it mean? who did this? is not something we all ask everyday. In Coders at Work [Siebel 2009] Guy Steele says that when the computer had 4,000 words of memory it was feasible for you to know everything about the machine, nowadays are you really going to know all the software that runs in your machine? are you going to do a core dump? I wouldn't think so. But you should know a few concepts behind it.

The interesting thing about this company is they were not looking for an object oriented developer but for a functional programmer, their rationalization I suppose is that if you grasp the OO concepts you will have a good start at functional programming. If you don't, well, you probably can't evolve to lambda.

That was the day I became a better developer, I realized that no matter how many years you've worked and you've delivered software, you can fail. You must try to keep learning new technologies but you better learn what you use everyday. I started asking more myself, why am I using an interface?, what is really dependency injection? How does my machine know what ip is the address for this website?

Since that day I devoted myself to functional programming, it has became part of my identity.

--

If you like comeback stories you might want to [hire me](http://www.linkedin.com/in/lnramirez) because I interviewed again with same company two years later and received and offer from them.

--

*I'm not totally sure that's exactly the problem they asked me but it's somewhere in the neighborhood:

`["2", "3", "4"] ~> [20, 31, 42]`

if you see they are numbers as string, so to go from the imput on the left to the right you can use a List

    scala> val l = List("2", "3", "4")
    l: List[String] = List(2, 3, 4)

Then you can parse the elements

    scala> l.map(x => Integer.parseInt(x))
    res1: List[Int] = List(2, 3, 4)

now that you have numbers you can multiply that by 10

    scala> res1.map(n => n*10)
    res2: List[Int] = List(20, 30, 40)

the trick is now to add 0 to first element, 1 to second, 2 to third. That means there must exist a list like structure if you will `(0 to 2)` or for that matter a Stream but lets keep it simple here:

    scala> 0 to 2
    res3: scala.collection.immutable.Range.Inclusive = Range 0 to 2

now we could zip both seqs

    scala> res2.zip(res3)
    res4: List[(Int, Int)] = List((20,0), (30,1), (40,2))

almost there, we only need to transform it one more time by adding the values

    scala> res4.map(x => x._1 + x._2)
    res5: List[Int] = List(20, 31, 42)

follow all that above if you are not into the whole brevity thing, if not:

    scala> List("2", "3", "4").map(Integer.parseInt(_))
               .map(_ * 10).zip(0 to 3)
               .map(x => x._1 + x._2)
    res13: List[Int] = List(20, 31, 42)
