# Realtime Message REST API User Guide

## Request Format

For POST and PUT requests, the body of the requests should be JSON. The Content-Type needs to be configured as `application/json`.

## Authentication

Authentication of the request is via the key-value pair included in the HTTP Header. The parameters are as follows:

Key|Value|Meaning|Source
---|----|---|---
`X-LC-Id`|{{appid}}| App ID of the current application| Check in Dashboard -> Settings -> App keys
`X-LC-Key`|{{appkey}}|App Key of the current application | Check in Dashboard -> Settings -> App keys

Some administrative interfaces require master key.

## Relevant Concept

The `_Conversation` table includes some built-in fields to define the attributes and participants of the conversations. For one-on-one conversations, group chats, chat rooms, official accounts, and bots, see [LeanMessage Overview - Conversation](./realtime_v2.html#conversation) for more information.

## Architecture

For the concept of "Conversation", we have categorized it into three subclasses:

- For one-on-one conversations and group chats, the related API is labeled with `rtm/conversations`.
- For chat rooms, the related API is labeled with `rtm/chatrooms`. In the `_Conversation` table, `tr` is set to true to indicate this type of conversation.
- For official accounts and bots, the related API is labeled with `rtm/service-conversations`. In the `_Conversation` table, `sys` is set to true to indicate this.

Besides these, there are also client-related requests which are labeled with `rtm/clients`. 
Lastly, some [global APIs](#Global_API) are labelled with `rtm/{function}`, like `rtm/all-conversations` which will search for conversations of all types.
