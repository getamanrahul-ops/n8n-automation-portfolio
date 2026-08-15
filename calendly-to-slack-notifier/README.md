# Calendly to Slack + Sheets Notifier

An n8n workflow that watches for new Calendly bookings and, within a minute of someone booking, posts a formatted Slack notification and logs the lead to a monthly Google Sheet tab - built around Calendly's Free plan, which has no webhook support.

## What it does

- Polls the Calendly API every minute for newly created bookings
- - Two independent guards prevent duplicate notifications: a 15-minute creation-time window, plus an n8n-native dedupe store keyed on the event URL (survives across runs/restarts)
  - - Pulls invitee details (name, email, timezone, Q&A answers, UTM source) and formats a rich Slack message - shows the time in both the invitee's and the host's timezone when they differ
    - - Posts to a configured Slack channel
      - - Appends the lead (name, email, meeting time) to a Google Sheet, filed under a tab named for the month it was booked in (e.g. `July-26`)
        - - A second scheduled trigger runs at 00:01 on the 1st of each month to pre-create that month's tab by duplicating a hidden `Template` tab - so the append step always has somewhere to write
         
          - ## Tools / Technologies
         
          - n8n, Calendly API, Slack API, Google Sheets API
         
          - ## Setup
         
          - This export has all credentials, IDs, and channel names replaced with placeholders. To run it yourself:
         
          - 1. Import `workflow.json` into n8n
            2. 2. Add three credentials and attach them to the matching nodes:
               3.    - Header Auth on `Get Calendly User`, `List Scheduled Events`, `Get Invitee Details` - a Calendly personal access token as a Bearer header
                     -    - Slack API on `Post to Slack`
                          -    - Google Sheets OAuth2 on `Get Sheet List`, `Duplicate Template`, `Append Lead`
                               - 3. Replace `YOUR_SPREADSHEET_ID` (appears in three nodes) with your actual Sheet ID, and set the Slack channel name on `Post to Slack`
                                 4. 4. In your target spreadsheet, create a hidden tab named `Template` with your desired headers/formatting - new monthly tabs are cloned from it
                                    5. 5. Set `HOST_TZ` at the top of the three Code nodes to your own timezone (defaults to `America/Denver` in this export)
                                      
                                       6. ## Key Responsibilities
                                      
                                       7. - Evaluated Google Apps Script and n8n approaches for catching new Calendly bookings, and designed around the Free-plan webhook limitation
                                          - - Built the polling + dedupe logic so the same booking can never notify twice
                                            - - Wrote the timezone-aware Slack message formatter (Luxon), including host/invitee time comparison
                                              - - Designed the auto-provisioning logic for monthly Google Sheet tabs, cloned from a template
                                               
                                                - ## Key Achievements
                                               
                                                - - Delivered automated Slack alerts for every new Calendly booking despite no webhook access
                                                  - - Removed manual lead logging entirely - every booking lands in the correctly dated Sheet tab with no monthly upkeep
                                                    - - Two-layer dedupe guard has run in production with zero duplicate notifications
                                                      - 
