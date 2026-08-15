# Telegram Equipment Photo Logger

An n8n workflow that turns a captioned photo sent to a Telegram channel into a fully filed, serial-numbered equipment record - no spreadsheet or Drive folder touched by hand. Handles both Electrical and Mechanical equipment through parallel branches with their own sheets and Drive folders.

## How it works

1. A channel post with a photo and a structured caption triggers the workflow (e.g. `Type: E, Panel Name: MCC-2, Equipment Name: Feeder Breaker, Location: Substation A`)
2. 2. The caption is parsed into fields; a `Type` of `E` (Electrical) or `M` (Mechanical) routes the item down the matching branch - Electrical requires Panel Name/Equipment Name/Location, Mechanical requires Equipment Name/Location/Details/Remark
   3. 3. The workflow reads the target sheet to work out the next serial number (max existing `Picture S/N` + 1)
      4. 4. Downloads the largest available resolution of the photo from Telegram, renames it `<serial>_<equipment name>.<ext>`, and uploads it to the matching Drive folder (separate folders for Electrical vs. Mechanical)
         5. 5. Shares the uploaded file (anyone-with-link) and builds the shareable URL
            6. 6. Appends a row to the matching Google Sheet tab with the equipment details and the Drive link
               7. 7. Replies in-channel confirming what was logged, or - if the caption doesn't parse - replies with the exact format expected
                 
                  8. ## Tools / Technologies
                 
                  9. n8n, Telegram Bot API, Google Sheets API, Google Drive API
                 
                  10. ## Setup
                 
                  11. This export has all credential references, the Sheet ID, and both Drive folder IDs replaced with placeholders:
                 
                  12. 1. Import `workflow.json` into n8n
                      2. 2. Attach credentials: a Telegram Bot credential on the Trigger and both Telegram/Download/Send nodes, a Google Sheets credential on the Get All Sheet Rows/Append Row to Sheet nodes (x2, one per branch), and a Google Drive credential on the Upload/Share nodes (x2)
                         3. 3. Set your Google Sheet ID on both branches' Sheets nodes, with two tabs matching each branch's columns (Electrical: Panel Name, Equipment Name, Location, Picture S/N, Picture Drive Link; Mechanical: Equipment Name, Location, Details, Picture S/N, Picture Drive Link, Remark)
                            4. 4. Set a Drive folder ID on each branch's Upload node (one for Electrical photos, one for Mechanical)
                               5. 5. Point the Telegram Trigger at your channel and post a test photo with a caption in the expected format
                                 
                                  6. ## Key Responsibilities
                                 
                                  7. - Designed the single-caption-format parser that extracts structured fields for two different equipment types from one free-text field
                                     - - Built the auto-incrementing serial number logic by reading the sheet's existing max value rather than maintaining a separate counter
                                       - - Set up the parallel Electrical/Mechanical branches so each type logs to its own sheet tab and Drive folder without duplicating the trigger or validation logic
                                         - - Added format-error handling that replies with the exact expected caption structure, rather than failing silently
                                          
                                           - ## Key Achievements
                                          
                                           - - Replaced fully manual equipment photo filing (rename, upload, share, log) with a single Telegram message
                                             - - Runs in production logging real equipment records with zero manual Drive/Sheets work
                                               - - One workflow now covers two equipment categories with independent numbering and destinations
                                                 - 
