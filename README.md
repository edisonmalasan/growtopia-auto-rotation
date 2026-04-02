# Growtopia Auto Rotation Farm

## Description
This is an advanced Auto Rotation farming script for Growtopia, written in Lua. It is designed to fully automate the farming process—including harvesting, breaking, planting, storing seeds, purchasing packs with gems, and storing those packs—across an entire list of rotation worlds. The script also includes extensive logging via Discord webhooks, proxy support, and anti-nuke mechanisms.

## Key Features
- **Multi-Bot Setup:** Allows configuring logic and worlds for multiple bots using a centralized `Bot` table, identifiable by the bot's uppercase username.
- **Discord Webhooks:** Comprehensive real-time updates on bot status, current world, uptime, profit, dropped items, pack information, and captcha notifications.
- **Auto Pack Purchasing & Storing:** Automatically purchases packs when a specific gem minimum is met, and drops them in designated storage worlds.
- **Seed Management:** Detects seed shortages, retrieves seeds from a seed storage world, plants them, and drops surplus seeds back.
- **Farming Toggles:** Highly customizable harvesting features including rooting, separate planting phases, ignoring planting, blacklist coordinates, and more.
- **Proxy Support:** Includes logic for automatic proxy integration to avoid IP-based bans.

---

## Configuration Guide

### 1. Bot Profile Setup
Each bot must be configured within the `Bot` table at the top of the script `Bot["BOT_NAME_UPPERCASE"]`. Make sure your key exactly matches your bot's name in uppercase.

```lua
Bot["YOUR_BOT_USERNAME"] = {
    slot = 1, -- Custom slot assigned to the bot for the webhook logging
    password = "yourpassword", -- Bot Password
    proxyIp = "ip:port:user:pass", -- Proxy format (leave empty if not using proxy)
    
    webhookLink = "YOUR_WEBHOOK_URL", -- General bot info webhook URL
    messageId = "YOUR_MESSAGE_ID", -- Message ID to edit (Patch) for the main webhook
    
    worldList = {"WORLD1", "WORLD2", "WORLD3"}, -- Ordered list of worlds to farm
    doorFarm = "DOORID", -- Door ID used in the rotation worlds
    
    webhookLinkPack = "YOUR_PACK_WEBHOOK_URL", -- Webhook URL for dropped pack info
    messageIdPack = "YOUR_PACK_MESSAGE_ID", -- Message ID to edit for pack info
    
    webhookLinkSeed = "YOUR_SEED_WEBHOOK_URL", -- Webhook URL for dropped seed info
    messageIdSeed = "YOUR_SEED_MESSAGE_ID", -- Message ID to edit for seed info
    
    storagePack = "PACKSTORAGE_WORLD", -- World Name for storing packs
    doorPack = "PACKDOORID", -- Door ID in the pack storage world
    
    storageSeed = "SEEDSTORAGE_WORLD", -- World Name for storing extra seeds
    doorSeed = "SEEDDOORID" -- Door ID in the seed storage world
}
```

### 2. General Settings
The global variables allow you to fine-tune the script's behavior to match your exact farming setup.

**Core Webhook Configuration**
*   `webhookOffline`: URL for triggering notifications when the bot disconnects or gets a captcha (uses POST request).

**Block & Item Logic**
*   `detectFarmable`: `true` automatically detects the most abundant block in the world. If `false`, define using `itmId` and `itmSeed` below bypasses the check.
*   `itmId` & `itmSeed`: Define the Item and Seed ID manually (e.g., Pepper Tree = `4584`).
*   `patokanSeed` & `patokanPack`: The background/foreground ID used in your storage worlds as anchor blocks around which to drop seeds and packs.
*   `blacklistTile`: Define coordinates to `blacklist` in the table (relative to the white door) if you want the bot to ignore specific tiles.

**Timing & Delays**
*   `delayHarvest`: Delay for punching a tree.
*   `delayPlant`: Delay before placing a seed over an empty tree slot.
*   `delayPunch`: Delay when breaking a block during the PNB (punch-and-break) phase.
*   `delayPlace`: Delay for placing a block during the PNB phase.

**Farming Phase Controls**
*   `separatePlant`: Set `true` if you want the bot to do a separate pass over the whole world specifically for planting seeds.
*   `dontPlant`: Set `true` if you want the bot to solely harvest/break and store seeds, entirely skipping planting.
*   `root`: Set `true` if farming rooted items (e.g., items requiring an extra punch to fully clear a root after breaking a block).
*   `takePick`: Auto retrieve and wear a pickaxe from storage before initiating the rotation loop.

**Breaking Control (PNB)**
*   `tileNumber`: Set amount of blocks the bot places in one stack before punching them (up to `5`).
*   `customTile`: Set `true` if you want the bot to use a designated coordinates `customX` and `customY` for breaking operations in every world instead of a dynamically found spot.

**Loop & Reconnect Features**
*   `looping`: Iterate endlessly across the `worldList`. Once it finishes the last world, it will start back at the first.
*   `proxy`: Must be manually `true` to use the defined proxy from the Bot profile.
*   `restartTimer`: Set `true` to clear the elapsed timing variables inside the webhook at the start of every farm cycle/loop.
*   `customShow`: Reduces webhook bloat by only showing `showList` amount of worlds remaining.

**Pack Settings**
*   `packName`: Internal store name of the pack to purchase (e.g., `"world_lock_10_pack"`).
*   `packList`: Array containing IDs of items given by the pack. (`242` for WL/World Lock).
*   `packPrice`: Price of the pack in gems.
*   `minimumGem`: The threshold condition to trigger pack buying during the rotation (e.g., `20000`).
*   `packLimit`: How many packs to buy max per visit (avoids exceeding backpack slots).
*   `buyAfterPNB`: Set `true` to buy packs between world transitions whenever `minimumGem` requirement is met.

### 3. Extra World Transition Logic
*   `joinWorldAfterStore`: Set `true` to randomize joining a list of fallback worlds `worldToJoin` after visiting storage. Helps reset or obscure the IP traffic loop.
*   `joinDelay`: Configurable idle delay in external worlds before coming back to the farming rotation.

---

## Technical Notes & Limitations

*   **Discord Webhook Modifying:** This script relies exclusively on `Invoke-RestMethod` executing Windows PowerShell (`powershell -command -`) for its webhooks. It modifies an existing message using `PATCH` calls. **A preset message ID is required to use the webhook.**
*   The script uses Windows APIs to communicate with webhooks. Ensure PowerShell execution has adequate permissions if not using an in-game executer that inherently resolves it.
*   **Safety/Reconnection:** Implements robust anti-captcha timeouts and `nuked` detection (world un-accessible or forcefully banned from world).


## Disclaimer

Note that botting scripts violate Growtopia's Terms of Service and might result in a ban. Use at your own risk.