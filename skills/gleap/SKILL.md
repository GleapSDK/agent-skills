---
name: gleap
description: Work with a Gleap workspace through the Gleap MCP server. Search and triage support tickets, read and answer customer conversations, check feature request demand, manage contacts, companies and CRM pipelines, maintain help center articles, analyze surveys, and pull support statistics. Use whenever "Gleap", a Gleap ticket, a Gleap dashboard link (app.gleap.io), or the user's Gleap inbox, feature requests, help center, surveys or support metrics come up.
---

# Gleap

[Gleap](https://www.gleap.ai) is a customer support and feedback platform: AI support, live chat, bug reporting, feature requests, roadmaps, help center and surveys. The `gleap` MCP server bundled with this plugin acts on one Gleap project, the one the user picked when they signed in.

## Before you act

1. **Never guess IDs.** Find tickets, contacts, companies, articles and workflows with the search and list tools first, then use the IDs they return.
2. **Read before you write.** Call `get_ticket` and `get_ticket_messages` before replying to, closing, merging or re-assigning a ticket.
3. **Ask before anything the customer will see.** `send_message` with `isNote: false` messages the customer directly. Only send when the user explicitly asks you to. If they want to review first, use `draft_reply_to_composer`, which stages the reply in the Gleap dashboard and sends nothing. Use `send_message` with `isNote: true` for internal notes.
4. **Confirm destructive or public actions.** Get an explicit yes before `close_ticket`, `archive_ticket`, `merge_tickets`, `delete_news`, `remove_pipeline_entry`, or any `publish_*` / `pause_*` tool. Tool annotations (`destructiveHint`) mark these.
5. **Link back.** Many results include a `dashboardUrl`. Share it so the user can open the record in Gleap.

## Choosing a tool

| You want to | Use |
|---|---|
| Find tickets by keyword | `search_tickets` |
| Filter tickets by status, assignee, priority, tags or date | `find_tickets` (set `includeArchived` for complete counts) |
| Read a ticket and its conversation | `get_ticket`, `get_ticket_messages`, `get_ticket_activity_log` |
| Find related or duplicate tickets | `search_similar_tickets`, then `link_tickets` or `merge_tickets` |
| Answer a customer | `draft_reply_to_composer` (review first) or `send_message` |
| Triage | `update_ticket`, `assign_ticket`, `add_ticket_tags`, `snooze_ticket`, `close_ticket`, `reopen_ticket` |
| Run an automation on a ticket | `get_workflows_available_for_ticket`, then `run_ticket_workflow` |
| Check feature request demand | `search_feature_requests`, `get_feature_request` |
| Look up a customer | `search_contacts`, `get_contact`, `get_contact_tickets`, `get_company` |
| CRM pipelines | `get_pipelines`, `get_pipeline_stats`, `get_pipeline_entries`, `move_pipeline_entry` |
| Answer from the knowledge base | `search_knowledgebase` |
| Help center | `get_help_center_collections_tree`, `get_help_center_article`, `create_help_center_article`, `update_help_center_article` |
| Surveys | `list_surveys`, `get_survey_results`, `get_survey_insights`, `search_survey_responses` |
| Support metrics | `get_statistics_facts` (response times, SLA, AI resolution), `get_statistics_list`, `get_statistics_bar_chart` |
| Team members | `get_teams`, `get_users` |

## Writing help center articles

Article content is TipTap JSON, localized per language. `update_help_center_article` only changes the locales you send and keeps the others. Read the article first and keep the existing structure.

## Not supported through this server

Billing, project settings, user and role management, and integrations configuration. Point the user to the Gleap dashboard at https://app.gleap.io for those.

## Setting up the Gleap SDK

If the user wants to add Gleap to an app rather than work with their workspace data, use the `gleap-sdk-setup` skill if it is installed (from [GleapSDK/agent-skills](https://github.com/GleapSDK/agent-skills)), or follow the docs at https://docs.gleap.ai.
