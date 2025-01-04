+++
date = "2014-04-28T00:00:00"
title = "Composing Scala and Java futures"
+++
For the last two years I have gravitated towards Scala and Functional programming. My work in that area has been focused into concurrency and especially *AKKA* interacting with the *Atmosphere Framework*. 

I've been using AKKA in its Scala version and Atmosphere is Java. Although I will love to write only Scala code, over and over again you need to interface with Java due to legacy code or because there aren't Scala libraries as mature as Java. That being the case of Atmosphere, per sites description 

> The Atmosphere Framework provides the enterprise features required to build massive scalable and real time asynchronous applications using transports like WebSocket, Server Side Events and traditional Ajax Techniques.
source: [async-io.org](http://async-io.org/)

Trying to keep this to the point, atmosphere communicates with subscribers using Broadcaster interface

    public interface Broadcaster {
    /**
     * Broadcast the {@link Object} to all suspended responses, eg. invoke {@link AtmosphereHandler#onStateChange}.
     *
     * @param o the {@link Object} to be broadcasted
     * @return a {@link Future} that can be used to synchronize using the {@link Future#get()}
     */
    Future<Object> broadcast(Object o);
    //more code omitted
    }

you can invoke broadcast and treat it as a fire and forget operation

    //Java code
    Broadcaster broadcaster = BroadcasterFactory.getDefault.lookup(target);
    broadcaster.broadcast(mymessage);

the broadcast method returns a future which makes it asynchronous, but what if you want to know if was able to broadcast the message? well, you could call get method on the future

    //Java code
    Broadcaster broadcaster = BroadcasterFactory.getDefault.lookup(target);
    Future<Object> future = broadcaster.broadcast(mymessage);
    future.get();

simple right? well, it just so happen that by invoking .get() you are blocking the thread and therefore you are no longer asynchronously processing your broadcasts. One nice way to do it is by wrapping the Java future with a Scala one like so:

        val broadcastFuture = future {
          val broadcast = x.broadcast(cmd)//this is the Java call
          broadcast.get()//this is blocking in some thread
        }
        broadcastFuture.onComplete {
          case Success(obj) =>
            log.debug(s"$cmd delivered to $target")
          case Failure(ex) =>
            log.error(s"$cmd was not delivered to target") //invoke here your fallback method
        }

is so simple and beautiful. I am so happy that all my concurrency knowledge comes from Scala. You see, although I remember writing a couple of synchronized methods in java I never did anything beyond homeworks and my ways to tackles this kind of problems are not polluted by the javaish way. Scala concurrency capabilities are so rich and simple. 

Probably next time I will show you how you can integrate Scala with Spring Framework. 