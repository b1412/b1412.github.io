---
layout: post
title: "Set up octopress on Ubuntu14.04"
date: 2015-12-23 23:36:53 +0800
comments: true
categories: Octopress, Markdown
tags: Octopress, Markdown
keywords: Octopress, Markdown
---
This is a step-by-step tutorial that you can follow to create your own blog by
octopress on Ubuntu14.04 and delopy to github pages.

In the tutorial,I will take my own [blog](http://blog.kidzhou.me "http://blog.kidzhou.me") as an example.

For more details about octopress, go to [offical document](http://octopress.org/docs "http://octopress.org/docs").

<!--more-->

##Install git

The build-in git in Ubuntu is too old.

```
sudo add-apt-repository ppa:git-core/ppa
sudo apt-get update
sudo apt-get install git
```

##Install ruby

###Add the PPA and install the package

```
sudo apt-add-repository -y ppa:rael-gc/rvm
sudo apt-get update
sudo apt-get install rvm
```
###Change your terminal window
Now, in order to always load rvm, change the Gnome Terminal to always perform a login.

At terminal window, click `Edit` > `Profile Preferences`, click on `Title and Command` tab and check `Run command as login shell`.

![Terminal Screenshot](/images/terminal.png)

###Logout and login
A lot of changes were made and in order to properly get all them working, you need to login and logout. This requires not only close terminal, but really logout and login again.

###Install a ruby
```
rvm install ruby
rvm rubygems latest 
```
Run ```ruby --version``` to be sure you're using Ruby 1.9.3.

##Install rake

```
sudo gem install rake
```

##Setup octopress

```
git clone git://github.com/imathis/octopress.git octopress
cd octopress
```
##Install octopress default theme

```
sudo gem install bundler
bundle install
rake install
```

##Install other theme (optional)

Install slash theme instead of default theme.

```
cd octopress
git clone git://github.com/tommy351/Octopress-Theme-Slash.git .themes/slash
rake install['slash']
rake generate
```
NOTE: Have problems when installing with zsh? Try ``` rake install\['slash'\] ```

##Github setup

* Create a github account and my account is [`b1412`](https://github.com/b1412 "https://github.com/b1412").

* Create a new Github repository and name the repository with the format username.github.io, where username is your GitHub user name or organization name.In this case, the repository is [`b1412.github.io`](https://github.com/b1412/b1412.github.io "https://github.com/b1412/b1412.github.io")

```
rake setup_github_pages
```

The rake task will ask you for a URL of the Github repo. Copy the SSH or HTTPS URL from your newly created repository
(in this case, https://github.com/b1412/b1412.github.io.git) and paste it in as a response.

This will:

* Ask for and store your Github Pages repository url.
* Rename the remote pointing to imathis/octopress from 'origin' to 'octopress'
* Add your Github Pages repository as the default origin remote.
* Switch the active branch from master to source.
* Configure your blog's url according to your repository.
* Setup a master branch in the _deploy directory for deployment.


##Create a new post

```
rake new_post["title"]
```
##Preview

```
rake generate
rake preview
```
You can preview these posts on your local server [http://localhost:4000](http://localhost:4000 "http://localhost:4000")

``new_post`` expects a naturally written title and strips out undesirable url characters when creating the filename.
The default file extension for new posts is markdown but you can configure that in the Rakefile.

##Deploy

```
rake deploy
```

**Do not forget** to push source branch to github.

```
git add .
git commit -m 'your message'
git push origin source
```


##Customize domain(optional)

###Add CNAME file

echo 'blog.kidzhou.me' >> source/CNAME

###Update DNS record

For this blog, I only add a sub-domain.

![DNS configuration](/images/dns.png "dns")

###Re-deploy
```
rake generate
rake deploy
```
**Do not forget**  ```rake generate```, otherwise you may face 404 error using new domain.


##References

* https://github.com/rvm/ubuntu_rvm

* http://pangyi.github.io/blog/20150125/ubuntuzhong-octopressan-zhuang

* https://github.com/tommy351/Octopress-Theme-Slash

* http://codelife.me/blog/2012/09/02/custom-domain-for-octopress
