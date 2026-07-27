# Channel Exporter Bot — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Exports Telegram channel messages/media to XLSX/CSV files for database import. Provides /list_exports, /download_export, and real-time message capture with resumable historical fetch.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Telegram channel owners
- database administrators

## Success criteria

- Owner can download complete XLSX/CSV exports containing all channel messages and media references

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Show main menu with status and help
- **/export_status** (command, actor: user, command: /export_status) — Show historical export progress and ETA
- **/list_exports** (command, actor: user, command: /list_exports) — List available export files with timestamps
- **/download_export** (command, actor: user, command: /download_export) — Download specific export file by ID
- **/force_export** (command, actor: user, command: /force_export) — Trigger immediate export of current data

## Flows

### Initial Setup
_Trigger:_ /start

1. Verify owner permissions
2. Begin historical message fetch

_Data touched:_ Message, Export

### Export Management
_Trigger:_ /force_export

1. Generate new export file
2. Store file with timestamp

_Data touched:_ Export

### Realtime Capture
_Trigger:_ new_channel_message

1. Store message metadata
2. Append to working dataset

_Data touched:_ Message

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Message** _(retention: persistent)_ — Stored message metadata and media references
  - fields: message_id, text, caption, media_type, media_file_id, timestamp
- **Export** _(retention: persistent)_ — Generated XLSX/CSV file metadata
  - fields: export_id, file_path, message_count, created_at

## Integrations

- **Telegram** (required) — Bot API messaging and channel access
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Add bot as channel admin
- Trigger exports via /force_export
- Download files via /download_export

## Permissions & privacy

- Only owner/admin can list/download exports
- Media stored as file_id/URL references only

## Edge cases

- Large media files exceeding Telegram API limits
- Duplicate messages during retry scenarios

## Required tests

- End-to-end export generation workflow with historical + real-time messages

## Assumptions

- Bot has full access to channel history
- Owner will handle media downloads via Telegram APIs
