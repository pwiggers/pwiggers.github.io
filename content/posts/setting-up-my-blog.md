+++
date = '2026-11-02T21:00:00+01:00'
draft = false
title = 'Setting up my blog on GitHub Pages'
tags = ['github-pages', 'dns', 'hugo']
+++

*This post is long. If you only care about the DNS problem, [jump to the troubleshooting section](#troubleshooting-dns-issues).*

## Why this blog

I started as a system administrator, managing complete on-premises environments for different customers. At my second employer, I migrated a legacy on-premises Team Foundation Server to Azure DevOps and then moved small services to Azure.
Later, I became a 3rd-line support engineer and eventually service desk team lead.

Now I want to go back to hands-on technical IT work, and this blog is part of how I will get there: learning in public, cementing what I learn, and hopefully helping someone along the way.

## The blogging stack

For this blog, I will be making use of a stack that I am not very familiar with. This will help me get used to being uncomfortable, figuring stuff out on the go, and getting used to using the command line tools again. I must admit, I have been using AI tools to come up with the stack. Only time will tell what will stay, and what will go or change.

- **GitHub Pages with Actions**: This is where the blog will be hosted. Changes will be pushed through the command line, and publication will automatically be done through GitHub Actions. That gives me a small CI/CD pipeline for blogging.
- **Hugo**: An open source framework for building websites. It is a static website generator that turns Markdown into plain HTML with no database or server to maintain. I have no experience with this and need to figure out all the possibilities.
- **PaperMod**: This is an open source theme to be used with Hugo. Think of it as a WordPress Theme, but for Hugo.
- **Custom domain**: GitHub Pages provides you with a generic GitHub URL. Since I want something more personal, I registered my own domain.  

One of the benefits of using open source software (Hugo and PaperMod), is that there are often no costs to using it. GitHub Pages is currently free for public repositories.
In this case, only my custom domain has a cost of €30.24 per year for the .com domain.  

## Publishing articles from a MacBook Pro

For an introduction article, this is quite long. I will go over setting up the blog and an error that cost me almost an hour to resolve. This is my first setup, and I will post an updated guide when I have settled on the full stack and workflow of the site.

In the steps below, I am using placeholder data. Make sure to change that data with your own.

### Creating a site

Here is a quick run down of the setup that I have used to configure the site.

1. Install Homebrew. Visit the Homebrew website to get the most up-to-date one line installation command. This is the command at the time of writing.

   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

2. Install Git, gh, Hugo, and Visual Studio Code (that's a cask) with Homebrew. Configure git global configs for your name and email

   ```bash
   brew install git hugo gh
   brew install --cask visual-studio-code
   ```

   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "your-registered-email@github.com"
   ```

   **Highly Recommended!** GitHub allows you to create a private email address that will be used in public commits (looks like: ID+USERNAME@users.noreply.github.com). You can find this option in your account settings > Emails > Keep my email address private. There you can also learn more about configuring the variables.

3. Authenticate gh tools with GitHub to allow pushing through the command line. Make sure to choose Github.com > HTTPS, and log in through the browser.

   ```bash
   gh auth login
   ```

4. Create a new site with Hugo and add the PaperMod theme as a submodule. Execute the commands in the folder that you use for your site.

   ```bash
   hugo new site blog
   cd blog
   git init -b main
   git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
   ```

5. Edit the hugo.toml file to include the domain the site will be hosted at, the title of the site, which theme to use, and set the cacheDir for the images

   ```toml
   baseURL = 'https://blog.yourdomain.com/'
   languageCode = 'en-us'
   title = 'Website title'
   theme = 'PaperMod'

   [caches]
     [caches.images]
       dir = ':cacheDir/images'
    ```

6. Make sure that the public directory is not committed since it is being recreated every time the GitHub actions will run

   ```bash
   printf "public/\nresources/_gen/\n.hugo_build.lock\n" > .gitignore
   ```

Following the steps above, will leave you with a new site on your local device. There is nothing published yet, but the framework is there.  
You can now move on to creating a blog post.

### Writing and publishing a post

Before you can start writing, you need to create a new post

```bash
hugo new content posts/my-first-post.md
```

Writing posts will happen in Markdown, another new aspect of this journey. I am using Visual Studio Code to write the posts. An added benefit is that you can preview the layout of the post quickly by using the shortcut CMD+SHIFT+V. Another option is to have a local Hugo server running where you can preview the complete site.  
First, start the server with the command

```bash
hugo server -D
```

By using the -D option, you are also able to preview articles that are still in *Draft* status.  

After you have written your post, you need to publish it to your site. Right now, the site only exists locally, just as your post. Both will be pushed to GitHub soon, but first, a workflow needs be defined so that GitHub actions will build the site and host the posts.  

First, create the workflow file, and open it with Visual Studio Code.

```bash
mkdir -p .github/workflows
code .github/workflows/hugo.yaml
```

Paste in the workflow YAML file, you can copy the file from the [Hugo website](https://gohugo.io/host-and-deploy/deploy-to-github-pages/).  
Make sure to change the time zone (TZ) to your local time zone and save the file.  

When that is done, add all the files, commit the changes, and create a repo on GitHub to hold your site, and push all the changes.

```bash
git add .
git commit -m "Initial site"
gh repo create YOUR-USERNAME.github.io --public --source=. --remote=origin --push
```

After this has been published, go to the GitHub website and open your new repository. Go to Settings > Pages and set Source to GitHub Actions. The change is instant and the first action should run. Check the *Actions* tab in the repository. A run should be active, or have a green checkmark. If not, run the action again.  
After that, the website is live at YOUR-USERNAME.github.io

With the website live, you can now publish your post. By default, Hugo won't show posts that are still in draft mode, and you won't see anything on your site just yet. Let's change that.

- Open your post in Visual Studio Code and make sure you are happy with it
- Change the Draft status at the top from *true* to *false* and save the file.
- Push the changes to GitHub

```bash
git add .
git commit -m "Publish first post"
git push
```

This push will trigger a new GitHub Action, and your post should be live in a few seconds to minutes. Congratulations! Your first post is now live!

## Connect your personal domain

Having your website at a general github.io address is nice, but having it run at your personal (sub)domain, is even better. These steps will make sure your website is hosted at subdomain.

In theory, it is quite easy, just follow these steps

1. Go to your GitHub repo settings, and select the pages tab
2. Scroll down to the custom domain setting and enter your defined custom URL. In my case this is blog.paulwiggers.com
3. Press the Save button
4. Create a DNS CNAME record and point it to YOUR-USERNAME.github.io. For example, I have created a CNAME record for **blog** (a CNAME record is an alias fo another name)
5. After the verification, select the *Enforce HTTPS* option

### Troubleshooting DNS issues

Isn't it always DNS that gets in the way?  
In my case, it certainly was. I entered the CNAME record, pointed it to the correct GitHub URL, and set the TTL to 5 minutes. But even after 30 minutes, the DNS records couldn't be verified, and GitHub kept giving me an error message that made no sense.

![GitHub DNS Verification error message](/images/github_dns_verification.jpg)

What do you mean, I have configured an A record? I have configured a CNAME.

![DNS configuration showing CNAME record](/images/dns_record_configuration.jpg)

What could be wrong?

Time to turn to the trusty *dig* command (Dig is a command line tool to querying DNS)

```bash
dig blog.paulwiggers.com +noall +answer

; <<>> DiG 9.10.6 <<>> blog.paulwiggers.com +noall +answer
;; global options: +cmd
blog.paulwiggers.com.    298    IN    CNAME    pwiggers.github.io.paulwiggers.com.
pwiggers.github.io.paulwiggers.com. 3598 IN A    85.10.159.38
```

This clearly shows the issue. The CNAME record is pointing to *pwiggers.github.io.paulwiggers.com* instead of *pwiggers.github.io*. The wildcard A record (name pointing at an IP address) at my registrar points to the 85.10.159.38 address.
If you take a look at the configuration picture again, you might see the issue at the end of the target definition. My, and many other, DNS provider(s) needs records to end with a **.** or they will assume that the entry is relative to my own domain and will append the full domain name. Adding a . to a domain name, defines it as a fully qualified domain name (FQDN).

I quickly added a . to the CNAME target, waited a few minutes and gave the dig command another shot

```bash
dig blog.paulwiggers.com +noall +answer           

; <<>> DiG 9.10.6 <<>> blog.paulwiggers.com +noall +answer
;; global options: +cmd
blog.paulwiggers.com.   300.  IN.  CNAME.  pwiggers.github.io.
```

The DNS check of GitHub can lag behind, so make sure that it is properly resolved before moving on. Also, the enforce HTTPS option only becomes available after a certificate has been issued. This process will start after the DNS has been verified.

That is looking much better! The record now resolves to the proper target and the site is live.
I can now push this post to a live website with a few simple commands

1. ```git add .```
2. ```git commit -m "publish first post"```
3. ```git push```

As you can see, the post is live. How a small . can have such an impact.

The key take away from this:

- Check with dig to make sure the DNS configuration matches your expectations
- Don't trust the DNS web UI to always display the correct values
- Know how your registrar handles relative names

Thank you for reading and until next time!

Paul
