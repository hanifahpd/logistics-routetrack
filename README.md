Route delay cost: definition sheet
Question: What does a late delivery cost, per route, per day?

Grain: One row per route per date. Key: route_id + date. This is the grain the collector script stores. If you price at stop level, say so here and name the stop table.

Data path:
- MySQL gives the route list: route, vehicle, driver, helper flag, warehouse, start date.
- A Python script logs in to the Wialon API and matches each route's vehicle to a Wialon unit by brand prefix plus registration number.
- For each route date, it requests that day's trips and raw GPS messages (00:00:00 to 23:59:59).
- Start and end time come from the trips. If there are none, it falls back to the first and last GPS message.
- Results are appended to a CSV and deduplicated on route_id + date, keeping the newest row. Wialon deletes history after 90 days, so this CSV is the long-term record.
- SQL joins the CSV to the planned delivery slots. Power BI reads the joined result.

Late stop
A route-day is late when its actual end time is more than minutes after the planned end of the delivery window. The collector doesn't calculate this. SQL does it downstream. If you measured lateness per stop instead of per route, rewrite this as arrival at the stop versus the slot, and name the table that holds arrivals.

Cost per stop
Minutes late × [driver cost per minute], times driver_amount (driver plus helper count), plus [rebooking fee] if the delivery was rebooked. Route cost is the sum for that day. Delete any part you didn't actually use. If you only ranked routes by minutes late and never priced them, write "ranked by delay minutes" and drop the cost claim from the CV.

Excluded;
- Routes with no matching Wialon unit (flagged Unit Not Found).
- Routes with no GPS activity (flagged No GPS Activity).
- Rows with a date error or a Wialon error.
- Cancelled routes and [test routes].
- Excluded rows stay in the table with their data_source flag. Reports filter them out, but they remain countable, so you can say how many routes had no usable GPS data.

Refresh;
[Daily at every hour starts from 6:00 to 23:00 via Task Scheduler/cron], covering the previous day.
The script has no scheduler built in, so keep this line only if you did set one up. Re-runs overwrite on route_id + date. Guard against overwriting a good row with an empty one on dates older than 90 days.

Accuracy check:
Pick trucks on days you remember. Stored start and end times match what Wialon showed before expiry.
A route everyone remembers as late is flagged late. A route everyone remembers as on time is not.
Count the share of rows flagged Unit Not Found or No GPS Activity. Report it, along with the registration-matching rule that caused the misses.

The late-route count agrees with the operations manager's own tally for one week.

Owner: me. Reviewer: [operations manager's name or role], who agreed the late rule and the cost rate.
