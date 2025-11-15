# Facebook Messenger Attendance Bot

## Overview
This project is a Facebook Messenger bot designed to automate group chat attendance tracking, member management, and enforce group rules. It simplifies daily attendance recording, customizes greetings, and implements a robust moderation system with advanced vulgar word detection, spam protection, and a progressive ban system. The bot ensures persistent data storage and per-group chat administrative control, aiming to provide a comprehensive solution for managing active Facebook Messenger groups. Key capabilities include authenticating with Facebook, tracking group members, managing daily check-ins, and a sophisticated moderation system.

## User Preferences
I prefer iterative development, where features are implemented and tested in small, manageable steps. I value clear and concise explanations, avoiding overly technical jargon unless absolutely necessary. I want the agent to prioritize robust error handling and security in its implementations. When making significant changes or architectural decisions, please ask for my approval first. I prefer the use of modern JavaScript features and clean, readable code. I also expect the agent to be proactive in identifying potential issues or improvements.

## System Architecture
The bot is built on **Node.js 20** and utilizes the `fca-unofficial` library for Facebook Messenger API interaction. Data persistence is managed entirely through **JSON files**, ensuring data survives bot restarts.

**UI/UX Decisions:**
- **Command-line Interface:** All interactions are command-based within the Messenger chat.
- **Paginated Help:** The help menu is paginated (5 commands per page).
- **Role-based Command Access:** Commands are filtered based on user roles (all users, admins, super admin, developer).
- **Clear Status Indicators:** Uses emojis for attendance and missed attendance.
- **Varied Bot Responses:** Employs multiple responses for invalid commands.

**Technical Implementations & Feature Specifications:**
- **Authentication:** Uses `appstate.json` for Facebook session cookies.
- **Attendance System:**
    - Daily check-in via `.present` command (Philippines timezone UTC+8).
    - Automatic member detection and live synchronization with group membership changes.
    - Tracks consecutive absences and auto-kicks after 3 days.
    - Admins and Super Admins are exempt from attendance tracking.
    - Manual and targeted absence resets, temporary exclusion mechanisms.
    - Uses group chat nicknames; prompts for nicknames if absent.
- **Moderation System:**
    - **Advanced Vulgar Word Detection:** Comprehensive detection with Unicode font normalization, triple-pass normalization, aggressive character replacement, de-duplication, special character handling, hyphenated variations, and split message detection (last 5 messages within 30s).
    - **Bot Mention Protection:** Kicks users who mention the bot's user ID inappropriately.
    - **Startup Scan:** Scans last 50 messages for missed vulgar words while offline.
    - **Warning System:** Auto-warning on vulgar words (all users, including regular admins).
    - **War Extreme Mode:** Temporarily disables warning detection.
    - **Spam Detection:** Auto-kick for 7 same messages or 7 invalid commands within 10 seconds.
    - **Unsent Message Spam Protection:** Kicks users who unsend 5+ messages within 60 seconds; implements progressive ban after 3 consecutive kicks.
    - **Progressive Ban System:** 1st ban (3 days), 2nd ban (7 days), 3rd+ ban (permanent); ban records are retained.
    - **Auto-lift Expired Bans:** Checks every minute.
    - **Manual Ban/Unban:** Admins can ban/unban users with unique Ban IDs.
    - **Admin Protection:** Admins cannot be banned directly.
    - **Super Admin Nuclear Option:** `.banall` command for mass banning.
    - **Permanent Warnings:** Developer and Super Admin can issue permanent warnings to admins.
    - **Pending Member Approval:** `.pendinglist`, `.pendingaccept`, `.pendingreject` commands for managing new members.
- **Message Management:**
    - **Message Caching:** Caches messages and attachments for 1 minute for unsent message recovery, including pictures.
    - **Instant Unsent Message Recovery:** Reveals deleted messages immediately.
- **Admin System:**
    - **Per-Group-Chat Admin System:** Admin privileges are managed per group and persist.
    - Dynamic management allowing admins to add/remove other admins within their group.
    - Global admin cleanup: Only DEVELOPER_ID (100092567839096) remains as global admin.
    - Auto-admin group management: Bot removes all group admin privileges except for Developer, Super Admin, and the Bot itself when added to a new group.
    - Admin command restrictions: `.unwarning`, `.addmin`, `.removeadmin` require higher privileges.
- **Notifications:**
    - **New User Alerts:** Super admin receives private messages with new user details (name, UID, account creation date, gender, profile URL).
- **Customization:**
    - Customizable group greetings.
    - Configurable auto-warning keywords.
    - Storage and display of server IP:port information.
- **Dual Protected User System:** DEVELOPER (100092567839096) and SUPER ADMIN (61561144200531) with identical exemptions from attendance, vulgarity detection, warnings, spam, and bans.

**System Design Choices:**
- **Persistent Storage:** All dynamic data is stored in JSON files.
- **Modular Command Handling:** Commands are structured for easy expansion and role-based access control.
- **Timezone Awareness:** Aligned with the Philippines timezone (UTC+8).
- **No Command Cooldown:** Commands execute instantly.

## External Dependencies
- **Facebook Messenger API:** `fca-unofficial` library.
- **JSON Files:** For data persistence.

## Recent Changes

### November 15, 2025 - Enhanced Vulgar Word Detection System
**Major Improvement:** Completely overhauled the obfuscation detection system to be both more aggressive at catching obfuscated words AND more false-positive proof

**What Changed:**
- **Obfuscated Word Detection:** Now specifically detects and flags words with special characters/numbers (e.g., "@$$", "f**k", "d1ck")
- **Minimum Length Check:** Words must meet minimum length requirements before being considered matches (prevents "di" → "dick" false positives)
- **Expanded Safe Word List:** Added 40+ safe words including: class, glass, pass, mass, grass, traps, wraps, claps, laps, caps, gaps, maps, taps, zaps, naps, raps, slaps, snaps, flaps, straps, di, dia, dip, dig, dim, din, dine, die, did, and all their variations
- **Smart Word Extraction:** Extracts both clean words and words with special characters for separate processing
- **Context-Aware Matching:** Checks if original words contain obfuscation indicators (special chars/numbers) to distinguish intentional obfuscation from false positives

**Technical Details:**
1. New Functions:
   - `isObfuscatedWord(word)` - Detects if a word contains obfuscation (special chars/numbers)
   - `extractAllWords(message)` - Extracts both clean and original words for comprehensive checking
   
2. Enhanced Detection Logic:
   - First pass: Check for explicitly obfuscated words (with special characters/numbers)
   - Second pass: Check normalized text with minimum length validation
   - Third pass: Verify original words are not in safe word list
   - Fourth pass: Ensure word length matches expected keyword length
   
3. Improved False Positive Prevention:
   - Word length must match or exceed keyword length after normalization
   - Safe words are checked at multiple stages
   - Space-bypass detection only triggers if actual vulgar word found
   - Obfuscation ratio calculation (40%+ special chars/numbers = obfuscated)

**Fixed Issues:**
- ✅ "@$$" is now correctly detected as "ass" (obfuscated word detection)
- ✅ "Bro HE is falling for it I have a 5 traps still un..." no longer triggers false positive ("traps" added to safe words)
- ✅ "Di" no longer triggers false positive for "dick" (added to safe words + minimum length check)

### November 13, 2025 - Pending Member Auto-Import Feature
**Enhancement:** Added automatic scanning and importing of existing pending members from Facebook approval queues

**What Changed:**
- **Auto-scan on Startup:** Bot now scans all groups for existing pending members 15 seconds after startup
- **Auto-scan on Group Join:** When bot is added to a new group, it automatically scans for pending members
- **Smart Detection:** Handles both `pendingRequestIDs` and `pendingParticipants` API formats
- **Duplicate Prevention:** Skips members already in the pending list
- **Detailed Logging:** Provides clear console output showing import progress

**Technical Details:**
1. New Functions:
   - `scanAndImportPendingMembers(threadID)` - Scans single group for pending members
   - `scanAllGroupsForPendingMembers()` - Scans all groups the bot is in

2. Data Structure Updates:
   - `addPendingMember()` now maintains both legacy fields (`name`, `timestamp`) and new fields (`nickname`, `addedDate`) for full backwards compatibility

3. Integration Points:
   - Startup: Runs 15 seconds after bot initialization
   - New Group: Runs 12 seconds after bot is added to a group (after admin scanning)

**How It Works:**
- Checks if group has approval mode enabled (`threadInfo.approvalMode`)
- Retrieves pending member IDs from Facebook API
- Fetches user info for each pending member
- Adds them to the tracking system if not already present
- Logs detailed information about import progress

**Example Console Output:**
```
🔍 Starting scan for existing pending members across all groups...
📊 Found 3 thread(s) to scan
🔍 Scanning for existing pending members in thread 123456...
📋 Using pendingRequestIDs: 2 member(s)
📋 Found 2 existing pending member(s) in thread 123456
✅ Imported pending member: John Doe (100012345)
✅ Imported pending member: Jane Smith (100067890)
✅ Imported 2 pending member(s) from thread 123456
✅ Pending member scan complete! Total imported: 2
```

### Replit Setup - November 15, 2025

### Environment Setup
- **Node.js Version:** v20.19.3 (confirmed working)
- **Package Manager:** npm (all dependencies installed successfully)
- **Workflow Configuration:** Bot runs continuously via `node bot.js` command
- **Output Type:** Console (backend-only application, no frontend)

### Files Added/Modified
1. **`.gitignore`** - Created to protect sensitive data:
   - Excludes `appstate.json` (Facebook session cookies)
   - Excludes `node_modules/`, data files, and temporary files
   - Protects environment variables and logs

2. **Workflow Setup** - Configured bot to run automatically:
   - Command: `node bot.js`
   - Status: Ready (requires appstate.json to start)
   - Logs available in console output

### Required User Action
To run the bot, you must create an `appstate.json` file with your Facebook session cookies. See the detailed guides:
- **Quick Method:** Follow `HOW_TO_GET_APPSTATE.md` for browser cookie extraction
- **Alternative:** Use `login-with-credentials.js` for credential-based login

### Project Status
✅ Dependencies installed (`@dongdev/fca-unofficial` and dependencies)
✅ Workflow configured and ready
✅ Git protection for sensitive files in place
⚠️ **Action Required:** Create `appstate.json` to start the bot

The bot will start automatically once `appstate.json` is provided.