# Telegram Auto-Forward Bot

A powerful Node.js bot that automatically forwards messages from Telegram chats to your "Saved Messages". Perfect for archiving important conversations, monitoring groups, or backing up media from multiple sources.

## ✨ Features

### Core Capabilities
- **Automatic Forwarding**: Monitors chats 24/7 and instantly forwards messages to your Saved Messages
- **Smart Media Handling**: Preserves original quality by forwarding directly when possible
- **Bypass Restrictions**: Automatically re-uploads content when forwarding is restricted
- **Flexible Filtering**: Choose to monitor private chats, groups, or both
- **Content Type Control**: Forward only media, only text, or all messages
- **Source Whitelisting**: Optionally filter by specific users or groups
- **Context Preservation**: Adds sender and chat information to re-uploaded content
- **Session Persistence**: No need to re-authenticate on every run

### Advantages

✅ **Quality Preservation**: Direct forwarding maintains original media quality and formatting  
✅ **Intelligent Fallback**: Automatically detects and bypasses forward restrictions  
✅ **Proper Filenames**: Preserves original filenames for documents and files  
✅ **Anti-Loop Protection**: Built-in guards prevent infinite forwarding loops  
✅ **Minimal Duplicates**: Fixed logic prevents duplicate messages  
✅ **Efficient**: Uses Telegram's MTProto API for fast, reliable operation  
✅ **Customizable**: Easy configuration via environment variables  
✅ **No Database Required**: Stateless operation, runs anywhere Node.js works

## 📋 Requirements

- **Node.js**: Version 14 or higher
- **Telegram Account**: Active account with phone number
- **API Credentials**: Obtained from https://my.telegram.org/apps

## 🚀 Installation

### Step 1: Clone or Download the Repository

```bash
git clone <your-repository-url>
cd telegram-auto-forward-bot
```

### Step 2: Install Dependencies

```bash
npm install
```

Required packages:
- `telegram` - MTProto client library
- `dotenv` - Environment variable management
- `input` - CLI input handling

### Step 3: Get Telegram API Credentials

1. Visit https://my.telegram.org/apps
2. Log in with your phone number
3. Click "API development tools"
4. Fill in the form:
   - **App title**: Any name (e.g., "Auto Forward Bot")
   - **Short name**: Any short name
   - **Platform**: Desktop
   - **Description**: Optional
5. Click "Create application"
6. Save your `api_id` and `api_hash` (keep them secret!)

### Step 4: Configure Environment Variables

Create a `.env` file in the project root:

```env
# Required: Your Telegram API credentials
API_ID=12345678
API_HASH=abcdef1234567890abcdef1234567890

# Optional: Bot behavior configuration
MODE=private          # Options: "private" or "group"
FORWARD_TYPE=media    # Options: "media" or "all"
```

**Configuration Options:**

| Variable | Default | Description |
|----------|---------|-------------|
| `API_ID` | *(required)* | Your API ID from my.telegram.org |
| `API_HASH` | *(required)* | Your API Hash from my.telegram.org |
| `MODE` | `private` | `private` = monitor DMs only<br>`group` = monitor groups/channels only |
| `FORWARD_TYPE` | `media` | `media` = forward only media<br>`all` = forward media + text |

### Step 5: Set Up Source Filtering (Optional)

Open `index.js` and configure these arrays if you want to whitelist specific sources:

```javascript
// Leave empty to allow all sources (subject to MODE filter)
const ALLOWED_GROUPS = [];  // Add group usernames or IDs
const ALLOWED_USERS = [];   // Add user usernames or IDs

// Example with filters:
const ALLOWED_GROUPS = ['techgroup', '1234567890'];
const ALLOWED_USERS = ['john_doe', '9876543210'];
```

**Note**: If both arrays are empty, all sources matching the MODE filter will be monitored.

## 🎯 Usage

### First Run (Authentication)

```bash
node index.js
```

You'll be prompted to:
1. Enter your phone number (with country code, e.g., `+1234567890`)
2. Enter the verification code sent to your Telegram app
3. Enter your 2FA password (if enabled)

After successful authentication, a `session.txt` file will be created.

### Subsequent Runs

```bash
node index.js
```

The bot will automatically use the saved session and start monitoring immediately.

### Keeping the Bot Running 24/7

For production use, you need to run the bot in the background so it continues running even after you close the terminal.

#### Option 1: Using PM2 (Recommended for Production)

PM2 is a production-grade process manager for Node.js applications. It handles automatic restarts, logging, and ensures your bot runs continuously.

**Step 1: Install PM2 globally**
```bash
npm install -g pm2
```

**Step 2: Start the bot with PM2**
```bash
pm2 start index.js --name telegram-bot
```

**Step 3: Verify the bot is running**
```bash
pm2 status
# You should see telegram-bot with status "online"
```

**Step 4: Save the PM2 process list**
```bash
pm2 save
```
This ensures your bot restarts if PM2 restarts.

**Step 5: Enable auto-start on system boot**
```bash
pm2 startup
# Follow the command it outputs (copy-paste the entire line)
```

**Useful PM2 Commands:**
```bash
# View real-time logs
pm2 logs telegram-bot

# Stop the bot
pm2 stop telegram-bot

# Restart the bot
pm2 restart telegram-bot

# Remove the bot from PM2
pm2 delete telegram-bot

# Monitor CPU and memory usage
pm2 monit

# List all PM2 processes
pm2 list
```

**PM2 Log Management:**
```bash
# View logs with timestamps
pm2 logs telegram-bot --lines 100

# Clear old logs
pm2 flush

# View error logs only
pm2 logs telegram-bot --err
```

#### Option 2: Using screen (Linux)

Screen allows you to run processes in detached terminal sessions.

```bash
# Create a new screen session
screen -S telegram-bot

# Run the bot
node index.js

# Detach from screen: Press Ctrl+A, then D

# Reattach to screen later
screen -r telegram-bot

# List all screen sessions
screen -ls

# Kill a screen session
screen -X -S telegram-bot quit
```

#### Option 3: Using nohup

Simple background process (no process management).

```bash
# Start bot in background
nohup node index.js > bot.log 2>&1 &

# View logs
tail -f bot.log

# Find process ID
ps aux | grep "node index.js"

# Stop the bot
kill <process_id>
```

**Comparison:**

| Method | Auto-Restart | Logs | Monitoring | Difficulty |
|--------|--------------|------|------------|------------|
| PM2 | ✅ Yes | ✅ Built-in | ✅ Yes | Easy |
| screen | ❌ No | ❌ Manual | ❌ No | Medium |
| nohup | ❌ No | ⚠️ File only | ❌ No | Easy |

**Recommendation:** Use PM2 for production deployments. It's the most reliable and feature-rich option.

## 📊 How It Works

### Message Processing Flow

```
New Message Received
        ↓
   [Guard 1: Skip if from own Saved Messages]
        ↓
   [Guard 2: Skip if sent by self]
        ↓
   [Guard 3: Check MODE filter (private/group)]
        ↓
   [Guard 4: Check whitelist (if configured)]
        ↓
   [Check FORWARD_TYPE filter]
        ↓
   ┌─────────────────┐
   │  Has Media?     │
   └────┬──────┬─────┘
        │      │
       Yes     No
        │      │
        ↓      ↓
   Forward   Forward
    Media     Text
        │      │
        └──────┘
           ↓
    Saved Messages
```

### Media Forwarding Strategy

**Attempt 1: Direct Forward**
- Tries to forward the original message
- Preserves quality, format, and original captions
- ✅ Best option when possible

**Attempt 2: Re-upload (Fallback)**
- Downloads media to local storage
- Re-uploads to Saved Messages with context caption
- Cleans up temporary file
- 🔄 Used when forwarding is restricted

### Text Forwarding Strategy

**Attempt 1: Direct Forward**
- Forwards original message with formatting preserved
- ✅ Maintains links, bold, italic, etc.

**Attempt 2: Manual Re-send**
- Sends message content with sender/chat context
- 🔄 Used when forwarding is restricted

## ⚠️ Limitations

### Technical Limits

1. **Telegram API Rate Limits**
   - ~20 requests per second
   - Bulk operations may be throttled
   - Bot will automatically retry on rate limit errors

2. **File Size Limits**
   - Direct forward: Up to 2GB (Telegram limit)
   - Re-upload: Limited by disk space and memory
   - Very large files may cause timeouts

3. **Session Requirements**
   - Requires active Telegram account
   - Session expires if not used for ~30 days
   - Re-authentication needed after session expiration

### Functional Limitations

1. **No Backward Processing**
   - Only forwards new messages received after bot starts
   - Cannot retrieve/forward historical messages

2. **Storage Requirements**
   - Temporary media downloads require disk space
   - `./media` folder created for temp files
   - Files are auto-deleted after upload

3. **Network Dependency**
   - Requires stable internet connection
   - Will miss messages during downtime
   - No message queue or retry mechanism for missed messages

4. **Forward Restrictions**
   - Some channels/groups disable forwarding
   - Bot automatically switches to re-upload mode
   - Original sender information may be lost in re-upload

5. **Authentication**
   - Requires phone number access during setup
   - Cannot run on multiple instances with same account simultaneously

## 🔒 Security Considerations

### Best Practices

✅ **Keep credentials secret**: Never commit `.env` or `session.txt` to version control  
✅ **Use .gitignore**: Add these files:
```
.env
session.txt
media/
node_modules/
```

✅ **Secure API credentials**: Treat `API_ID` and `API_HASH` like passwords  
✅ **Session file security**: `session.txt` grants full account access  
✅ **Run in trusted environment**: Only run on secure, personal servers

### Warnings

⚠️ **Account Safety**: Unofficial clients may violate Telegram ToS  
⚠️ **Spam Risk**: Excessive forwarding may trigger spam detection  
⚠️ **Privacy**: Bot has full access to your account messages  
⚠️ **Shared Hosting**: Don't run on shared/untrusted servers

## 🛠️ Troubleshooting

### Common Issues

**Problem: "Error: Cannot find module 'telegram'"**
```bash
# Solution: Install dependencies
npm install
```

**Problem: "API_ID or API_HASH is undefined"**
```bash
# Solution: Check your .env file exists and is formatted correctly
cat .env  # Should show API_ID=... and API_HASH=...
```

**Problem: Session expired / Login required again**
```bash
# Solution: Delete old session and re-authenticate
rm session.txt
node index.js
```

**Problem: Bot forwards messages it just sent**
```bash
# Solution: Update to latest code version with anti-loop protection
# The fixed code ignores messages in your own Saved Messages chat
```

**Problem: Duplicate messages appearing**
```bash
# Solution: Ensure you're using the corrected forwardMedia function
# The issue was fixed by removing the separate caption message
```

**Problem: "MESSAGE_FORWARDS_RESTRICTED" for all messages**
```bash
# This is normal behavior - the bot automatically switches to re-upload mode
# Check that re-uploaded messages appear with context captions
```

### Debug Mode

Add console logs to track message processing:

```javascript
// Add after line: const chat = await message.getChat();
console.log('Processing message from:', sender?.username || sender?.firstName);
console.log('Chat:', chat?.title || chat?.username);
console.log('Message type:', message.media ? 'media' : 'text');
```

## 📝 Example Use Cases

### 1. Archive Important Conversations
```env
MODE=private
FORWARD_TYPE=all
```
Monitors all private messages and saves them to your archive.

### 2. Monitor Specific Groups Only
```javascript
// In index.js:
const ALLOWED_GROUPS = ['project_group', 'team_updates'];
```
```env
MODE=group
FORWARD_TYPE=all
```

### 3. Media Backup from Multiple Sources
```env
MODE=private
FORWARD_TYPE=media
```
Saves all photos, videos, and files from private chats.

### 4. Group Media Collector
```env
MODE=group
FORWARD_TYPE=media
```
Collects media from all groups you're in.

## 📚 Advanced Configuration

### Custom Filters

Edit the event handler in `index.js` to add custom logic:

```javascript
// Example: Only forward messages with specific keywords
if (message.message && !message.message.includes('important')) {
  return; // Skip messages without "important" keyword
}

// Example: Only forward large files
if (message.media?.document?.size < 1024 * 1024) {
  return; // Skip files smaller than 1MB
}
```

### Modify Caption Format

Edit the `buildCaption` function:

```javascript
function buildCaption(sender, chat, messageText = '') {
  const timestamp = new Date().toISOString();
  return `📩 Forwarded Message
From: ${sender?.firstName} (@${sender?.username})
Chat: ${chat?.title}
Time: ${timestamp}

${messageText}`;
}
```

## 🤝 Contributing

Contributions are welcome! Areas for improvement:
- Add database support for message logging
- Implement message queue for retry mechanism
- Add web dashboard for configuration
- Support multiple accounts
- Add message filtering by keywords/regex

## 📄 License

This project is provided as-is for personal purposes. Use responsibly and in accordance with Telegram's Terms of Service.

## ⚡ Quick Start Summary

```bash
# 1. Install
npm install

# 2. Configure
echo "API_ID=your_id_here
API_HASH=your_hash_here" > .env

# 3. Run
node index.js

# 4. Authenticate (first time only)
# Enter phone number, verification code, and password

# 5. Bot is now running!
# Press Ctrl+C to stop
```

---

**Need Help?** Check the troubleshooting section or open an issue on GitHub.

**Contact**: https://t.me/darkcoderhamster

**Disclaimer**: This bot uses Telegram's unofficial API. Use at your own risk and ensure compliance with Telegram's Terms of Service.
