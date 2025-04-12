---
title: "How my lack of knowledge of mocking in Scala Spark applications was affecting code structure"
date: 2024-09-25 00:00:00 +00:00
modified: 2024-09-25 00:00:00 +00:00
tags: [spark, testing, scala]
---

I was working with Scala Spark applications recently. Because I didn't know very much about mocking table reads in Spark and had to write unit test for function, I removed all `spark.read` calls to function. When I looked at function to be tested, I noticed that it was less cohesive. 

Here is the version function which didn't have `spark.read`. 

```scala
def processThirdPartyEvents(entities: DataFrame, subEntities: DataFrame, entityEvents: DataFrame, thirdPartyEvents: DataFrame)
```

And how it is used

```scala

val entityEventTable = applicationConfig.getString("entity_event_table")
val entityEvents = getEntityEvents(entityEventTable, jobConfig.year, jobConfig.month, jobConfig.day).cache()

val thirdPartyEventTable = applicationConfig.getString("third_party_event_table")
val startDate = LocalDate.of(jobConfig.year.toInt, jobConfig.month.toInt, jobConfig.day.toInt)
val endDate = startDate.plusDays(1)

val thirdPartyEvents = getThirdPartyEvents(thirdPartyEventTable, startDate, endDate)

processThirdPartyEvents(entities, subEntities, entityEvents, thirdPartyEvents)
```

What I don't like in such declaration is that I have to provide `entityEvents` and `thirdPartyEvents` to function declaration in order to test `processThirdPartyEvents`. Also, I don't like that place which it is calling knows about those events. If I had known more about mocking `spark.read`, I would call `spark.read` in the `processThirdPartyEvents` and didn't declare get those events from arguments. I don't want to make this function public so that it can be tested. I wanted to hide and test it as blackbox.


I like following declaration much better because it hides which events its using. And place where it is called would not know about those events.

```scala
def processThirdPartyEvents(entities: DataFrame, subEntities: DataFrame)
```

So my advice is to learn more about tools you use :).