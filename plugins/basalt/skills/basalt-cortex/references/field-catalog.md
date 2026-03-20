# Basalt Field Catalog

Discovered from 80+ real email threads across 2015-2026 by Gemini 3.1 Pro. This catalog is the **springboard** for per-thread extraction — the AI selects relevant fields from this list AND can add new ones it discovers.

## How to use this catalog

When extracting metadata from an email thread or chat conversation:
1. Read the thread content
2. Scan this catalog for fields that match what's in the conversation
3. Add only the fields that are **evidenced by the content** — never add empty fields
4. If you discover something new that isn't in this catalog, add it anyway and note it as a new field
5. The universal fields (type, channel, subject, date, direction, participants, summary, tags) are always present

---

## Universal Fields (always present)

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Deterministic ID: `comm_{unix_ts}_{4char_slug}` |
| `type` | string | Always `communication` |
| `channel` | string | `email`, `chat`, `sms`, `call`, `meeting`, `note` |
| `subject` | string | Thread subject line |
| `date` | string | ISO 8601 with timezone |
| `direction` | string | `inbound`, `outbound`, `bidirectional` |
| `participants` | array | List of `{name, email}` objects |
| `message_count` | number | Messages in thread |
| `summary` | string | 1-3 sentences, dense with names. Used for semantic search |
| `tags` | array | Keyword tags for filtering |
| `source` | string | `gmail`, `google_chat`, `slack`, etc. |
| `source_id` | string | Thread ID for idempotency |

---

## Freeform Semantic Fields (add when present in content)

### Intent & Sentiment (any industry)

| Field | Type | When to add | Examples |
|-------|------|------------|---------|
| `primary_intent` | string | Always useful | `support_request`, `sales_inquiry`, `billing_update`, `project_feedback`, `scheduling`, `referral`, `personal`, `legal_notice`, `admin`, `newsletter_response` |
| `urgency_indicator` | string | When urgency is expressed | `critical_outage`, `date_specific_deadline`, `standard`, `low_priority` |
| `client_sentiment` | string | When tone is notable | `frustrated`, `apologetic`, `appreciative`, `neutral`, `disputed`, `enthusiastic` |
| `internal_delegation` | array of objects | When tasks are handed off | `[{assignor: "Natalie", assignee: "Ravelle", task: "follow up with Nick"}]` |

### Client Lifecycle & Relationships

| Field | Type | When to add | Examples |
|-------|------|------------|---------|
| `client_lifecycle_stage` | string | When stage is clear from context | `prospect`, `active_onboarding`, `active_project`, `active_maintenance`, `churning`, `churned` |
| `churn_reason` | string | When client is leaving | "Sold the business", "Migrating to Shopify", "In liquidation" |
| `referral_chain` | object | When someone introduces someone | `{referrer: "Chris Burgess", referee: "Alexander Dark", context: "BNI referral"}` |
| `multi_entity_portfolio` | array | When one person runs multiple businesses | `["Pumping Records", "Pumping Management", "Pumping Entertainment"]` |
| `business_relationship` | string | When the relationship type is notable | `client`, `partner`, `vendor`, `affiliate`, `referrer`, `prospect`, `former_client` |

### Project & Technical

| Field | Type | When to add | Examples |
|-------|------|------------|---------|
| `project` | string | When thread is about a specific project | `micro-skin-website`, `acps-launch`, `vsb-chatbot` |
| `project_blocker` | string | When something is blocking progress | "No payment facility - Eway", "Client can't provide access", "Designer reviewing" |
| `technical_assets` | array | When tech infrastructure is discussed | `["WordPress", "forms plugin"]`, `["Cloudflare DNS"]`, `["Stripe"]` |
| `third_party_platforms` | array | When external platforms are mentioned | `["Shopify"]`, `["Commerce7"]`, `["Eway"]`, `["Rezdy"]` |
| `infrastructure_action` | string | When a technical operation is happening | `provision`, `migrate`, `update`, `debug`, `teardown` |
| `design_changes` | array | When visual changes are requested | `["darker grey", "white outline adjustment"]` |
| `product` | string | When a specific product is being discussed | `L2Chat`, `Flare AI`, `JezPress` |

### Financial & Commercial

| Field | Type | When to add | Examples |
|-------|------|------------|---------|
| `financial_document` | string | When money docs are referenced | `remittance_advice`, `purchase_order`, `invoice`, `quote` |
| `amount` | string | When dollar figures are mentioned | "$500", "$770/month", "PO 4400426649" |
| `pricing_model` | string | When pricing structure is discussed | "Affiliate commission", "Subscription renewal", "One-off project" |

### People & Identity

These fields appear on **communication** notes when identity details are discovered in a thread. They should ALSO trigger an update to the corresponding **contact** file's `identifiers` block — see basalt-format.md for the contact identity schema.

| Field | Type | When to add | Examples |
|-------|------|------------|---------|
| `real_name` | string | When display name differs from real name | "George Petridis" (display: "George Fuel") |
| `also_known_as` | string | When aliases/brand names exist | "George Fuel", "Nova Pitstop" |
| `phone` | string | When phone numbers appear in content | "02 4954 6828", "+61 414 232 794" |
| `companies` | array | When multiple companies are involved | `["Verge Safety Barriers", "Uni-Fit", "Adenium"]` |
| `role_title` | string | When job title is mentioned | "Events & Placement Coordinator", "Director" |
| `google_chat_id` | string | When a Google Chat user ID is present | "users/105776807049217487682" |
| `additional_emails` | array | When signature reveals extra emails | `["info@pumpingrecords.com"]` |
| `website` | string | When a personal/business URL is shared | "www.novapitstop.com.au/book-online" |

### Community & Networking

| Field | Type | When to add | Examples |
|-------|------|------------|---------|
| `community_affiliation` | string | When networking/community groups are involved | "BNI Sunrisers", "Maryland Shortland Scouts" |
| `mass_broadcast` | string | When email is one-to-many | `security_alert`, `newsletter`, `event_invite` |

### Content-Specific (emergent)

| Field | Type | When to add | Examples |
|-------|------|------------|---------|
| `seo_rankings` | array of objects | When search rankings are discussed | `[{term: "Heritage Architect Newcastle", position: 4}]` |
| `newsletter_edition` | string | When responding to a newsletter | "Google already answered the question" |
| `issue` | string | When a bug/problem is reported | "Wrong phone number in chatbot", "Forms not working" |
| `bug_report` | boolean | When it's a product bug report | `true` |
| `meeting_time` | string | When a meeting is scheduled | "1pm", "Wednesday 10am" |
| `meeting_location` | string | When a meeting place is specified | "Jeremy's office", "Zoom" |
| `deadline` | string | When a date constraint exists | "Wednesday night launch", "COB Friday 6th March" |
| `decision` | string | When something was decided | "Launch rescheduled", "Landing page approved" |
| `action` | string | When someone committed to doing something | "Stefan calling Julie this arvo" |
| `website` | string | When a URL is shared | "www.novapitstop.com.au/book-online" |
| `domain` | string | When a domain name is the topic | "peece.com.au", "bluewaterelectricalservices.com.au" |
| `ai_context` | string | When AI is part of the discussion | `client_deployment`, `product_demo`, `product_ideation` |
| `user_testing_queries` | array | When someone reports search/AI test results | `["Heritage Architect Newcastle", "recliner repairs near me"]` |
| `key_quote` | string | When a quote captures the essence | "Did you just bang that out in a casual afternoon?" |

---

## Adding New Fields

This catalog is a living document. When extracting from a new thread, if you encounter metadata that doesn't fit any existing field:

1. Add it with a descriptive `lowercase_snake_case` name
2. Use the simplest appropriate type (string > array > object)
3. The field will naturally join the catalog as more threads are processed

Examples of fields that might emerge in other industries:
- A tradie: `site_address`, `materials_list`, `safety_requirements`
- A lawyer: `case_number`, `court_date`, `opposing_party`, `statute_reference`
- A recruiter: `candidate_name`, `role_applied`, `salary_range`, `interview_stage`
- A property manager: `property_address`, `tenant_name`, `lease_expiry`, `maintenance_type`
