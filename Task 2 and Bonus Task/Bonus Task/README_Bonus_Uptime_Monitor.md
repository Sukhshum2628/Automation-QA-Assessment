# Bonus Task --- n8n Uptime Monitor

## Overview

This independent n8n workflow monitors the public RealWorld Conduit demo
application used during Task 1:

`https://demo.realworld.show/`

It checks the application every five minutes, records response
information, and sends a Discord notification when the endpoint does not
return HTTP 200. A separate daily schedule is configured to send an
uptime summary at 8:00 AM.

The monitored application is an external public demo, not an application
hosted or owned by this project.

## File

  -------------------------------------------------------------------------------------
  File                                              Description
  ------------------------------------------------- -----------------------------------
  `Bonus_UptimeMonitor_SukhshumVaishnavi(1).json`   Bonus uptime-monitor workflow

  -------------------------------------------------------------------------------------

## Workflow logic

### Five-minute monitoring path

1.  **Every 5 Minutes:** Schedule Trigger starts a check every five
    minutes.
2.  **Set: Target + Start Timer:** Provides the target URL and records
    the start timestamp.
3.  **Ping Target App:** Sends an HTTP GET request with an eight-second
    timeout. The request is configured to retry up to three times,
    waiting two seconds between attempts.
4.  **Code: Record Check + Response Time:** Calculates elapsed time,
    reads the HTTP status and determines whether the target is
    considered healthy.
5.  **IF: Status != 200?:** Checks the Boolean `up` value.
    -   If `up` is `true`, the healthy path runs through
        `No Action - App Healthy`.
    -   If `up` is `false`, the workflow sends a Discord downtime alert.

HTTP 200 is treated as healthy. A different status code or a request
that produces no response is treated as unavailable.

### Daily summary path

A separate Schedule Trigger is configured for 8:00 AM. It reads the
check history retained in n8n workflow static data, calculates the total
checks, successful checks, failed checks, uptime percentage, average
response time and maximum response time, and sends the summary to
Discord.

## Configuration

### Target URL

The `targetUrl` field in `Set: Target + Start Timer` is configured as:

`https://demo.realworld.show/`

If you monitor a different application, replace this value with a
publicly reachable URL that is expected to return HTTP 200.

### Discord credential

Select the configured Discord Webhook credential in both:

-   `Discord: Send Downtime Alert`
-   `Discord: Send Daily Summary`

Do not store webhook URLs or other secrets directly in this repository.

### Timezone

The workflow timezone should be set to `Asia/Kolkata` so that the daily
summary schedule is interpreted as 8:00 AM India Standard Time.

## Testing performed

### Unavailable-target test

An intentionally invalid target URL was used for a controlled test. The
workflow:

-   received no successful HTTP response;
-   marked `up` as `false`;
-   routed through the true output of `IF: Status != 200?`; and
-   sent a downtime notification to Discord.

### Healthy-target test

The target URL was restored to `https://demo.realworld.show/`. The
workflow successfully followed the healthy path:

-   the response was treated as HTTP 200;
-   `up` was `true`;
-   the IF node followed its false output; and
-   `No Action - App Healthy` executed without sending a downtime alert.

## Activation and operation

1.  Import the JSON file into n8n.
2.  Verify the target URL.
3.  Select the Discord webhook credential in both Discord nodes.
4.  Set the workflow timezone to `Asia/Kolkata`.
5.  Save the workflow.
6.  Test both the healthy and unavailable branches.
7.  Restore the real target URL after the unavailable-target test.
8.  Activate/publish the workflow.

The n8n instance and host machine must remain running for the scheduled
checks to execute. If n8n is running locally, stopping the n8n process
or shutting down the computer will stop monitoring.

## Important note about daily history

The daily summary uses `$getWorkflowStaticData('global')` to retain
check history. n8n static data is intended to persist through successful
production executions; manual editor executions should not be relied on
to populate persistent history. Verify the daily summary after the
activated workflow has accumulated production checks. For stronger
long-term history, a persistent data store such as an n8n Data Table or
external database would be preferable.

## Screenshots to include

Add genuine screenshots from your n8n instance:

-   Full uptime-monitor canvas showing both schedules.
-   Healthy execution showing the no-action branch.
-   Unavailable-target execution showing the downtime-alert branch.
-   Discord downtime notification.
-   Daily summary notification after production check history has
    accumulated.

## Limitations

-   The workflow monitors the configured public URL and does not
    establish an availability guarantee for the external service.
-   It treats HTTP 200 as the only healthy status. Change the condition
    if the monitored service legitimately uses another success status.
-   The daily summary depends on n8n static-data persistence and should
    be verified with production executions.
