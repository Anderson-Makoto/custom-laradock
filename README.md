#TURNOVERBNB DOCKER:

### This repo is a collection of docker containers that will help us setup our development environment ###

### This guide will walk you through the configuration. ###

### We will need docker ce and docker compose. ###
### This tutorial should work for both linux and mac, but the docker install has some special tips for Mint users ###


#STAGE 0: Clone this repo to your machine. 
I like to use ~/turnoverbnb/docker and I will assume this folder is used in the tutorial.

This tutorial also assumes you have the laravel repo cloned in ~/turnoverbnb/laravel.

Useful aliases:
alias tbnbup='docker-compose up -d mysql nginx redis'
alias tbnbssh='docker-compose exec --user laradock workspace bash' 

You can add then to the end of ~/.bashrc on linux (not sure how to add aliases on mac).
Do . ~/.bashrc to make your changes take effect.

Now all you need to do to start the project is enter the docker folder and type 'tbnbup' and after that
type 'tbnbssh' to enter the workspace machine. This will be useful later in this tutorial.

#STAGE 1: Download Docker CE and docker compose.
Install Docker CE on Mac: https://docs.docker.com/docker-for-mac/install/

Install Docker CE on Ubuntu: https://docs.docker.com/install/linux/docker-ce/ubuntu/

Important! For linux mint, follow the ubuntu tutorial and but when adding the repositiory, instead of doing this:

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

Do this:

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(. /etc/os-release; echo "$UBUNTU_CODENAME") stable"

Install docker compose: https://docs.docker.com/compose/install/

Finally, there is a last step for linux users if you don't want to run sudo for every docker command. Not sure if this works for Mac.

sudo groupadd docker

sudo usermod -aG docker $USER

Reboot

#STAGE 2: Configure this repo's .env to point to your local laravel folder and set your innodb variables.
Just do nano .env and update the "APP_CODE_PATH_HOST=" value with the path to your laravel folder. 

By default it's: APP_CODE_PATH_HOST=~/turnoverbnb/laravel

Remove the .example from the .sql file that is going to create the dump db:

cd mysql/docker-entrypoint-initdb.d/

cp createdb.sql.example createdb.sql

Now, go back to mysql/ and edit mysql.cnf.
Adjust the variables according to your PC's avialable RAM:

innodb_log_file_size=1GB

innodb_buffer_pool_size=4GB

Usually, I set the innodb_log_file_size to 1GB and the innodb_pool_size to around 25% the total RAM.

#STAGE 3: Configure your laravel .env to docker
All you need to do is to replace some variables on laravel's .env.

Replace DB_HOST=127.0.0.1 with DB_HOST=mysql

Replace REDIS_HOST=127.0.0.1 with REDIS_HOST=redis

That's all, the dump and homestead dbs remain and you can switch between then just like before.
The password is also the same, so don't worry about it.

#STAGE 4: Start the components
In the docker folder, type "docker-compose up -d mysql redis nginx" (or use the tbnbup alias)

If you just do docker-compose up, it's going to start a lot of unnecessary machines, so use the -d parameter and 
pass those three components. They are all we need.

After adding those three components, the workspace component will also be installed. The workspace is where 
our laravel repo code will be. You will login to it to use tinker, configure passport, run migrations and etc.

Think of entering the workspace component to be the same as doing vagrant ssh on homestead. 

I also strongly suggest you to quickly take a look at the laradock documentation, it's pretty good!

https://laradock.io/documentation/

#STAGE 5: Run the seeds or the dump
Login on the workspace component with the laradock user (always recommended, unless you need root for some reason)

To do it, run "docker-compose exec --user laradock workspace bash" (or use the tbnbssh alias).

Now your are on the command line of the workspace component. 

Here is where you will be able to run art migrate:reload (to create a fresh seeded db) or art dump:restore (to restore a production db)
 normally and use tinker normally when you need. (don't do this now).

Just remember to configure s3cmd before running a dump restore. S3cmd is already installed, but not configured.

#STAGE 6: Configure your hosts file
It should be something like this (remember it's a tab between the Ip and the hostname):

127.0.1.1	laravel.test

Now just type laravel.test on the browser and you should see the app!

It should be laravel.test, if you want to change it, you need to edit the nginx config files.

#STAGE 7: Run composer update and npm install
If you haven't done this already, enter your workspace (docker-compose exec --user laradock workspace bash)
and run "composer update" and then "npm install".
Also run "art key:generate" to generate an app key in your .env.

Finally, run art migrate:reload to create a new seeded db.

#STAGE 8: Configuring phpunit and xdebug
This is the last step, make sure everything is working before doing it.
Also remember to start your components.
You don't have to touch the components to install phpunit or xdebug, everything is already configured there.
All you need to do is to configure the interpreter on phpstorm.

###1- Configure the connection between docker and phpstorm

On phpstorm, go to Settings->Build, Execution, Development->Docker

On the right side, click the plus sign to add a docker connection using the unix socket option. 

This is slightly different for Mac users, that probably will only need to click the plus sign and then ok without 
changing anything. 

###2- Configure the interpreter

Go to Settings->Languages & Frameworks->PHP

Here you will find the CLI interpreter option.

Click on "..." on the right side of it to add a new interpreter. 

Click the plus sign and select the "From Docker, Vagrant..." option.

Select the Docker Compose option (not Docker!)

On server, select the connection you setup on step 1.

On configuration files, go to the root of this repo and select the docker-compose.yml file.

After it's loaded, chose the service that is going to be used. Select workspace.

Click on the refresh button under "General" and make sure that the PHP and Xdebug versions are shown.

Click apply. 

On this page, you will also need to configure the path mappings, it's the third option from the top of the screen.

Click on the folder on the right side, then the plus sign and add an entry with the path to the laravel project
on your machine on the left side and with /var/www on the right side. 

On my config, it was like this:

/home/alexandre/turnoverbnb/laravel	/var/www

Click Ok.


###3- Configure the test framework

Go to Settings->Languages & Frameworks->PHP->Test Frameworks and click the plus sign to add a new interpreter.

Select the "PHPUnit by Remote Interpreter" option and select the intepreter you've created in step 2. It should
appear as "workspace".

Make sure the PHPUnit library section shows the phpunit version.

Click ok to save and then apply.
Now the tests and the debugger should work.

###4- Configure the servers to debug from browser

Go to Settings->Languages & Frameworks->PHP->Servers and add click the plus sign to add a server
and set its name to laradock (it's important that this name is used).

On host, put laravel.test.

Check the use path mappings option and navigate to your laravel folder under Project Files. 

On the right side of the laravel folder, type /var/www and type enter. Apply.

Ready!

#End of readme.

