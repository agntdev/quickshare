# FileLink Bot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that generates shareable download links for whitelisted users who upload files or paste URLs. Each link creation is reported to the owner's Telegram chat.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- paid customers with access to the whitelist

## Success criteria

- User receives a valid download link after upload or URL paste
- Owner receives a notification for every new link created

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Check whitelist status and show onboarding
- **/help** (command, actor: user, command: /help) — Show usage instructions and contact info
- **/mylinks** (command, actor: user, command: /mylinks) — List recent links created by the user
- **Upload file** (button, actor: user) — Upload a file to generate a download link
- **Paste URL** (button, actor: user) — Send a file URL to generate a download link

## Flows

### onboarding_flow
_Trigger:_ /start

1. Check user ID against whitelist
2. If not whitelisted: show access denied message
3. If whitelisted: show welcome and usage instructions

_Data touched:_ User

### upload_flow
_Trigger:_ file_upload

1. Receive file
2. Validate file size (max 100MB)
3. Store file metadata
4. Generate download link
5. Send link to user
6. Notify owner chat

_Data touched:_ File record, Shareable link

### url_flow
_Trigger:_ text_message

1. Validate URL protocol (http/https)
2. Attempt to fetch URL metadata
3. Store URL metadata
4. Generate download link
5. Send link to user
6. Notify owner chat

_Data touched:_ File record, Shareable link

### mylinks_flow
_Trigger:_ /mylinks

1. Retrieve user's recent file records
2. Format as list
3. Send to user

_Data touched:_ File record

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user identified by user ID for whitelist check
  - fields: telegram_user_id
- **File record** _(retention: persistent)_ — Metadata for uploaded or linked file
  - fields: original_name, size, mime_type, uploader_id, creation_timestamp, expiry, shareable_token
- **Shareable link** _(retention: persistent)_ — Stable URL or token that resolves to stored file
  - fields: token, url, file_record_id

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Manage whitelist of Telegram user IDs
- Configure owner notification chat ID

## Notifications

- Owner receives a message for every new upload or generated link with details

## Permissions & privacy

- Access restricted to whitelisted users only
- File metadata stored securely
- User IDs not shared with third parties

## Edge cases

- User sends unsupported file type
- User sends invalid URL
- User exceeds file size limit
- User is not whitelisted but tries to access features

## Required tests

- Verify whitelist check on /start
- Test file upload flow with valid and invalid files
- Test URL paste flow with valid and invalid URLs
- Verify owner notifications are sent for all link creations

## Assumptions

- Owner will manage whitelist externally
- File size limit is 100MB
- Links do not expire by default
