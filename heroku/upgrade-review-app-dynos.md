# Upgrade Review App Dynos based on Production Dynos

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

# Fetch dyno config from production and scale review app accordingly
production_app_slug="crowdbotics-slack"

production_dyno_config=$(heroku ps --json -a $production_app_slug)

heroku ps:type standard-1x -a $review_app_slug

# Use jq to parse the JSON and scale the review app
echo "$production_dyno_config" | jq -r 'group_by(.type) | .[] | {type: .[0].type, count: length, size: .[0].size} | "\(.type)=\(.count):\(.size)"' | while read scale_command; do
    echo "Scaling $review_app_slug to $scale_command"
    heroku ps:scale $scale_command -a $review_app_slug
done
```