---
layout: post
title:  "Rails from first principles: Building a has_many bi-directional association"
date:   2020-08-12 18:30:00 +0100
categories: programming
tags: ruby rails web
---
A great way to learn is by trying to explain, so as part of my rails association learning process, I’ll attempt to explain a slightly more complex bi-directional association between two models. This is part of the “Building advanced forms” exercise of The Odin Project.

Objective: I want to build a proof of concept flight-booking app, and the first requirement is to create associations between an Airport model and a Flight model.

Steps:

1. Each airport will have many flights, and each flight will belong to an airport. Airports have 3-letter international codes, e.g. LIS for the Lisbon international airport.
2. Each flight, however, will need both origin and destination airports.
3. Each airport will have departing and arriving flights.

So what does this mean for our model associations?

#### Step 1:

![Step 1](https://miro.medium.com/max/1400/1*y9lnEwls9B0u73SmeiY_QQ.png)

The airport model has a Code variable, which is a string field, such as “SFO” or “LIS”. This association creates the `Airport.find(1).flight` and `Flight.find(1).airport` methods which let you query Airport flights and flight airports, respectively.

#### Step 2:

Now we need to separate flight airports into origin airports and destination airports, since flights fly from A to B. We will call these `from_airport` and `to_airport` to make the relationships easier to understand. The Flight associations are changed to:

{% highlight ruby %}
belongs_to :from_airport, class_name: “Airport”
belongs_to :to_airport, class_name: “Airport”
{% endhighlight %}

This means that each flight will have a from_airport and a to_airport, but that these are simply different instances of the Airport model. This means we have to create a `from_airport_id` field and a `to_airport_id` field in the Flight model in order to associate each Flight to two airports, the origin and destination ones.

![Step 2](https://miro.medium.com/max/1400/1*-k_eFSdzP0zjd_O-xtZsSw.png)

#### Step 3:

The next step is to add departures and arrivals to the airport model. A departing flight of an airport is a flight whose from_airport field corresponds to the airport id of that airport. An arriving flight is a flight whose to_airport field corresponds to the airport’s id.

So what we want to do is to call these flights `departing_flights` and `arriving_flights`, and associate them to a `from_airport_id` field and a `to_airport_id` field, in the Flight model. This association is done by setting a `foreign_key`.

{% highlight ruby %}
has_many :departing_flights, foreign_key: :from_airport_id, class_name: “Flight”
has_many :arriving_flights, foreign_key: :to_airport_id, class_name: “Flight”
{% endhighlight %}

![Step 3](https://miro.medium.com/max/1400/1*mA2rxjieQ7QUyY9EsASQUg.png)

In narrative form, this means that Airports have flights, and they are either departing or arriving depending depending on whether the `from_airport_id` or `to_airport_id` field corresponds to the airport ID.

Likewise, Flights have “from airports” (origins) and “to airports” (destinations), which is set by the ids in `from_airport_id` and `to_airport_id`.

We can now use methods such as `Airport.first.departing_flights` to get the departing flights of the first airport, and `Flight.first.from_airport` to get the origin of the first flight.

If we wanted to, we could change `from_airport` to `origin` and `to_airport` to `departure` but we’ll leave it as it is in the interest of simplicity. We could also add `inverse_of` to make database querying more efficient, but we’ll also leave that out as it’s not a strict requirement.

And that’s it. Our app now has Airports with many departing and arriving flights, as well as Flights with origins and destinations.

— — —

Hope this was insightful. Feedback is always welcome.