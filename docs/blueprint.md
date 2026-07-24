# Private File Share Bot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A Telegram bot that allows the owner to upload files and generate shareable download links accessible only to members of a specified Telegram group or channel. Links remain valid until manually revoked by the owner.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Telegram group/channel owner
- group members

## Success criteria

- Owner can upload files and generate shareable links
- Group members can access files via links only if they are in the specified group/channel
- Owner can revoke links at any time

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu
- **/upload** (command, actor: user, command: /upload) — Initiate file upload process
- **Generate Share Link** (button, actor: user, callback: file:generate_link) — Create a shareable download link for a selected file
  - inputs: file_id
  - outputs: download_link
- **Revoke Link** (button, actor: user, callback: link:revoke) — Revoke a shareable download link
  - inputs: link_token
  - outputs: revocation_status

## Flows

### File Upload
_Trigger:_ /upload

1. User initiates upload
2. Bot requests file
3. User sends file
4. Bot stores metadata and content
5. Bot shows manage panel

_Data touched:_ File record

### Link Generation
_Trigger:_ file:generate_link

1. User selects file
2. Bot generates token
3. Bot posts link in group/channel
4. Bot logs link creation
5. Owner notified

_Data touched:_ Download link, Audit log

### Link Access
_Trigger:_ download_link_clicked

1. User clicks link
2. Bot verifies group/channel membership
3. Bot checks link validity
4. Bot serves file or denies access
5. Bot logs download event

_Data touched:_ Download link, Audit log

### Link Revocation
_Trigger:_ link:revoke

1. User selects link
2. Bot revokes link
3. Bot updates status
4. Owner notified
5. Bot logs revocation

_Data touched:_ Download link, Audit log

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **File record** _(retention: persistent)_ — Metadata and content for uploaded files
  - fields: filename, size, MIME/type, upload timestamp, owner, public link token, access scope (group/channel id), revoked flag
- **Download link** _(retention: persistent)_ — Shareable token and metadata for file access
  - fields: opaque token, associated file id, created timestamp, revoked status
- **Audit log** _(retention: persistent)_ — Record of all file and link activity
  - fields: event type, user id, timestamp, file id, link token

## Integrations

- **Telegram** (required) — Bot API messaging and file serving
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Upload files
- Generate share links
- Revoke links
- View audit logs

## Notifications

- Owner notified in private chat on uploads, link creations, revocations, and downloads

## Permissions & privacy

- Only group/channel members can access shared files
- File content is stored securely
- Audit logs retained until deleted by owner

## Edge cases

- User clicks revoked link
- User not in allowed group/channel attempts access
- Owner tries to revoke non-existent link

## Required tests

- Verify group/channel membership check works
- Test file upload and download flow
- Validate link revocation stops access

## Assumptions

- Owner is the only user who can upload files
- Single access scope (group/channel) is used for all files
- Links remain valid until manually revoked
