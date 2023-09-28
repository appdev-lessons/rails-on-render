# Deploying a Rails app to Render

Congratulations! You've built a web application using Ruby on Rails, and now it's time to share it with the world. In this guide, we'll walk you through the process of deploying your Rails app on the internet using [Render](https://render.com). 

This guide is similar to [our previous guide](https://learn.firstdraft.com/lessons/214-deploying-to-render) for deploying static and dynamic Sinatra apps, so you may want to quickly review that guide (i.e. be sure you have a Render account and you understand how to deploy from a Blueprint before you proceed).

Specifically, this guide covers deploying a **database-backed** app. But first, let's see how to deploy a Rails app that does not have a database.

## Before we begin

As a reminder, Web Services on the free instance type for Render are automatically spun down after 15 minutes of inactivity. When a new request for a free service comes in, Render spins it up again so it can process the request.

This will cause a delay in the response of the first request after a period of inactivity while the instance spins up.

If you decide to host larger apps, and want something more powerful to serve real customers, then you can easily scale Render up to a paid plan. Alternatively, you may end up going with a service like Heroku, Fly or hatchbox.io. But, the Render individual plan should be sufficient (and free!) for now.

## Deploying a non-database Rails app

These are the first steps you should take to deploy any Rails app on Render; even ones like "Rock, Paper, Scissors" that don't have a database.

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

### Credentials and master key

When we create a new Rails app a file called `credentials.yml.enc` is added to the `config` directory. This file contains some of the application's sensitive information in an encrypted format. Some of that information is required by Render to properly deploy the app. In order for Render to decrypt the file, we need a key stored in a `config/master.key`. 

We haven't provided this key for your app, so you will need to follow these steps.

First, delete the `config/credentials.yml.enc` file, since we will generate a new one:

![](/assets/delete-original-credentials.png)
{: .bleed-full }

Now, run this command at the bash prompt:

```
% EDITOR="code --wait" rails credentials:edit
```

That will open a temporary file in your editor window in a new tab. This temporary file displays the `secret_key_base` that will be encrypted by the new `credentials.yml.enc` file. Once you've had a look at the file, you can simply close it in the editor tab (that file will be erased once you close it):

![](/assets/new-credentials.png)
{: .bleed-full }

After you close that temporary file, your `config/` folder looks something like this:

---

![](/assets/updated-config-folder.png)

---

Note the two files I've highlighted that were created by this process:

- A new `credentials.yml.enc` file: This is the encrypted file that will be decrypted with...
- the new `master.key` file.

<div class="bg-red-100 py-1 px-5" markdown="1">

The `master.key` file has a light gray, muted color. That's because it is included in our `.gitignore` file. We never want to publish this key to GitHub, and we won't make that mistake since we are `.gitignore`-ing the file! 

That `master.key` will be stored safely in our codespace, but you should also copy it to another secure location in case your codespace is eventually deleted due to inactivity. Now could be a great time to start using a password manager like [Bitwarden](https://bitwarden.com/) to safely store information like this. (Bitwarden and other password managers aren't limited to passwords; you can store other sensitive information like API keys there.)
</div>

Now you can run a git commit and push to save the new `credentials.yml.enc` file. (This one is encrypted, so we can store it on GitHub.)

### Deploy to Render

With our credentials and `master.key` setup, it's time to deploy!

Return to the previous guide, and follow the steps to [create a new Blueprint](https://learn.firstdraft.com/lessons/214#create-a-new-blueprint) and then [deploy the Blueprint](https://learn.firstdraft.com/lessons/214#deploy-your-blueprint).

Once you've done that, there's another step to take in the dashboard for your app.

Visit [dashboard.render.com](https://dashboard.render.com/) and open the app you just created. Go to the "Environment" tab, and in the "Environment Variables" add a key called `RAILS_MASTER_KEY` with the value copy-pasted over from your `config/master.key` file. Be sure to "Save Changes" when you've done so:

![](/assets/set-master-key-env-on-render.png)
{: .bleed-full }

This will trigger another deployment of your app (visit the "Events" tab to view the progress). When the deployment finishes, your app will be live!

---

Are you looking to deploy an app that has a database? Read the next section for the additional steps to take there!

## Deploying a database-backed Rails app

<div class="bg-red-100 py-1 px-5" markdown="1">

Databases on the free plan are deleted after 90 days. Also, free plans are only eligible to have one database. If you plan to use your app beyond 90 days, or if you plan to have multiple database-backed apps, consider upgrading to a paid plan.
</div>

<aside markdown="1">
Detailed instructions on creating Blueprints can be found in the [Render Docs](https://render.com/docs/deploy-rails#deploy-to-render).
</aside>


---
