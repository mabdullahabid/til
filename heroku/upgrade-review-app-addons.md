# Upgrade Review App Addons based on Production Addons

``` bash
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

# Define the production app slug
production_app_slug="crowdbotics-slack"

# Fetch addon config from production
production_addons=$(heroku addons --json --app $production_app_slug)

# Upgrade CloudAMQP
cloudamqp_plan=$(echo "$production_addons" | jq -r '.[] | select(.addon_service.name == "cloudamqp" and .config_vars[] == "CLOUDAMQP_GREEN_APIKEY") | .plan.name')
if [ -n "$cloudamqp_plan" ]; then
    echo "Destroy exsting cloudamqp addon on $review_app_slug"
    heroku addons:destroy cloudamqp --app $review_app_slug --confirm $review_app_slug

    echo "Create new cloudamqp addon with plan $cloudamqp_plan on $review_app_slug"
    heroku addons:create $cloudamqp_plan --app $review_app_slug
fi

# Provision Heroku Redis
redis_plan=$(echo "$production_addons" | jq -r '.[] | select(.addon_service.name == "heroku-redis" and .config_vars[] == "REDIS_URL") | .plan.name')
if [ -n "$redis_plan" ]; then
    echo "Provisioning heroku-redis with plan $redis_plan on $review_app_slug"
    heroku addons:upgrade $redis_plan --app $review_app_slug
fi

# Upgrade New Relic
newrelic_plan=$(echo "$production_addons" | jq -r '.[] | select(.addon_service.name == "newrelic" and .config_vars[] == "NEW_RELIC_LICENSE_KEY") | .plan.name')
if [ -n "$newrelic_plan" ]; then
    echo "Upgrading newrelic to plan $newrelic_plan on $review_app_slug"
    heroku addons:upgrade $newrelic_plan --app $review_app_slug
fi

# Provision Heroku PostgreSQL
postgresql_plan=$(echo "$production_addons" | jq -r '.[] | select(.addon_service.name == "heroku-postgresql" and .config_vars[] == "DATABASE_URL") | .plan.name')
if [ -n "$postgresql_plan" ]; then

    OLD_DB=$(heroku addons --json --app $review_app_slug | jq -r '.[] | select(.name | startswith("postgresql-")).name')

    # echo "Provisioning heroku-postgresql with plan $postgresql_plan on $review_app_slug"
    heroku addons:create $postgresql_plan --app $review_app_slug

    # # Wait for the new database to be available
    heroku pg:wait --app $review_app_slug

    NEW_DB=$(heroku addons --json --app $review_app_slug | jq -r '.[] | select(.config_vars | .[] | startswith("HEROKU_POSTGRESQL_")) | .name')

    # Print the values to verify
    echo "OLD_DB: $OLD_DB"
    echo "NEW_DB: $NEW_DB"

    # Copy the data from the old database to the new database
    heroku pg:copy $OLD_DB $NEW_DB --app $review_app_slug --confirm $review_app_slug

    # Promote the new database
    heroku pg:promote $NEW_DB --app $review_app_slug

    # Remove the old database
    heroku addons:destroy $OLD_DB --app $review_app_slug --confirm $review_app_slug
fi
```