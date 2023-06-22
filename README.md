# laravel_deploy

This is a command line php script written in PHP for deploying a Laravel project from development to production. It is designed to work for laravel-only projects as well as laravel-vue3-inertiajs projects.

In brief, it takes the name of the project, copies the contents of the public folder to `/var/www/html/project-name`, and the rest of the project files are copied to `/var/www/project-name_code`. It does a lot more than this, described below, as well as doing stuff in order to solve various routing problems caused by deploying a project from http://localhost:8000 to http://localhost/project-name, using my own personal built-in solutions, but it should still work fine with other generic Laravel projects, as long as they are intended to work on a directory above the root of the server url.

Here is an example of the project being run on a linux webserver:

```
alex@myserver:~/dev/www/my-project$ sudo laravel_deploy 
Project name: (leave blank for "my-project"):
There's already a deployed project called my-project - replace? (y/n):y
Apache2 config file? (leave blank for default: /etc/apache2/sites-available/000-default.conf):
..........................
- Deleting old directory /var/www/my-project_code
- Deleting old directory /var/www/html/my-project
- Running npm run build
- Copying public files to /var/www/html/my-project
- Copying php files to /var/www/my-project_code
- Updating the links in index.php
- Changing the permissions of all the copied files to 770 and changing ownership to www-data and alex
- Restarting the Apache webserver
..........................
Your new project has been deployed to http://localhost/my-project
```

In full, it does the following:

* check that you did sudo, and that the working directory you are in contains directories consistent to being in the root of a Laravel project
* ask the user for the project name, if different from the name of the directory you are in
* ask the user for the apache2 config file that will need to be updated, if different from /etc/apache2/sites-available/000-default.conf
* run npm run dev if `node_modules` directory is found
* copy `public/` directory to `/var/www/html/project-name`
* copy `app/` `bootstrap/` `config/` `resources/` `routes/` `storage/` and `vendor/` directories to `/var/www/project-name_code`, skipping out `node_modules/`, `database/`, `storage/` and `test/` directories
* fixes a bug in inertiajs 1.0 that doubles up the subdirectory, ie. that changes a link, eg. `http://localhost/my-project/login` to http://`localhost/my-project/my-project/login` - see comments in the code for more info on this
* amends `index.php` in the public directory to make the links reach the new location of the php files (ie. changing `../` to `../../my-project_code/`)
* adds the following lines to the apache2 configuration file (if they are not already present):
```
# ------- 
# created automatically by Alex Windsor's Laravel Deployment script
<Directory /var/www/html/project-name>
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
# -------
```
* for InertiaJS projects, scan the `/build/assets` folder to get the names of all the `.css` and `.js` files, then replaces the following directives in `resources/views/app.blade.php`:
```
@vite('resources/js/app.js')
@vite('resources/css/app.css')
```
...with `<link rel="stylesheet"...` and `<script type="module" src="...` tags
* runs chmod and chown commands to update permissions to 770 and update ownership to www-data and whatever your username is on the server
* restarts the apache2 webserver