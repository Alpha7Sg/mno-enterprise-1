<p align="center">
<img src="https://raw.github.com/maestrano/maestrano-ruby/master/maestrano.png" alt="Maestrano Logo">
<br/>
<br/>
</p>

# Maestrano Enterprise Engine

The Maestrano Enterprise Engine can be included in a Rails project to as to boostrap an instance of Maestrano Enterprise Express.

The goal of this engine is to provide a base that you can easily extend with custom style or logic.

- - -

1.  [Install](#install)
2.  [Building the Frontend](#building-the-frontend)
3.  [Modifying the style - Theme Previewer](#modifying-the-style---theme-previewer)
4.  [Extending the Frontend](#extending-the-frontend)
5.  [Replacing the Frontend](#replacing-the-frontend)
6.  [Generate a database extension](#generate-a-database-extension)
7.  [Deploy a Puma stack on EC2 via Webistrano/Capistrano](#deploy-a-puma-stack-on-ec2-via-webistranocapistrano)

- - -

## Install

Add mno-enterprise to your Gemfile. A 'mno-enterprise-deployer-token' (Github oauth token) must have been generated by the 'mno-enterprise-deployer' github account.
```ruby
# Maestrano Enterprise Engine
gem 'mno-enterprise', git: 'https://<mno-enterprise-deployer-token>:x-oauth-basic@github.com/maestrano/mno-enterprise.git', branch: '3.0-dev'
```

Run the install script
```bash
rails g mno_enterprise:install
```

The install script will perform three things:
- Propose popular gems to install (e.g. Rspec)
- Generate an initializer for Maestrano Enterprise (config/initializers/mno_enterprise.rb)
- Install and build the mno-enterprise-angular frontend
- Create a /frontend directory in your application for all frontend customisations/overrides


## Building the frontend
The Maestrano Enterprise frontend is a Single Page Application (SPA) that is separate from the Rails project. The source code for this frontend can be found on the [mno-enterprise-angular Github repository](https://github.com/maestrano/mno-enterprise-angular)

Build the frontend by running the following command:
```bash
bundle exec rake mnoe:frontend:dist
```
This will create a "dashboard" directory under the /public folder with the compiled frontend.

Building the frontend is only required if you modify the CSS and/or JavaScripts files under /frontend.

## Modifying the style - Theme Previewer
The Maestrano Enterprise Express frontend is bundled with a Theme Previewer allowing you to easily modify and save the style of an Express instance without reloading the page.

The Theme Previewer is available by accessing the following path: /dashboard/theme-previewer.html
```
e.g.: http://localhost:7000/dashboard/theme-previewer.html
```

Under the hood this Theme Previewer will modify the LESS files located under the /frontend directory.

Two types of "save" actions are available in the Theme Previewer.

**Save:**  
This action will temporarily save the current style in /frontend/src/app/stylesheets/theme-previewer-tmp.less so as to keep it across page reloads on the Theme Previewer only. This action will NOT publish the style, meaning that it will NOT apply the style to the /dashboard/index.html page.

**Publish:**  
This action will save the current style in /frontend/src/app/stylesheets/theme-previewer-published.less and rebuild the whole frontend. This action WILL publish the style, meaning that it WILL apply the style to the /dashboard/index.html page.

## Extending the Frontend
You can easily override or extend the Frontend by adding files to the /frontend directory. All files in this directory will be taken into account during the frontend build and will override the base files of the mno-enterprise-angular project.

Files in this folder MUST follow the [mno-enterprise-angular](https://github.com/maestrano/mno-enterprise-angular) directory structure. For example, you can override the application layout by creating /frontend/src/app/views/layout.html in your project - it will override the original src/app/views/layout.yml file of the mno-enterprise-angular project.

You can also add new files to this directory such as adding new views. This allows you to easily extend the current frontend to suit your needs.

## Replacing the Frontend

In some cases you may decide that the current [mno-enterprise-angular](https://github.com/maestrano/mno-enterprise-angular) frontend is not appropriate at all.

In this case we recommend cloning or copying the [mno-enterprise-angular](https://github.com/maestrano/mno-enterprise-angular) repository in a new repository so as to keep the directory structure and build (Gulp) process. From there you can completely change the frontend appearance to fit your needs.

Once done you can replace the frontend source by specifying your frontend github repository in the /bower.json file. You can then build it by running the usual:
```bash
bundle exec rake mnoe:frontend:dist
```

## Generate a database extension

If you want to add fields to existing models, you can create a database extension for it.

```
rails g mno_enterprise:database_extension Model field:type
```

eg:

```
rails g mno_enterprise:database_extension Organization growth_type:string
```


## Deploy a Puma stack on EC2 via Webistrano/Capistrano
First, prepare your server. You will find a pre-made AMI on our AWS accounts called "AppServer" or "Rails Stack" that you can use.

Then, setup your new project via webistrano/capistrano.

When you're done, you can prepare the project by running the following generator for each environment your need to deploy (uat, production etc.)
```bash
# rails g mno_enterprise:puma_stack <environment>
$ rails g mno_enterprise:puma_stack production
```
This generator creates a script folder with all the configuration files required by nginx, puma, upstart and monit.

Perform a deploy:update via webistrano/capistrano (which will certainly fail). The whole codebase will be copied to the server.

Login to the server then run the following setup script
```bash
# sh /apps/<project-name>/current/scripts/<environment>/setup.sh
$ sh /apps/my-super-app/current/scripts/production/setup.sh
```
This script will setup a bunch of symlinks for nginx, upstart and monit pointing to the config files located under the scripts directory created previously.

That's it. You should be done!
