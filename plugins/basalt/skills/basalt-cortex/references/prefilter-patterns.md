# Pre-filter Patterns

Applied before any AI call. Goal: skip 60-80% of content at near-zero cost.

## Step 1: Idempotency Check (always first)

Check if this item has already been processed:

```python
import json
state = json.load(open(f"{CORTEX_DIR}/state.json"))
if source_id in state.get("processed_source_ids", []):
    skip  # already processed
```

For file-based sources, check if a Basalt file already exists with matching `source_id` in frontmatter.

## Step 2: Automated Sender Patterns (Gmail/Chat)

Skip if `from` address matches (case-insensitive):

```
no-reply@, noreply@, do-not-reply@, donotreply@
notifications@, notification@, alerts@, alert@
mailer-daemon@, postmaster@, bounce@
auto-confirm@, automated@, robot@, bot@
newsletter@, newsletters@, marketing@, offers@, promotions@
billing@, invoices@, receipts@, system@
```

Skip if sender domain is a known automated source:
```
xero.com, myob.com, quickbooks.com
stripe.com, paypal.com, square.com
github.com, gitlab.com, bitbucket.org
atlassian.com, jira.com, trello.com
slack.com, notion.so, asana.com
google.com (calendar invites, drive shares)
cloudflare.com, aws.amazon.com
linkedin.com, facebook.com, twitter.com
mailchimp.com, sendinblue.com, constantcontact.com
```

## Step 3: Content Length Filter

Skip if body is too short to contain useful knowledge:
- Email body < 50 characters after stripping HTML → skip
- Chat message < 20 characters → skip
- Web page < 200 characters → skip (likely JS-rendered blank)

## Step 4: Subject Line Patterns (Gmail)

Skip if subject matches:
```
/^(re:\s*)*out of office/i
/^(re:\s*)*automatic reply/i
/unsubscribe/i (in body, not subject)
/^your .* receipt/i
/^your .* order/i (unless from a real person)
/^payment .* received/i
/^invoice .* from/i
```

## Step 5: Bot Detection (Chat/Slack)

Skip messages from:
- Users with `[BOT]` or `[APP]` in their display name
- Slack app messages (check `subtype: bot_message`)
- Google Chat app messages (check `sender.type: BOT`)
- Webhook-generated messages

## Step 6: Duplicate Content Detection

For communications that are part of a thread:
- Only process the thread once (use thread ID, not individual message IDs)
- For Gmail: the thread ID groups all replies together
- For Slack: use `thread_ts` to group threaded replies
- For Chat: use the space + thread key

## Keep Rules (override skips)

Always process if:
- From/to a known client domain (check existing `clients/` folder)
- Contains the owner's email in To/CC
- Has attachments (may contain contracts, quotes, briefs)
- Subject contains: quote, proposal, contract, agreement, invoice (from real senders)
