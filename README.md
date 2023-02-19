This is a small php script designed to run on the command line.

IMPORTANT: This program assumes that you are running apache2 on linux mint, with php modules installed. It assumes that the apache2 config file to update is :

/etc/apache2/sites-available/000-default.conf

If your server is different, you can change the code to match your environment.

You need to run this in the root directory of a laravel as root. You give an input as the name of your project, eg:

sudo laravel_deploy my-project

It deploys a laravel project by doing the following:

* copies all the files and folders except the public/ folder to /var/www/my-project_code
* copies the contents of the /public folder to /var/www/html/my-project
* changes all the links in the /var/www/html/my-project/index.php file to point to ../../xxx.php
* updates the file permissions of everything in /var/www to 770 and changes owners to www-data and whoever you ran sudo as
* updates the /etc/apache2/sites-available/000-default.conf file with the following code:
<Directory /var/www/html/" . $project_name . ">
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>

If there is already a project deployed with the same name, the script does nothing, unless you pass the -o argument after the project name, in which case it will overwrite any existing deployed project.

