---
title: "More details for the places you'll go"
date: 2023-05-01T00:00:00-00:00
---

{{< rawhtml >}}
<span class="byline">posted by <a href="https://github.com/michaelkirk">Michael Kirk</a>, May 2023</span>
{{</ rawhtml >}}

As of [v0.5.1](https://github.com/headwaymaps/headway/releases/tag/v0.5.1), Headway now shows details like website, phone number, and opening hours for businesses and other places.

{{< figure src="museo.png" class="screenshot" alt="Screenshot of a map on maps.earth showing the Instituto Cultural Cabañas in Guadalajara Mexico, showing its website, phone number, and opening hours.">}}

## How does it work?

Headway uses the [Pelias](https://pelias.io) geocoder.

When you type ["Instituto Cultural Cabañas"](https://maps.earth/search/Instituto%20Cultural%20Cabañas) or ["123 Station Road"](https://maps.earth/search/123%20Station%20Road) into the search bar,
it's the geocoder that finds where that is on the map.
If instead of a name, you start with a location by clicking on the map,
Pelias can also tell you what is at that location in a process called _reverse geocoding_.
Pelias gets information about places from a few sources, but most relevant to us is the [OpenStreetMap](https://openstreetmap.org/about) community mapping project.

When you visit a Headway deployment like [maps.earth](https://maps.earth), data is not being served directly from OpenStreetMap.
A big part of the Headway project is taking data from sources like OSM, and transforming it into the various high performance formats needed for the services which make up a Headway deployment, like Pelias.
This data that Headway prepares includes map tiles for rendering, routing graphs for giving directions, and a database for our geocoding services.

{{<figure class="screenshot" src="osm-import-diagram.jpg" alt="A diagram showing data flowing from the OpenStreetMap.org database through the Headway build process to the maps.earth frontend">}}

In summary, data originating from OpenStreetMap is transformed by the Headway build process into a format suitable for our "Places" API, where it can be used by the Headway frontend application.

## But wait, these opening hours are wrong!

Although OpenStreetMap is known for having pretty good geometry data, my anecdotal experience with POI data like phone numbers, websites, and opening hours has been less consistent.

Phone numbers are actually relatively good.
Websites are more often missing or outdated.
Opening hours are rarely present and almost never up to date.
To be fair, even big consumer products, like Google Maps and Apple Maps, have a lot of outdated opening hours.

I want to reiterate that this has only been my anecdotal experience.
I don't want to diminish anyone's work, and I expect that quality varies by region.
In fact, if you know of areas that have particularly good POI data, [I'd love to know](mailto:info@maps.earth) where and how they did it!

My hunch is that geometric data is more consistently high quality because it's more visible, and it's more useful to a wider range of folks.
Compounding with that, geometry tends to be relatively unchanging.
Construction happens, as does erosion and sea level rise, all of which result in geometry changing,
but keeping up with the pace of changes to every local book shop's opening hours is a different kind of problem.

It's less about the one-time effort and more like maintaining a garden. You're never really done, but my hope is that exposing POI data in Headway will encourage people to do a little more gardening.

{{<figure class="two-up first" src="garden-weeds.jpg" alt="A garden patch overgrown with dandelions and scrubby grass">}}
{{<figure class="two-up" src="garden-mulched.jpg" alt="The same garden patch, with weeds removed. The soil has been freshly mulched over with straw.">}}

For now, "editing" in Headway simply links the user to editing the node on openstreetmap.org,
but there are some problems with this approach.

As a user, it can be a strange experience to be sent to some external site.
You might not be logged into or have an OpenStreetMap account.
You might not even know or care what OSM is.
A paragraph of explanatory text next to an outgoing link can only do so much to assuage that.

Secondly, because Headway isn't serving "live" OSM data, any edits a user makes on openstreetmap.org won't be visible until their Headway instance maintainer imports the changes into their next build.
Headway does a lot to make builds easy for maintainers, but it's still a lot of computation to run through the build process which outputs the map tiles, places, and routing that might be affected by the updated input.
In the case of a planet instance, like [maps.earth](https://maps.earth), a full build takes a couple of days to complete.
We do it about once a month.
That's a discouraging amount of time to have to wait before seeing your changes reflected.

## "Can't you just..."

Building a system to show edits in Headway sooner (even immediately) is possible, but it poses some interesting questions — both technical and social.

Microsoft's [Map builder](https://wiki.openstreetmap.org/wiki/Bing_Mapbuilder) is an interesting case study.
When a Microsoft user edits [Bing Maps](https://www.bing.com/maps) with Map builder, their edits are also (eventually) contributed to OpenStreetMap.
That sounds like a great way to funnel the labor of Microsoft's huge user base into OpenStreetMap "for free", right?

Unfortunately, the initial Map builder integration broke OpenStreetMap's systems for quality control and community moderation.
Consequently, Map builder was [temporarily blocked](https://www.openstreetmap.org/user_blocks/5701).
To their credit, Microsoft has done some work since then to better reconcile their requirements with those of OpenStreetMap.
Bing users can once again see their edits uploaded to OpenStreeMap, apparently in a way that the OSM moderators are OK with.
Still, some people have [lingering concerns](https://www.openstreetmap.org/user/Pieter%20Vander%20Vennet/diary/400909) about Map builder's longer term implications for the OpenStreetMap project.

I think it's a nuanced situation overall.
The different tradeoffs between the various needs of different groups are in tension.
I think that reasonable, even well intentioned, people might come to different conclusions.
For now anyway, Headway has punted and simply links to the openstreetmap.org editor,
but the future of editing right in Headway is to be determined.
We would appreciate [your input](https://github.com/headwaymaps/headway/discussions/278) on the matter!

{{<figure class="two-up first" src="garden-box.jpg" alt="The author and a friend bend over a raised garden bed, pulling weeds from the soil">}}
{{<figure class="two-up" src="garden-grown.jpg" alt="The same garden plot from earlier in the post, with pumpkin vines sprawling over the fence, corn, flowers, and other garden plants have grown tall. It's a bit ragged over all, but verdent.">}}

In the meanwhile, I hope you enjoy knowing a little more about the places around you. Happy gardening! 🌱

