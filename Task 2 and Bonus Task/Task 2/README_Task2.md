# Task 2 --- GitHub Trending Digest Automation

## Overview

This n8n workflow retrieves recently created GitHub repositories tagged
with `machine-learning`, ranks them by stars, enriches the leading
repository with its README content, and sends a digest to a Discord
channel.

The main workflow supports two entry points:

-   **Schedule Trigger:** runs hourly (configured for testing).
-   **Webhook Trigger:** accepts a POST request for manual testing.

A separate Error Handler workflow is included for unhandled workflow
failures.

## Files

  ------------------------------------------------------------------------------------------
  File                                                   Description
  ------------------------------------------------------ -----------------------------------
  `Task2_Workflow_SukhshumVaishnavi(1).json`             Main GitHub Trending Digest
                                                         workflow

  `Task2_ErrorHandler_Workflow_SukhshumVaishnavi.json`   Separate workflow for unhandled
                                                         execution errors

  `Automation_QA_Assessment_Work_Explanation.docx`       Explanation of the workflow design
                                                         and testing

  `screenshots/`                                         Execution and configuration
                                                         evidence (add screenshots before
                                                         submission)
  ------------------------------------------------------------------------------------------

## Main workflow logic

1.  **Trigger:** Starts from the hourly schedule or the manual webhook.
2.  **Fetch repositories:** Calls the GitHub Search API for repositories
    with the `machine-learning` topic created within the previous seven
    days. Results are requested in descending star order.
3.  **Transform:** Extracts repository name, owner, URL, star count and
    description; sorts the results and keeps the top five.
4.  **Select and validate:** Selects the highest-ranked repository and
    checks that valid repository data exists before README enrichment.
5.  **README enrichment:** Fetches the selected repository's README. If
    this request fails, a fallback README message is used while
    retaining the repository metadata.
6.  **Conditional formatting:** Checks whether the selected repository
    has more than 1,000 stars and routes it to the hot-repository or
    normal-digest formatter.
7.  **Discord notification:** Sends the formatted digest to the
    configured Discord channel.

If the GitHub search fails or returns no matching repositories, the
workflow uses its no-data path and sends a Discord alert instead of
attempting README retrieval with missing repository information.

## Error handling

The separate `Morning Brief - Error Handler` workflow contains:

`Error Trigger → Discord: Send Workflow-Level Error Alert`

The Error Handler must be selected in the main workflow under **Settings
→ Error Workflow**. The Discord credential must also be selected in the
Error Handler's notification node.

The main workflow's no-data branch handles expected conditions such as a
failed repository search or no matching results. The separate Error
Trigger workflow is intended for unhandled execution failures; these are
different mechanisms.

## Credentials and configuration

### Discord

Create a Discord webhook and configure it as an n8n Discord Webhook
credential. Select that credential in:

-   `Discord: Post Digest`
-   `Discord: Send No-Data Alert`
-   `Discord: Send Workflow-Level Error Alert` in the Error Handler
    workflow

Do not commit the webhook URL, bot tokens, or other secrets to the
repository.

### GitHub

The HTTP Request nodes are configured for unauthenticated access to
GitHub's public API. GitHub's unauthenticated rate limit may apply. If
authentication is added later, configure it consistently on both GitHub
HTTP Request nodes.

### Workflow setup

1.  Import the main workflow JSON into n8n.
2.  Import the Error Handler JSON as a separate workflow.
3.  Configure the Discord webhook credential in the relevant nodes.
4.  Save the Error Handler workflow.
5.  In the main workflow, open **Settings → Error Workflow**, select
    `Morning Brief - Error Handler`, and save.
6.  Keep the main workflow inactive until manual testing and credential
    setup are complete.
7.  Activate/publish it when ready for scheduled production executions.

## Testing performed

The main workflow was manually tested using the Webhook Trigger's test
URL. The observed execution:

-   received the webhook POST request;
-   fetched GitHub repository results;
-   selected the top five repositories;
-   fetched the leading repository's README;
-   executed the applicable star-threshold formatting branch; and
-   delivered the digest to Discord.

The tested repository followed the normal-digest branch because its star
count was below the 1,000-star threshold.

The Error Handler was imported, configured and linked to the main
workflow. A production-triggered failure test should be performed and
captured before describing end-to-end Error Handler execution as
verified.

## Manual webhook test

1.  Open the Webhook Trigger node in n8n.
2.  Click **Listen for Test Event**.
3.  Copy the displayed Test URL.
4.  Send a POST request from PowerShell:

``` powershell
Invoke-RestMethod -Method POST `
  -Uri "PASTE_YOUR_N8N_TEST_WEBHOOK_URL_HERE" `
  -ContentType "application/json" `
  -Body "{}"
```

5.  Inspect the execution canvas and confirm that the digest arrives in
    Discord.

The test URL is temporary and is only available while the node is
listening. Use the production webhook URL only after activating the
workflow.

## Screenshots to include

Add genuine screenshots from your own n8n instance:

-   Full main workflow canvas.
-   Successful main workflow execution.
-   Discord digest notification.
-   Main workflow Settings showing the selected Error Workflow.
-   Error Handler canvas and, once tested, its production-triggered
    execution.

## Notes

-   The hourly schedule is currently configured for testing; adjust it
    if a different delivery cadence is required.
-   The workflow JSON does not contain your actual n8n credentials.
    Credentials must be configured in the n8n instance after import.
-   A successful manual webhook execution proves the tested path, but
    does not by itself prove that the scheduled production trigger or
    Error Trigger has been tested.
