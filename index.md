---
title: "How to Find 30-Minute Appointment Slots in Google Calendar"
description: "Find appointment times that fit around existing Google Calendar bookings, with a runnable JavaScript example for 15-, 30-, and 60-minute appointments."
lang: en-AU
---

# How to Find 30-Minute Appointment Slots in Google Calendar

A caller asks for a 30-minute appointment at 9:30 am. The calendar is free at 9:30, but the next booking starts at 9:45. There’s room for a 15-minute call. A 30-minute appointment would overlap by 15 minutes.

To find times you can offer a caller, check the whole appointment against your opening hours and every busy period on the calendar.

## Try it with a sample calendar

Imagine a Brisbane business that takes appointments between 9:00 and 11:00 am. It offers start times every 15 minutes and already has a booking from 9:45 to 10:15. Save the example as `booking-slots.js` and run `node booking-slots.js`. The example uses fictional data and runs without a Google account.

```js
const minute = 60_000;

const opening = Date.parse("2026-11-25T09:00:00+10:00");
const closing = Date.parse("2026-11-25T11:00:00+10:00");
const busy = [
  {
    start: Date.parse("2026-11-25T09:45:00+10:00"),
    end: Date.parse("2026-11-25T10:15:00+10:00"),
  },
];

function availableStarts(durationMinutes) {
  const duration = durationMinutes * minute;
  const starts = [];

  for (let start = opening; start + duration <= closing; start += 15 * minute) {
    const end = start + duration;
    const fits = busy.every(block => end <= block.start || start >= block.end);
    if (fits) starts.push(start);
  }

  return starts;
}

const clock = new Intl.DateTimeFormat("en-AU", {
  timeZone: "Australia/Brisbane",
  hour: "2-digit",
  minute: "2-digit",
  hourCycle: "h23",
});

for (const duration of [15, 30, 60]) {
  const times = availableStarts(duration).map(start => clock.format(start));
  console.log(`${duration} min: ${times.join(", ") || "no slots"}`);
}
```

Expected output:

```text
15 min: 09:00, 09:15, 09:30, 10:15, 10:30, 10:45
30 min: 09:00, 09:15, 10:15, 10:30
60 min: no slots
```

A 30-minute appointment at 9:15 fits because it ends exactly at 9:45. The next available start is 10:15, when the existing booking ends. The morning has 90 free minutes in total, split into two 45-minute gaps, so a one-hour appointment cannot fit.

## Use real Google Calendar availability

1. Set the opening and closing times for the requested date in the business’s time zone. Use a named zone such as `Australia/Sydney` when converting local hours to timestamps so the offset follows daylight saving.
2. Query Google Calendar’s FreeBusy endpoint for that time window. Check the calendar’s `errors` field before using the response, then convert each `busy` period’s `start` and `end` timestamps with `Date.parse`. Use those periods in place of the sample `busy` array. [Google Calendar FreeBusy reference](https://developers.google.com/workspace/calendar/api/v3/reference/freebusy/query)
3. Recheck the selected time just before creating the event. Availability can change while the caller is deciding.

Secretary Builder connects to Google Calendar so its AI receptionist can check availability and book appointments during a phone call. [See how Secretary Builder handles appointment booking](https://secretarybuilder.com.au/features/appointment-booking)
