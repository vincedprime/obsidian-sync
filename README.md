# obsidian-sync
Sync your obsidian notes for free

1. Create a git repo. Clone in local. Go to obsidian and make that project folder path as you vault location.
2. Create a bash script
```bash
#!/bin/bash
# Setting explicit path as cron runs in a very minimal shell. It often doesn’t even know where git is.
export PATH="/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"

# Log file if needed
LOG_FILE="$HOME/obsidian-daily-sync.log"

log_message() {
    echo "$(date '+%Y-%m-%d %H:%M:%S'): $1" >> "$LOG_FILE"
}

REPO_PATH="$HOME/Documents/obsidian/brain-dump"  # REPLACE THIS WITH YOUR ACTUAL OF THE REPO

log_message "=== Starting git daily sync ==="

if [ ! -d "$REPO_PATH" ]; then
    log_message "ERROR: Repository directory does not exist: $REPO_PATH"
    exit 1
fi

cd "$REPO_PATH" || {
    log_message "ERROR: Failed to navigate to repository directory"
    exit 1
}

log_message "Successfully navigated to $REPO_PATH"

# Set git config as it may not have git config properly setup
git config user.email "your@mail.com"
git config user.name "your-username"

log_message "Git config set successfully"

log_message "Pulling latest changes from remote..."
if git pull --rebase origin main 2>&1 >> "$LOG_FILE"; then
    log_message "Successfully pulled latest changes"
else
    log_message "WARNING: Pull failed or had conflicts, attempting to continue..."
fi

git add . || {
    log_message "ERROR: Failed to add changes to git"
    exit 1
}

log_message "Changes added to git"

if git diff --cached --quiet; then
    log_message "INFO: No changes to commit"
    log_message "=== Sync completed (no changes) ==="
    exit 0
fi

# Create commit message per time
DATETIME=$(date '+%Y-%m-%d %H:%M:%S')
git commit -m "$DATETIME: dump" || {
    log_message "ERROR: Failed to create git commit"
    exit 1
}

log_message "Commit created successfully"

# Push to your branch - It main for this
if git push origin main 2>&1; then
    log_message "SUCCESS: Changes pushed to origin/main"
    log_message "=== Sync completed successfully ==="
else
    log_message "ERROR: Failed to push changes to origin/main"
    log_message "This could be due to network issues, authentication problems, or conflicts"
    exit 1
fi
```
3. Once done. Make this file executable
`chmod +x ~/obsidian-daily-sync.sh`
4. You can verify if this is woriking why direcrly executing this - `~/obsidian-daily-sync.sh`
5. Setup cron
`crontab -e` Once the file is opened add this
`0 12 * * * /Users/yourname/obsidian-daily-sync.sh` - This is cron so you can setup how frequently you want to push. This is everyday at 12pm.
Once done exit `:wq`

My sync - https://github.com/vincedprime/brain-dump
