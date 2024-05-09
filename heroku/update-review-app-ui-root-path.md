# Upgrade Review App Environment Variable (UI_ROOT_PATH)

```bash
#!/bin/bash

# Check if logged into the correct Heroku account
current_account=$(heroku auth:whoami)
expected_account="abid.abdullah@crowdbotics.com"

if [ "$current_account" != "$expected_account" ]; then
    echo "Please log into the correct Heroku account."
    heroku login
    exit 1
fi

# Prompt for the Heroku review app slug
read -p "Enter the Heroku review app slug: " review_app_slug

# Fetch the latest UI_ROOT_PATH environment variable from the staging app
staging_app_slug="crowdbotics-slack-dev"
ui_root_path=$(heroku config:get UI_ROOT_PATH -a $staging_app_slug)

# Update the UI_ROOT_PATH environment variable in the review app
heroku config:set UI_ROOT_PATH=$ui_root_path -a $review_app_slug
```