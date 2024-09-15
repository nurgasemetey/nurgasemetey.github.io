---
title: "My experience with data backfilling and achieving reproducibility"
date: 2024-09-15 00:00:00 +00:00
modified: 2024-09-15 00:00:00 +00:00
tags: [data-engineering]
---

Recently, I was busy with some data engineering related things and I want to share some insights.

# Make jobs resilient to failure

So you wrote following job that runs daily.

```sql
SELECT *
FROM item_events
WHERE item_events.item_id in (
    SELECT id
    FROM item
    WHERE item.status='ACTIVE'
) and item_events.date = {DATE}
```

and deployed to production using your workflow management tool(ex. Airflow). Some time later you notice that logic is wrong and have to you run job again for last month.

The thing is that can wrong is that you depend on `item.status`. Some items's statuses can have another value. Maybe month ago, they were `ACTIVE`.

There are ways to tackle this.

**Take a snapshot of tables**

You can take snapshots of `item` table for each day.

```sql
SELECT *
FROM item_events
WHERE item_events.item_id in (
    SELECT id
    FROM item_snapshot
    WHERE item.status='ACTIVE' and date_snapshot={DATE}
) and item_events.date = {DATE}
```
As you see, for that old date, item's `status` will still be `ACTIVE`. Thus, you can be sure that on the job rerun it will give same results.

**Introduce new values to make it job reproducible**

You can introduce new fields that will be valid throughout interval: `active_start_date` and `active_end_date`.

```sql
SELECT *
FROM item_events
WHERE item_events.item_id in (
    SELECT id
    FROM item
    WHERE item.active_start_date<={DATE} and item.active_end_date>={DATE}
) and item_events.date = {DATE}
```

On the rerun, it will still give correct results because we did stabilize it by introducing dates where it should be taken into account.