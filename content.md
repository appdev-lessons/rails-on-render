# Deploying a Rails app to Render

Congratulations! You've built a web application using Ruby on Rails, and now it's time to share it with the world. In this guide, we'll walk you through the process of deploying your Rails app on the internet using [Render](https://render.com). 

This guide is similar to [our previous guide](https://learn.firstdraft.com/lessons/114-deploying-to-render) for deploying static and dynamic Sinatra apps, so you may want to quickly review that guide (i.e. be sure you have a Render account and you understand how to deploy from a Blueprint before you proceed).

Specifically, this guide covers deploying a **database-backed** app. But first, let's see how to deploy a Rails app that does not have a database.

## Before we begin

As a reminder, Web Services on the free instance type for Render are automatically spun down after 15 minutes of inactivity. When a new request for a free service comes in, Render spins it up again so it can process the request.

This will cause a delay in the response of the first request after a period of inactivity while the instance spins up.

If you decide to host larger apps, and want something more powerful to serve real customers, then you can easily scale Render up to a paid plan. Alternatively, you may end up going with a service like Heroku, Fly or hatchbox.io. But, the Render individual plan should be sufficient (and free!) for now.

## Deploying a non-database Rails app

These are the first steps you should take to deploy any Rails app on Render; even ones like "Rock, Paper, Scissors" that don't have a database.

### Update the `render.yaml` file

As in the previous render guide, let's set up the `render.yaml` file in your project repository. This file tells Render how to manage your app's deployment.

Open the file called `render.yaml` in your codespace (you should see it on the left panel file explorer in the root directory, i.e. not in a sub-folder). 

If you don't find the file, you can create it, and _carefully_ copy and paste this code into that new `render.yaml` file:

```yaml{3,8-10}
services:
  - type: web
    name: MYAPPNAME # the name of this service, eg hello-world
    env: ruby # this app is written in ruby
    plan: free # make sure to set this to free or you'll get billed $$$
    buildCommand: "./bin/render-build.sh" # we already created these two files for you
    startCommand: "./bin/render-start.sh"
    envVars: # this section sets some ENV variables needed by Render for deployment
      - key: SECRET_KEY_BASE
        generateValue: true
```

Make one change to this file: replace `MYAPPNAME` with your application's name (e.g. `rock-paper-scissors` or something else; only use letters and dashes in the name, no spaces). 

Also, note the new section in this Rails-specific `render.yaml`: We need to tell Render to set a `SECRET_KEY_BASE` environment variable when we deploy. _You don't need to make any changes to these lines_. This step allows the production app to decrypt any credentials we might have encrypted in the `config/credentials.yml.enc` file.

Commit and push any `render.yaml` changes to your repository to proceed.

### Deploy to Render

Return to the previous guide, and follow the steps to [create a new Blueprint](https://learn.firstdraft.com/lessons/114-deploying-to-render#create-a-new-blueprint) and then [deploy the Blueprint](https://learn.firstdraft.com/lessons/114-deploying-to-render#deploy-your-blueprint).

When the deployment finishes, your app will be live!

Remember, you can interact with you app anytime on your [Render dashboard](https://dashboard.render.com/).

---

Are you seeking to deploy an app that has a database? Read the next section for the additional steps to take there!

## Deploying a database-backed Rails app

<div class="bg-red-100 py-1 px-5" markdown="1">

Databases on the free plan are deleted after **90 days**. Also, free plans are only eligible to have **one database**. If you plan to use your app beyond 90 days, or if you plan to have multiple database-backed apps, consider upgrading to a paid plan. You can always upgrade your plan at a later point, so you can begin with the free offering, and later upgrade to a $7/month starter plan, which is sufficient for most side projects.

Alternatively, you can connect an external database that will not be deleted via another free service: [ElephantSQL](https://www.elephantsql.com/). Keep reading to see details on those additional ElephantSQL steps; but you should begin by following the steps immediately below to deploy with a database, and then later you can connect the external ElephantSQL database if you so choose.
</div>

### Update the `render.yaml` file

Since we can only deploy one database-backed app for free on Render, we need to add to our `render.yaml` file:

<aside markdown="1">
Detailed instructions on creating Blueprints can be found in the [Render Docs](https://render.com/docs/deploy-rails#deploy-to-render). You will note some differences between their guide and ours, which is more beginner friendly.
</aside>

```yaml{3,11-18}
services:
  - type: web
    name: MYAPPNAME # the name of this service, eg hello-world
    env: ruby # this app is written in ruby
    plan: free # make sure to set this to free or you'll get billed $$$
    buildCommand: "./bin/render-build.sh" # # we already created these two files for you
    startCommand: "./bin/render-start.sh"
    envVars: # this section sets some ENV variables needed by Render for deployment
      - key: SECRET_KEY_BASE
        generateValue: true
      - key: DATABASE_URL
        fromDatabase:
          name: db
          property: connectionString
databases: # this section provides some additional database configuration
  - name: db
    plan: free
    ipAllowList: []
```

In addition to generating a `SECRET_KEY_BASE`, you need to add a `DATABASE_URL` environment variable and configure it properly by _carefully_ copy-pasting the above `render.yaml` changes into your file. Again, the only thing you need to change in this file is the `MYAPPNAME` to a name of your choosing (e.g. `msm-queries`).

### Updating the `bin/render-build.sh` file

There's one more step to take. Open the `bin/render-build.sh` file in your project repository and uncomment the last three lines:

```bash{8-10}
#!/usr/bin/env bash
# exit on error
set -o errexit

bundle install

# For Ruby on Rails apps uncomment these lines to precompile assets and migrate your database.
bundle exec rake assets:precompile
bundle exec rake assets:clean
bundle exec rake db:migrate
```

These lines are specific to a full-fledged, database-backed Rails app, and will make sure that the app deploys correctly and the database migration is run.

### Deploy to Render

After you make the changes to `render.yaml` and `bin/render-build.sh` (**and git commit and push those changes**): 

Return to the previous guide, and follow the steps to [create a new Blueprint](https://learn.firstdraft.com/lessons/114-deploying-to-render#create-a-new-blueprint) and then [deploy the Blueprint](https://learn.firstdraft.com/lessons/114-deploying-to-render#deploy-your-blueprint).

This time around, you will see that Render is not only going to create a "Web Service", but also a "Database" for us. If you don't see the two services, then you may not have setup your `render.yaml` file correctly in the previous step:

![](/assets/creating-db-and-webservice.png)
{: .bleed-full }

If you have already setup one free database-backed app (the `plan: free` option in the `render.yaml` section for `database:`), then you will be asked to "Update Existing Resource". You will need to either: remove the other database, pay for a new database, or use an alternative deployment service that offers more free databases with some other caveats (i.e. Fly, which requires credit card information on file):

![](/assets/no-second-free-db.png)
{: .bleed-full }

When the deployment finishes, your app will be live!

Because we set Render to deploy from our `main` branch and connected the app with the GitHub repository, anytime you make a change to your app and commit and push that change, the live app will re-deploy with your updates!

Remember, you can interact with you app anytime on your [Render dashboard](https://dashboard.render.com/).

<div class="bg-red-100 py-1 px-5" markdown="1">

Before you begin CRUDing in your live app to add records to the database, consider whether you want the database around for longer than 90 days. If you do want your records to persist and not be deleted (and not have to pay), then follow the steps in the next section to connect a free, persistent ElephantSQL database.
</div>

### Connect an external database with ElephantSQL

Your free database provided by Render will be deleted after 90 days if you do not upgrade to a paid plan. 

As an alternative to paying, follow these steps to connect a free, external database provided by [ElephantSQL](https://www.elephantsql.com/) to your deployed Render app. This database will not be deleted!

First follow the steps in [this guide from ElephantSQL](https://www.elephantsql.com/docs/index.html) to:

- Sign up with your GitHub account
- Create a "New instance"
- Use the plan "Tiny Turtle (Free)", and provide a name (e.g. `my-app-name`)
- Select a region (choose a region nearby you for low latency)
- After you complete the steps, you should be brought to a dashboard page where you can find a URL that looks something like this:

```
postgres://xxxxx:yyyyyy@zzzzz.db.elephantsql.com/xxxxx
```

The format of that is standard for database connections and breaks down to:

```
postgres://[USERNAME]:[PASSWORD]@[HOSTNAME]/[DATABASE_NAME]
```

where the username and database name are typically identical.

![](/assets/elephant-sql-details-page.png)

<div class="bg-red-100 py-1 px-5" markdown="1">

Note the "Max database size" on the Tiny Turtle free plan is only 20 MB. This is pretty small, and as your app and database grows you may hit this limit fairly quickly. In that case, follow the steps later in this guide on [Migrating Databases](#migrating-databases) back to Render, and upgrading to a paid plan there. Unfortunately, when your database starts to grow there's no free lunch!
</div>

Once you have that long URL from ElephantSQL copied to your clipboard, you can head to your [Render dashboard](https://dashboard.render.com/), and find the app you wish to connect the external database. On the dashboard for your app, click on "Environment" and note the `DATABASE_URL`:

- Delete the current `DATABASE_URL` environment variable with the trash icon
- Click "Add Environment Variable"
- Create a new variable, again with the key `DATABASE_URL`, but this time with the value taken from ElephantSQL (e.g. `postgres://username:password@hostname/databasename`)
- Click "Save Changes"

![](/assets/delete-free-db-url.png)
{: .bleed-full }

This will trigger a new deployment of your app, and now your live app will be connected to the database on ElephantSQL. At this point, you could even delete the old database from your Render dashboard:

![](/assets/delete-free-db.png)
{: .bleed-full }

**Before you delete it, make sure there are no records you want to keep.** If the Render database does have some records that you are interested in keeping, then you will need to migrate those records from the Render-supplied database to your new, external ElephantSQL database. 

Similar to how you connected your external database, Render also provides you with the URL to connect via the `DATABASE_URL` environment variable to the internal database that they provide. You can find that in the dashboard page for your database on Render. **This is exactly the `DATABASE_URL` environment variable that you deleted in the previous step.** Your database and contents were not deleted by the step, you only deleted (well, _exchanged_) the connection to the database!

![](/assets/internal-db-connection.png)
{: .bleed-full }

## Database Backups

Database backups are critical for any long term deployment. Render supplies robust infrastructure for hosting applications, but ensuring the integrity and availability of your data is still important. Among other reasons, backups are important:

- As a safety net, protecting your data from accidental deletions, updates, or other data corruptions. 
- As a way to revert your database to a previous state.
- As a way to migrate your database to another platform (e.g. ElephantSQL, Heroku, Fly, etc.)

### Backups on Render

- On render
- On Elephant


## Migrating Databases

- On render
- On elephant

![](/assets/elephant-sql-details-page.png)
{: .bleed-full }

## Background Scheduled Jobs



## Continuous Deployment

---
