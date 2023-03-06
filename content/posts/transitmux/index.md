---
title: "Adding transit to maps.earth"
date: 2023-03-06T00:00:00-00:00
---

You can use Headway to host a map area of any size — from your local town up to something larger like a province or even the entire planet.
[maps.earth](https://maps.earth) is our planet scale demonstration of Headway.
Until recently, Headway couldn't support public transit routing on a map of this scale at all.

Now, if you're in Seattle or Los Angeles, you can try out [public transit directions on maps.earth](https://maps.earth/directions/transit/openstreetmap%3Avenue%3Away%2F221703439/openstreetmap%3Avenue%3Away%2F226019357).
Transit regions are still very limited, but I wanted to write up why that is, how we got here, and what might be next as we continue to expand what's practical.

## Giving directions is hard

It's easiest to start by talking about non-transit directions, which have been supported planet-wide on maps.earth for a while now.

There are lots of ways to get from A to B, but you want to avoid walking along major highways and you hopefully can't drive down a bicycle-only trail.
Headway uses [Valhalla](https://github.com/valhalla/valhalla) to optimize this kind of route planning.

More than just which roads go where, Valhalla also utilizes some of the rich "tag" data from [OpenStreetMap](https://openstreetmap.org), like speed limits and bicycle-only routes, to decide which route is best.

Valhalla's routing is pretty sophisticated already, and there are tons of opportunities for future work to make it choose safer, faster, and more pleasant routes — especially for pedestrians and cyclists.

Valhalla is also great for its performance characteristics. Thanks in part to its tile based approach, even a relatively modest machine can quickly serve up routes for the entire planet.

{{< figure src=allrode-to-rome.png class="screenshot" alt="screenshot of a Headway map framing most of central Europe that shows several alternatives for driving directions from Allrode, Germany to Rome, Italy">}}

Valhalla has to sift through a huge network of potential paths and consider each segment based on mode, speed, and other factors.
Doing this in a way that can scale across continents adds further complexity.
It's not an easy task, but at its core, it's solving a classic graph traversal problem, for which there are well understood, fast, solutions.
Let's compare that to planning a trip using public transportation, like a bus or train.

## Transit directions are harder

With private transportation, you can start your trip whenever you want.
Public transportation operates on a schedule and a fixed route.
An optimal trip schedule might involve transfers at fixed points.

With transit, you face decisions like "Should I just wait here, or should I walk two blocks farther to catch a different bus that leaves 10 minutes sooner?"
That's a question you don't need to consider when walking, biking, or driving.

If you're driving across Los Angeles to the Santa Monica pier, you might take [the](https://la.curbed.com/2017/9/6/16264074/socal-los-angeles-the-freeways) [101](https://en.wikipedia.org/wiki/U.S._Route_101_in_California) to the [110](https://en.wikipedia.org/wiki/Interstate_110_and_State_Route_110_(California)) to the [10](https://en.wikipedia.org/wiki/Interstate_10_in_California)
— I affectionately call this _Doing the [186](http://google.com/search?q=186%20in%20binary)_ 🤓.
You don't have to coordinate your transfer from one freeway to the next, you just get on.
You may get stuck in traffic for a while, but the sooner you get in, the sooner you get out.

With transit, the picture gets more complicated. You have to consider when and where the stops are, and the ramifications on which transfers are compatible between multiple legs.

{{< figure src=bus-to-santa-monica.png class="screenshot" alt="screenshot of Headway map framing Santa Monica to Los Angeles, showing transit directions from Los ANgeles City College to Santa Monica Pier. Several alternative trip options are shown." >}}

There are some quite different options for transit trips. Do we care more about arrival time, travel duration, or minimizing transfers? It depends!

(Personally, I'd probably take the 4 bus all the way, even if it's a little slower, because transfers introduce uncertainty and interrupt my reading. Reasonable people might disagree!)

Because of the cascading effects of each step in a transit route on future routing decisions, it becomes very difficult to implement fast transit routing without more specialized algorithms.

## Headway: strategic laziness

Headway is, at its core, some UI on top of a bunch of map related software written by other people, stitched together in an opinionated way.

In the same way we use Valhalla for our private transportation routing, [OpenTripPlanner](https://www.opentripplanner.org) is responsible for transit routing in Headway.
OpenTripPlanner's transit routing, based on the [RAPTOR](https://www.microsoft.com/en-us/research/wp-content/uploads/2012/01/raptor_alenex.pdf) algorithm, gives decent results, but is expensive in terms of memory and cpu.
OTP isn't designed to host the entire planet. Even a moderately sized country like Germany takes ~20GB of RAM, which is already out of reach for a lot of hosters. Compare that to Valhalla, which takes less than 1/2 that for the entire planet.
That's why, historically, transit was only supportable on regional deployments of Headway, like [seattle.maps.earth](https://seattle.maps.earth) and not our planet instance at [maps.earth](https://maps.earth).

Requiring an entire Headway deployment for every transit region is kind of annoying, even for me living in Seattle. For one, I'm not always in Seattle. But also, I'd like people all over the planet to be able to try transit on maps.earth.

In the interest of making some kind of progress on this problem, I realized that most often when I'm looking for transit directions, it's a trip within a single metro area.
It may not always be the same metro area from day to day, but I'm less frequently traveling from one city to another than I am within a single city.
So, for me anyway, there's value in launching independent transit zones, even if we can't yet route between them.

So for now, we simply piece-meal together separate OTP instances, for each supported transit region and our new service routes transit requests to the appropriate back end based on a spatial comparison.

Right now [maps.earth](https://maps.earth) only supports transit in [Seattle](https://github.com/headwaymaps/headway/blob/v0.4.5/k8s/configs/planet/opentripplanner-seattle-deployment.yaml) and [Los Angeles](https://github.com/headwaymaps/headway/blob/v0.4.5/k8s/configs/planet/opentripplanner-losangeles-deployment.yaml). This is for the embarrassingly simply reason that those are the two places I've been recently enough to test it.
Other Headway deployments can add as many (or as few) transit regions as they'd like to support.

There are definitely some fun computational problems to address if we were to add a lot of new transit zones, however there are some perhaps less fun problems more immediately.

## Fun problems

The current approach is very simplistic and has some obvious limitations.

We don't handle overlapping zones, so it's ambiguous if your trip is covered by more than one zone.
Finding a way to stitch together trips across zones could be an interesting challenge.
In particular, things like long distance trains between metro regions would be a good thing to support.
Another challenge might be megalopolises where transit regions sort of meld into each other, like in the northeastern US.

Maybe the most obvious problem of all, with these two zones, maps.earth only covers about 0.1% of global transit users.

# Less fun problems

That brings me to the less fun problems. How do we get more coverage? We could continue to just add more zones, but due to the hungry nature of the underlying data structures, we'll quickly run into the limits of our computing budget.

We work to make Headway practical to host on a modest machine. I think we've had some success for people who want to host something small, like their own city. Unfortunately, hosting a larger region, or a planet scale instance, is still necessarily pretty heavy. 

Transit in particular is likely to be out of reach for most beyond a regional deployment or two.

If you'd like to see maps.earth add transit support for a new area, and would be willing to help test it out, please [contact us](info@maps.earth) or [file an issue](https://github.com/headwaymaps/headway/issues/new).
Realistically though, we'll only be able to add a couple small areas without some recurring donations to help pay our server bills. It might be interesting to see how far we can get with this approach.

## What's next for public transit in Headway?

The introduction of the [transitmux service](https://github.com/headwaymaps/headway/tree/v0.4.5/services/transitmux) to proxy to multiple transit regions is one of the more recent improvements we've made to Headway's transit functionality, but it follows the original OpenTripPlanner integration that happened last year, and a long tail of UI work and other improvements.

Transit schedules change, and thus the transit feeds need to be regularly reimported.
With only a couple transit zones, it's easy to rationalize kicking off build processes by hand, but as coverage increases, making this automatic will be more important.

I'd love to add support for [GTFS-RT](https://developers.google.com/transit/gtfs-realtime/guides/feed-entities) which would allow leveraging the real time updates some transit agencies publish with more precise arrival times and other deviations from the base GTFS feed we currently use.

Last, but definitely not least, retrieving trip plans can be quite slow right now, especially with a larger transit graph like LA's.
I haven't yet spent enough time diagnosing the issue to decide if there's some configuration problem, if there's some worthwhile optimizations that could be done within OTP, or maybe there's a need for an alternate implementation.

I try to use Headway for my own getting around — I don't always succeed. Previously, one of the most common reasons I'd have for using some other app was a lack of transit directions on maps.earth.
A couple weeks in, and I now find myself using Headway for a lot more of my own travel needs.
So, that's some kind of progress, but like many types of progress, it's clarified how there is so much more work to do.
