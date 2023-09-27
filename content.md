# Deploying a Rails app to Render

Congratulations! You've built a web application using Ruby on Rails, and now it's time to share it with the world. In this guide, we'll walk you through the process of deploying your Rails app on the internet using [Render](https://render.com). 

This guide is similar to [our previous guide](https://learn.firstdraft.com/lessons/114-deploying-to-render) for deploying static and dynamic Sinatra apps, so you may want to quickly review that guide (i.e. be sure you have a Render account and you understand how to deploy from a Blueprint before you proceed).

Specifically, this guide covers deploying a **database-backed** app. But first, let's see how to deploy a Rails app that does not have a database.

## Before we begin

As a reminder, Web Services on the free instance type for Render are automatically spun down after 15 minutes of inactivity. When a new request for a free service comes in, Render spins it up again so it can process the request.

This will cause a delay in the response of the first request after a period of inactivity while the instance spins up.

If you decide to host larger apps, and want something more powerful to serve real customers, then you can easily scale Render up to a paid plan. Alternatively, you may end up going with a service like Heroku, Fly or hatchbox.io. But, the Render individual plan should be sufficient (and free!) for now.

## Deploying a non-database Rails app

### Update the `render.yaml` file

As in the previous render guide, let's set up the `render.yaml` file in your project repository. This file tells Render how to manage your app's deployment.

Create a new file named `render.yaml` in your project root directory if you do not find one there already and replace `MYAPPNAME` with your application's name (e.g. `hello-world` or something else; only use letters and dashes in the name, no spaces).

```yaml{3}
services:
  - type: web
    name: MYAPPNAME # the name of this service, eg your app name
    env: ruby # this app is written in ruby
    plan: free # make sure to set this to free or you'll get billed $$$
    buildCommand: "./bin/render-build.sh" # we already created these two files for you
    startCommand: "./bin/render-start.sh" 
```

Commit and push this change to your repository to proceed.

### Credentials and key

In order to 


## Deploying a database-backed Rails app

<div class="bg-red-100 py-1 px-5" markdown="1">

Please note that databases on the free plan are deleted after 90 days. If you plan to use your app beyond 90 days, consider upgrading to a paid plan.
</div>

<aside markdown="1">
Detailed instructions on creating Blueprints can be found in the [Render Docs](https://render.com/docs/deploy-rails#deploy-to-render).
</aside>


---
