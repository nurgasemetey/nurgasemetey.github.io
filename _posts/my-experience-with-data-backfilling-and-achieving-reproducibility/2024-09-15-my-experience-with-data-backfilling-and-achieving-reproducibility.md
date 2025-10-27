---
title: "My experience with data backfilling and achieving reproducibility"
date: 2024-09-15 00:00:00 +00:00
modified: 2024-09-15 00:00:00 +00:00
tags: [data-engineering]
---

Recently, I was busy with some data engineering related things and I want to share some insights.

# Make jobs resilient to failure

So you wrote following job that runs daily 

```sql
SELECT *
FROM item_events
WHERE item_events.item_id in (
    SELECT id
    FROM item
    WHERE item.status='ACTIVE'
) AND item_events.date = {DATE}
```

and deployed to production using your workflow management tool(ex. Airflow). Some time later you notice that logic is wrong and have to you run job again for last month.

The thing is that can go wrong is that you depend on `item.status`. Some items can have different statuses right now than month ago. On job rerun, events for items that were active in the previous month but are no longer active will not be saved.

You must remember that jobs will **always** fail for some reason. What is important is that how can you achieve correctness of backfilled data.

There are ways to solve this.

**Take a snapshot of tables**

You can take snapshots of `item` table for each day.

Example of snapshot data
```csv
id,name,status,snapshot_date
1,itemA,ACTIVE,2024-09-13
1,itemA,ACTIVE,2024-09-14
1,itemA,PASSIVE,2024-09-15
```

New  job
```sql
SELECT *
FROM item_events
WHERE item_events.item_id in (
    SELECT id
    FROM item_snapshot
    WHERE item.status='ACTIVE' AND snapshot_date={DATE}
) AND item_events.date = {DATE}
```
As you see, for that old date, item's `status` will still be `ACTIVE`. Thus, you can be sure that on the job rerun it will give same results.

**Introduce new values to make job reproducible**

You can introduce new fields that will be valid throughout interval: `active_start_date` and `active_end_date`.

```sql
SELECT *
FROM item_events
WHERE item_events.item_id in (
    SELECT id
    FROM item
    WHERE item.status='ACTIVE' OR (item.active_start_date<={DATE} AND item.active_end_date>={DATE})
) AND item_events.date = {DATE}
```

On the rerun, it will still give correct results because we did stabilize it by introducing dates where it should be taken into account.


Of course, you can develop custom scripts to backfill data. Downside of this is that your script can contain errors too. However, approaches above will allow to rerun job by only clicking to relevant days from your workflow management tool.