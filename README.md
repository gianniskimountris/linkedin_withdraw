# LinkedIn Page Invitation Withdrawal — Cowork Skill

A Claude Cowork skill that automatically withdraws all sent LinkedIn **page** invitations older than 1 month.

## What It Does

- Navigates to `linkedin.com/mynetwork/invitation-manager/sent/ORGANIZATION/`
- Scrolls to load all sent page invitations (handles infinite scroll)
- Identifies every invitation sent **2+ months ago**
- Withdraws them one by one with modal confirmation
- **Keeps** all invitations sent within the last month (including "1 month ago")

## Installation

1. Download or clone this repository
2. Copy the `SKILL.md` file into your Cowork skills folder:
   ```
   %APPDATA%\Claude\local-agent-mode-sessions\skills-plugin\<session-id>\skills\linkedin-withdraw\SKILL.md
   ```
3. Restart Claude Cowork (or the skill will be picked up automatically)

## Usage

Just say any of the following in Claude Cowork:

- *"Withdraw my old LinkedIn page invitations"*
- *"Clean up my LinkedIn sent page invitations older than 1 month"*
- *"Remove all LinkedIn page invites older than a month"*

## Requirements

- **Claude Cowork** with Claude in Chrome extension installed and connected
- Active **LinkedIn** session in your browser
- The Chrome extension must have access to LinkedIn

## How It Works

LinkedIn's invitation manager uses React synthetic events, which means standard JavaScript `.click()` calls don't work. This skill uses actual simulated mouse clicks via the Claude in Chrome browser extension to:

1. Open the withdrawal modal for each old invitation
2. Click the confirmation button

The skill handles:
- Infinite scroll loading
- Multiple page invitations per person (each checked individually)
- Rate-limited `find` tool (falls back to coordinate-based clicking)
- LinkedIn's sticky header offset for accurate click positions

## Notes

- **"1 month ago"** is treated as recent and **kept**
- Only **"2 months ago"** and older are withdrawn
- LinkedIn enforces a 3-week cooldown before you can re-invite someone after withdrawing
- The skill works on the **ORGANIZATION** (pages) tab, not the People tab

## License

MIT — free to use, modify, and share.
