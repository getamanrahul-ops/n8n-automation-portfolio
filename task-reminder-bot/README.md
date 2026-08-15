# Telegram Task Reminder Bot

A two-workflow n8n system that turns a Google Sheet into a full task-tracking bot: assigns tasks, pings people on a schedule until they're done, and handles replies - all through Telegram, with no webhook (the n8n instance isn't publicly reachable, so the bot polls instead).

## Why two workflows

- `bot-handler.json` - the interactive half. Polls Telegram's `getUpdates` every minute and handles everything a person does: `/task`, `/tasks`, `/mine`, `/start` to register, inline-button presses (mark done, snooze, reassign, request/approve extension), and photo-as-proof replies.
- - `scheduled-reminders.json` - the proactive half, on two independent schedules: a daily 10:00 run that pings anyone with a task due (day 0, then every N days per the task's interval) and expires tasks past their window, and a Saturday 11:00 run that posts a digest of everything still open.
 
  - Both read/write the same Google Sheet and share the same config shape, so they're meant to run together.
 
  - ## How a task flows
 
  - 1. An admin runs `/task Safety Shoes | @manum_arl | 60 | 7` in the group (name, assignee, duration in days, reminder interval in days - a 5th number makes it auto-repeat after completion)
    2. 2. The bot resolves the assignee (by reply, numeric ID, @username, or name), writes a row to the `Tasks` sheet, and DMs them with inline buttons
       3. 3. The scheduled workflow pings them every `interval` days until the `duration` window closes, posting to the group and DMing the assignee each time
          4. 4. They close it out with the done button or by replying to the reminder with a photo; either logs to a `History` tab and, if the task repeats, immediately reopens the next cycle
             5. 5. Extension requests route to whoever's listed in `ADMIN_IDS` for approval, capped at `MAX_EXTENSIONS`
               
                6. ## Tools / Technologies
               
                7. n8n, Telegram Bot API (long-polling, no webhook), Google Sheets API
               
                8. ## Setup
               
                9. Both exports have the bot token, Sheet ID, group chat ID, and admin user ID replaced with placeholders - they were hardcoded in a `Config` node rather than n8n's credential store, so replace them directly in that node:
               
                10. 1. Import both workflows into n8n
                    2. 2. In `bot-handler.json`'s P. Config node and `scheduled-reminders.json`'s A. Config and C. Config nodes, set: `BOT_TOKEN` (from @BotFather), `SHEET_ID`, `GROUP_CHAT_ID`, `ADMIN_IDS` (comma-separated Telegram user IDs), `TZ`, `MAX_EXTENSIONS`
                       3. 3. Attach a Google Sheets credential to every HTTP Request node currently pointing at `sheets.googleapis.com`
                          4. 4. Build the target Sheet with three tabs: `Tasks` (columns A to U, see `TASK_COLS` in the Code nodes for the exact header row), `People` (Name, Username, Telegram ID, Role), and `Log`/`History` for the audit trail
                             5. 5. Activate both workflows - the handler starts registering users the moment someone DMs the bot `/start`
                               
                                6. ## Key Responsibilities
                               
                                7. - Designed the Sheet-as-database schema (Tasks/People/Log/History) and the column layout the bot reads and writes
                                   - - Built the polling handler from scratch: command routing, inline-button callbacks, admin permission checks, and photo-as-proof matching (by reply, by task ref in the caption, or by process of elimination when only one task is open)
                                     - - Solved for the n8n instance not being publicly reachable by moving the bot from webhook to a `getUpdates` polling loop with offset tracking, so no update is ever double-processed
                                       - - Built the recurring-task logic - closing a repeating task immediately reopens the next cycle rather than waiting for a new assignment
                                        
                                         - ## Key Achievements
                                        
                                         - - Deployed and running in production, tracking real recurring compliance-style tasks (e.g. safety equipment checks) for the team
                                           - - Self-test workflow validates the full config (sheet, tabs, headers, token, group) before going live
                                             - - Extension requests, reassignment, and the Saturday digest all shipped as iterative additions on top of the original day-0/interval reminder loop
                                               - 
