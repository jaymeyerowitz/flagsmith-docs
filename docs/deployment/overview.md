---
title: Deployment Overview
sidebar_label: Overview
sidebar_position: 1
---

## Paid Hosting

If you would rather skip the hosting and jump straight to integrating Flagsmith with your own application, you can use
[https://flagsmith.com/](https://flagsmith.com/) right now. We have
[paid plans with pricing to suit both startups and enterprise customers alike](https://flagsmith.com/pricing).

## One Click Installers

[![Deploy to DigitalOcean](https://www.deploytodo.com/do-btn-blue.svg)](https://cloud.digitalocean.com/apps/new?repo=https://github.com/flagsmith/flagsmith/tree/main)

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/flagsmith/flagsmith/tree/main)

[![Deploy on Railway](https://railway.app/button.svg)](https://railway.app/new/template?template=https%3A%2F%2Fgithub.com%2FFlagsmith%2Fflagsmith&plugins=postgresql&envs=DJANGO_ALLOWED_HOSTS%2CGUNICORN_THREADS%2CGUNICORN_WORKERS%2CPORT%2Cenv&DJANGO_ALLOWED_HOSTSDefault=%5B%27*%27%5D&GUNICORN_THREADSDefault=1&GUNICORN_WORKERSDefault=1&PORTDefault=8000&envDefault=prod)

![Fly.io](/img/logos/fly.io.svg)

We're big fans of [Fly.io](https://Fly.io)! You can deploy to fly.io really easily:

```bash
git clone git@github.com:Flagsmith/flagsmith.git
cd flagsmith
flyctl postgres create --name flagsmith-flyio-db
flyctl apps create flagsmith-flyio
flyctl postgres attach --postgres-app flagsmith-flyio-db
flyctl deploy
```

Fly.io has a global application namespace, and so you may need to change the name of the application defined in
[`fly.toml`](https://github.com/Flagsmith/flagsmith/blob/main/fly.toml) as well as the commands above.

### Caprover

You can also deploy to a [Caprover Server](https://caprover.com/) with
[One Click Apps](https://caprover.com/docs/one-click-apps.html).

## Self Hosting Overview

You will need to run through the following steps to get set up:

1. Create a Postgres database to store the Flagsmith data.
2. Deploy the API and set up DNS for it. If you are using health-checks, make sure to use `/health` as the health-check
   endpoint.
3. Visit `http://<your-server-domain:8000>/api/v1/users/config/init/` to create an initial Superuser and provide DNS
   info to the platform.
4. Deploy the Front End Dashboard and set up DNS for it. Point the Dashboard to the API using the relevant Environment
   Variables. If you are using health-checks, make sure to use `/health` as the health-check endpoint.
5. Create a new Organisation, Project, Environment and Flags via the Dashboard.
6. When using our SDKs, you will need to override the API URL that they point to, otherwise they will default to connect
   to our paid-for API at `https://api.flagsmith.com/api/v1`. See the SDK documentation for the library you are using.

## Deployment Options

We recommend running Flagsmith with [Docker](/deployment/docker). We have options to run within
[Docker](/deployment/docker), [Kubernetes](/deployment/kubernetes) or [RedHat OpenShift](/deployment/openshift).

## Architecture

The Flagsmith architecture is based around a simple REST API that is accessed by both SDK clients and the Flagsmith
Dashboard Front End Web App.

![Application Architecture](/img/architecture.svg)

## Dependencies

Running the API has the following hard dependencies:

- Postgres database(We have tested it for 11.12 but it can work for other versions too) - the main data store

The API can also optionally make use of the following 3rd party services:

- Google Analytics - for API analytics
- InfluxDB - for API analytics
- SendGrid - for transactional email
- AWS S3 - to store Django Static Assets
- GitHub - oAuth provider
- Google - oAuth provider

## InfluxDB

Flagsmith has a soft dependency on InfluxDB to store time-series data. You don't need to configure Influx to run the
platform, but SDK traffic and flag analytics will not work without it being set up and configured correctly. Once your
docker-compose is running:

1. Create a user account in influxdb. You can visit http://localhost:8086/ and do this. Create an Initial Bucket with
   the name `flagsmith_api`
2. Go into Data > Buckets and create a second bucket, `flagsmith_api_downsampled_15m`.
3. Go into Data > Tokens and grab your access token.
4. Edit the `docker-compose.yml` file and add the following `environment` variables in the api service to connect the
   api to InfluxDB:
   - `INFLUXDB_TOKEN`: The token from the step above
   - `INFLUXDB_URL`: `http://influxdb`
   - `INFLUXDB_ORG`: The organisation ID - you can find it
     [here](https://docs.influxdata.com/influxdb/v2.0/organizations/view-orgs/)
   - `INFLUXDB_BUCKET`: `flagsmith_api`
5. Restart `docker-compose`
6. Log into InfluxDB, create a new bucket called `flagsmith_api_downsampled_15m`
7. Create a new task with the following query. This will downsample your per millisecond data down to 15 minute blocks
   for faster queries. Set it to run every 15 minutes.

```text
option task = {name: "Downsample", every: 15m}

data = from(bucket: "flagsmith_api")
	|> range(start: -duration(v: int(v: task.every) * 2))
	|> filter(fn: (r) =>
		(r._measurement == "api_call"))

data
	|> aggregateWindow(fn: sum, every: 15m)
	|> filter(fn: (r) =>
		(exists r._value))
	|> to(bucket: "flagsmith_api_downsampled_15m")
```

Once this task has run you will see data coming into the Organisation API Usage area.

You will need to generate some flag evaluations within our SDKs to start to see
[Flag Analytics](/advanced-use/flag-analytics.md) data coming into the influx database.

## API Telemetry

Flagsmith collects information about self hosted installations. This helps us understand how the platform is being used.
This data is _never_ shared outside of the organisation, and is anonymous by design. You can opt out of sending this
telemetry on startup by setting the `ENABLE_TELEMETRY` environment variable to `False`.

We collect the following data on startup and then once every 8 hours per API server instance:

- Total number of Organisations
- Total number of Projects
- Total number of Environments
- Total number of Features
- Total number of Segments
- Total number of Users
- DEBUG django variable
- ENV django variable
- API server external IP address

## Running Flagsmith on Flagsmith

Flagsmith uses Flagsmith to control features on the front end dashboard. If you are self hosting the platform, you will
sometimes see features greyed out, or you may want to disable specific features, e.g. logging in via Google and Github.
If you are using your own Flagsmith environment then you will need to have a replica of our flags in order to control
access to those features.

To do this,firstly create a new project within your self-hosted Flagsmith application. This is the project that we will
use to control the features of the self-hosted Flagsmith instance. We will then point the self hosted front end
dashboard at this Flagsmith project in order to control what features show for your self hosted Flagsmith instance.

Once you have created the project, you need to set the following
[Front End](https://github.com/Flagsmith/flagsmith-frontend) environment variables in order to configure this:

- `FLAGSMITH_ON_FLAGSMITH_API_KEY`
  - The Flagsmith Client-side Environment Key we use to manage features - Flagsmith runs on Flagsmith. This will be the
    API key for the project you created as instructed above.
- `FLAGSMITH_ON_FLAGSMITH_API_URL`
  - The API URL which the Flagsmith front end dashboard should communicate with. This will most likely be the domain
    name of the Flagsmith API you are self hosting: Flagsmith runs on Flagsmith. E.g. For our SaaS hosted platform, the
    variable is `https://api.flagsmith.com/api/v1/`. For example, if you were running everything locally using the
    standard [docker-compose setup](https://github.com/Flagsmith/flagsmith-docker), you would use
    `http://localhost:8000/api/v1/`

Once you have set this up, you should see the Flagsmith front end requesting its own flags from the API (you can look in
your browser developer console to see this). You can now start creating flags and overriding the default behaviours of
the platform. For example, if you wanted to disable Google OAuth authentication, you would create a flag called
`oauth_google` and disable it.

### Current Flagsmith Feature Flags

The list of the flags and remote config we're currently using in production is below:

```json
[
 {
  "description": "Whether to show the try it buttons",
  "name": "try_it",
  "type": "FLAG"
 },
 {
  "description": "Displays top area butter bar for in app messaging",
  "name": "butter_bar",
  "value": "<html> to show at the top of the dashboard page"
 },
 {
  "description": "Turning this on will prevent users from creating any additional organisations",
  "name": "disable_create_org",
  "type": "FLAG"
 },
 {
  "description": "Disables audit log for anyone under scale-up plan",
  "name": "scaleup_audit",
  "type": "FLAG"
 },
 {
  "description": "Configures integrations",
  "name": "integration_data",
  "value": "{\n    \"datadog\": {\n        \"perEnvironment\": false,\n        \"image\": \"https://app.flagsmith.com/static/images/integrations/datadog.svg\",\n        \"docs\": \"https://docs.flagsmith.com/integrations/datadog/\",\n        \"fields\": [{\n            \"key\": \"base_url\",\n            \"label\": \"Base URL\"\n        }, {\n            \"key\": \"api_key\",\n            \"label\": \"API Key\"\n        }],\n        \"tags\": [\"logging\"],\n        \"title\": \"Datadog\",\n        \"description\": \"Sends events to Datadog for when flags are created, updated and removed. Logs are tagged with the environment they came from e.g. production.\"\n    },   \n    \"dynatrace\": {\n        \"perEnvironment\": true,\n        \"image\": \"/static/images/integrations/dynatrace.svg\",\n        \"docs\": \"https://docs.flagsmith.com/integrations/dynatrace/\",\n        \"fields\": [\n            {\n                \"key\": \"base_url\",\n                \"label\": \"Base URL\"\n            },\n            {\n                \"key\": \"api_key\",\n                \"label\": \"API Key\"\n            },\n            {\n                \"key\": \"entity_selector\",\n                \"label\": \"Entity Selector\"\n            }\n        ],\n        \"tags\": [\n            \"logging\"\n        ],\n        \"title\": \"Dynatrace\",\n        \"description\": \"Sends events to Dynatrace for when flags are created, updated and removed. Logs are tagged with the environment they came from e.g. production.\"\n    },\n    \"slack\": {\n        \"perEnvironment\": true,        \n        \"isOauth\": true,\n        \"image\": \"https://app.flagsmith.com/static/images/integrations/slack.svg\",\n        \"docs\": \"https://docs.flagsmith.com/integrations/slack/\",\n        \"tags\": [\"messaging\"],\n        \"title\": \"Slack\",\n        \"description\": \"Sends messages to Slack when flags are created, updated and removed. Logs are tagged with the environment they came from e.g. production.\"\n    },\n    \"amplitude\": {\n        \"perEnvironment\": true,\n        \"image\": \"https://app.flagsmith.com/static/images/integrations/amplitude.svg\",\n        \"docs\": \"https://docs.flagsmith.com/integrations/amplitude/\",\n        \"fields\": [{\n            \"key\": \"api_key\",\n            \"label\": \"API Key\"\n        }],\n        \"tags\": [\"analytics\"],\n        \"title\": \"Amplitude\",\n        \"description\": \"Sends data on what flags served to each identity.\"\n    },\n    \"new-relic\": {\n        \"perEnvironment\": false,\n        \"image\": \"https://app.flagsmith.com/static/images/integrations/new_relic.svg\",\n        \"docs\": \"https://docs.flagsmith.com/integrations/newrelic\",\n        \"fields\": [{\n            \"key\": \"base_url\",\n            \"label\": \"New Relic Base URL\"\n        }, {\n            \"key\": \"api_key\",\n            \"label\": \"New Relic API Key\"\n        }, {\n            \"key\": \"app_id\",\n            \"label\": \"New Relic Application ID\"\n        }],\n        \"tags\": [\"analytics\"],\n        \"title\": \"New Relic\",\n        \"description\": \"Sends events to New Relic for when flags are created, updated and removed.\"\n    },\"segment\": {\n        \"perEnvironment\": true,\n        \"image\": \"https://app.flagsmith.com/static/images/integrations/segment.svg\",\n        \"docs\": \"https://docs.flagsmith.com/integrations/segment\",\n        \"fields\": [{\n            \"key\": \"api_key\",\n            \"label\": \"API Key\"\n        }],\n        \"tags\": [\"analytics\"],\n        \"title\": \"Segment\",\n        \"description\": \"Sends data on what flags served to each identity.\"\n    },\"rudderstack\": {\n        \"perEnvironment\": true,\n        \"image\": \"https://app.flagsmith.com/static/images/integrations/rudderstack.svg\",\n        \"docs\": \"https://docs.flagsmith.com/integrations/rudderstack\",\n        \"fields\": [            {\n                \"key\": \"base_url\",\n                \"label\": \"Rudderstack Data Plane URL\"\n            },\n            {\n                \"key\": \"api_key\",\n                \"label\": \"API Key\"\n            }\n        ],\n        \"tags\": [\n            \"analytics\"\n        ],\n        \"title\": \"Rudderstack\",\n        \"description\": \"Sends data on what flags served to each identity.\"\n    },\"webhook\": {\n        \"perEnvironment\": true,\n        \"image\": \"https://app.flagsmith.com/static/images/integrations/webhooks.svg\",\n        \"docs\": \"https://docs.flagsmith.com/integrations/webhook\",\n        \"fields\": [            {\n                \"key\": \"url\",\n                \"label\": \"Your Webhook URL Endpoint\"\n            },\n            {\n                \"key\": \"secret\",\n                \"label\": \"Your Webhook Secret\"\n            }\n        ],\n        \"tags\": [\n            \"analytics\"\n        ],\n        \"title\": \"Webhook\",\n        \"description\": \"Sends data on what flags served to each identity to a Webhook Endpoint you provide.\"\n    },\n\"heap\": {\n        \"perEnvironment\": true,\n        \"image\": \"https://app.flagsmith.com/static/images/integrations/heap.svg\",\n        \"docs\": \"https://docs.flagsmith.com/integrations/heap\",\n        \"fields\": [{\n            \"key\": \"api_key\",\n            \"label\": \"API Key\"\n        }],\n        \"tags\": [\"analytics\"],\n        \"title\": \"Heap Analytics\",\n        \"description\": \"Sends data on what flags served to each identity.\"\n    },\"mixpanel\": {\n        \"perEnvironment\": true,\n        \"image\": \"https://app.flagsmith.com/static/images/integrations/mp.svg\",\n        \"docs\": \"https://docs.flagsmith.com/integrations/mixpanel\",\n        \"fields\": [{\n            \"key\": \"api_key\",\n            \"label\": \"Project Token\"\n        }],\n        \"tags\": [\"analytics\"],\n        \"title\": \"Mixpanel\",\n        \"description\": \"Sends data on what flags served to each identity.\"\n    }\n}"
 },
 {
  "description": "Determines what integrations show",
  "name": "integrations",
  "value": "[\"amplitude\",\"datadog\",\"dynatrace\",\"new-relic\",\"segment\",\"rudderstack\",\"webhook\",\"slack\", \"heap\",\"mixpanel\"]"
 },
 {
  "description": "Controls RBAC and 2FA based on organisation plan",
  "name": "plan_based_access",
  "type": "FLAG"
 },
 {
  "description": "Flag usage chart - requires InfluxDB",
  "name": "flag_analytics",
  "type": "FLAG"
 },
 {
  "description": "Organisation Analytics usage chart - requires InfluxDB",
  "name": "usage_chart",
  "type": "FLAG"
 },
 {
  "description": "Enables Dark Mode in UI",
  "name": "dark_mode",
  "type": "FLAG"
 },
 {
  "name": "segment_operators",
  "description": "Determines what rules are shown when creating a segment",
  "value": "[{\"value\":\"EQUAL\",\"label\":\"Exactly Matches (=)\"},{\"value\":\"NOT_EQUAL\",\"label\":\"Does not match (!=)\"},{\"value\":\"PERCENTAGE_SPLIT\",\"label\":\"% Split\"},{\"value\":\"GREATER_THAN\",\"label\":\">\"},{\"value\":\"GREATER_THAN_INCLUSIVE\",\"label\":\">=\"},{\"value\":\"LESS_THAN\",\"label\":\"<\"},{\"value\":\"LESS_THAN_INCLUSIVE\",\"label\":\"<=\"},{\"value\":\"GREATER_THAN\",\"label\":\"SemVer >\",\"append\":\":semver\"},{\"value\":\"GREATER_THAN_INCLUSIVE\",\"label\":\"SemVer >=\",\"append\":\":semver\"},{\"value\":\"LESS_THAN\",\"label\":\"SemVer <\",\"append\":\":semver\"},{\"value\":\"LESS_THAN_INCLUSIVE\",\"label\":\"SemVer <=\",\"append\":\":semver\"},{\"value\":\"CONTAINS\",\"label\":\"Contains\"},{\"value\":\"NOT_CONTAINS\",\"label\":\"Does not contain\"},{\"value\":\"REGEX\",\"label\":\"Matches regex\"}]"
 },
 {
  "name": "oauth_github",
  "description": "OAuth with Github - Below is a sample value - you will need to create your own credentials",
  "value": "{\n  \"url\": \"https://github.com/login/oauth/authorize?scope=user&client_id=b706a0da3e9d3115ea9d&redirect_uri=https%3A%2F%2Fapp.flagsmith.com%2Foauth%2Fgithub\"\n}"
 },
 {
  "name": "oauth_google",
  "description": "OAuth with Google - Below is a sample value - you will need to create your own credentials",
  "value": "{\n \"clientId\":\"232959427810-br6ltnrgouktp0ngsbs04o14ueb9rch0.apps.googleusercontent.com\",\n \"apiKey\":\"AIzaSyCnHuN-y6BIEAM5vTISXaz3X9GpEPSxWjo\"\n}"
 }
]
```

We also have a Segment that manages the ui Dark Mode:

Segment Name: `dark_mode` Segment Rules: Trait `dark_mode` EXACTLY MATCHES `True`

Then use this rule to override the `dark_mode` Feature Flag.

## Manual Installation

If you want a more configurable environment, you can manually install both the Front End and the API.

### Server Side API

The source code and installation instructions can be found at
[the GitHub project](https://github.com/Flagsmith/flagsmith/tree/main/api). The API is written in Python and is based on
Django and the Django Rest Framework. The Server side API relies on a Postgres SQL installation to store its data, and a
Redis installation as a cache.

### Front End Website

The source code and installation instructions can be found at
[the GitHub project](https://github.com/Flagsmith/flagsmith/tree/main/frontend). The Front End Website is written in
React/Javascript and requires NodeJS.
