---
title: "How my lack of knowledge of mocking in Scala Spark applications was affecting code structure"
date: 2024-09-25 00:00:00 +00:00
modified: 2024-09-25 00:00:00 +00:00
tags: [spark, testing, scala]
---

I was working with Scala Spark applications recently. Because I didn't know much about mocking table reads in Spark and had to write a unit test for a function, I removed all the `spark.read` calls from the function. When I looked at the function to be tested, I noticed that it had become less cohesive.

Here is the version of the function without `spark.read` calls:

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

What I don't like about this declaration is that I have to provide `entityEvents` and `thirdPartyEvents` as arguments in order to test `processThirdPartyEvents`. I also don’t like that the caller needs to know about those events. Had I known more about mocking `spark.read`, I would have called `spark.read` directly inside `processThirdPartyEvents` instead of requiring those events as parameters. I don’t want to make this function public solely for testing purposes; I wanted to hide it and test it as a black box.

I much prefer this declaration because it hides which events are used, so the caller doesn’t need to be concerned with them:

```scala
def processThirdPartyEvents(entities: DataFrame, subEntities: DataFrame)
```

So my advice is: learn more about the tools you use!