# RaHa Studio - LinkedIn AEC Content Pipeline

An n8n workflow that watches AEC/BIM industry RSS feeds, picks the most-trending story from the last 48 hours, and turns it into a ready-to-post LinkedIn post - headline, caption, and a generated image - logged to a sheet and pushed to Telegram for review, entirely on free-tier tools.

## How it works

1. Runs every 2 days, builds a list of RSS feed sources and fetches them over HTTP
2. 2. Parses and normalizes the feed XML, filters to items published in the last 48 hours, and ranks them to pick the single most relevant trending topic
   3. 3. Builds a prompt from the chosen topic and sends it to an LLM via OpenRouter, requesting both post copy and an image-generation prompt back as JSON - configured with a model fallback chain (a free auto-router first, then two named free models) so a single throttled/removed model doesn't stall the run
      4. 4. Parses the LLM's JSON response and generates an accompanying image via Pollinations (a free image API)
         5. 5. Uploads the image to Google Drive and shares it (anyone-with-link) to get a stable public URL
            6. 6. Checks whether auto-posting is available; either way, composes the row data and logs the post text, hashtags, and image link to a Google Sheet
               7. 7. Sends the draft to Telegram - one message with the post text, one with the generated image - for review before publishing
                 
                  8. ## Tools / Technologies
                 
                  9. n8n, OpenRouter (free-tier LLM routing), Pollinations (free image generation), Google Drive, Google Sheets, Telegram Bot API
                 
                  10. ## Setup
                 
                  11. Credential references, both Google Sheet IDs, and the Drive folder are replaced with placeholders:
                 
                  12. 1. Import `workflow.json` into n8n
                      2. 2. Attach an HTTP Header Auth credential (OpenRouter API key as a Bearer header) to the Generate Post + Image Prompt node
                         3. 3. Attach Google Drive and Google Sheets credentials to their respective nodes
                            4. 4. Set your own feed-source sheet ID (read by Build Feed List) and your post-log sheet ID (written by Log to Google Sheet)
                               5. 5. Attach a Telegram credential to both Notify via Telegram nodes and point them at your review chat
                                  6. 6. Check the model list on the OpenRouter node against openrouter.ai/models before running - free-tier model availability shifts over time
                                    
                                     7. ## Key Responsibilities
                                    
                                     8. - Built the RSS ingestion and 48-hour recency filter that keeps the content pipeline current without manual topic research
                                        - - Designed the trending-topic ranking logic that picks a single story per run from multiple feeds
                                          - - Wired the OpenRouter call with an explicit model fallback chain to keep the pipeline running through free-tier throttling
                                            - - Built the review step (Sheet log + Telegram preview) so nothing posts without a human glance first
                                             
                                              - ## Key Achievements
                                             
                                              - - Runs the full pipeline - research, copy, image, review - on entirely free-tier infrastructure
                                                - - Produces a consistent stream of educational Revit/BIM content for RaHa Studio without manual drafting
                                                  - - Paired with a Sales Navigator outreach strategy targeting ADU specialists, solo architects, and general contractors across key US states
                                                    - 
