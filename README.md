# Building a Cloud Server to run Django Applications

Author: [Desmond Duggan](www.desmondrduggan.com)

## Step 1. Setup the Server

The first step to get your Django site up and running is to get a cloud server. I recommend using Digital Ocean, but there are many options including Linode and Amazon AWS. 

You only need the basic server, so go with the lowest price solution running an Ubuntu OS. 

Test drive your server by logging in as root: 

	$ ssh root@your.new.ip.address
	
##### User accounts

It is bad practice to operate your server with the root user, so make your default user account you plan to use. 

After you ssh to the server with your root user, execute the following commands to create a new user:

	$ sudo adduser bilbo
	$ sudo adduser bilbo sudo
	
Then you can ssh with that user:

	$ ssh bilbo@your.new.up.address
	

##### SSH Keys
Great, now you can login to your server, but each time to ssh, you will be prompted to enter your user password. This is not ideal for two reasons: first, it's annoying; second, it's only as secure as your password is. 

To solve both of these problems, you can use highly secure ssh key pairs to authenticate your ssh request. 

To do this, ensure that you have a ssh keypair on your home computer by running 1 `$ ssh-add -l`. 

If the output of this command spits out a key identifier and path, then you're good to go. Otherwise, generate a keypair using the following commands:

	$ cd ~ # go to the home directory
	$ mkdir ~/.ssh # make a hidden folder called ssh
	$ chmod 700 ~/.ssh/ # give appropriate permissions so only you have acccess
	$ ssh-keygen -t rsa # make your key
	
	$ ssh-add -l ~/.ssh/myKey # tell your system to use this key. Don't give the path to the .pub version. It's likely called id_rsa. 
	
Now you have a pair of public and private keys on your machine. The next step is to add your public key to the list of authorized keys in your home directory on the server. We can do this with the following command:

	$ ssh-copy-id bilbo@your.ip
	
Test it out. `ssh bilbo@your.ip` You should not be prompted for a password because the server authenticated your identity by verifying your associated ssh key was on the authorized keys list. 

To see your authorized key on the server you can check the content of the authorized_keys file in your .ssh directory:
	
	$ cat ~/.ssh/authorized_keys

##### Github
First, install git

	sudo apt-get install git

Then config with your username and password. These are the names that get associated with your commits. 

	git config --global user.name "Your Name Here"
	git config --global user.email "your_email@example.com"
	
If you start using git at this point, every time you try to do a push, pull, clone etc to Github, you will be prompted for your username and password. That's annoying! 

To avoid having to do this, we can use the same process we did to authorize your home computer, but this time we will authorize your server with github. 

On the server, do the following to generate a ssh keypair. 

	$ cd ~ # go to the home directory
	$ mkdir ~/.ssh # make a hidden folder called ssh
	$ chmod 700 ~/.ssh/ # give appropriate permissions so only you have acccess
	$ ssh-keygen -t rsa # make your key
	
	$ ssh-add -l ~/.ssh/myKey # tell your system to use this key. Don't give the path to the .pub version. It's likely called id_rsa. 
	
If you run into an error like `Could not open a connection to your authentication agent`, then add the following command to your `~/.profile`

	# SSH agent
	SSH_ENV=$HOME/.ssh/environment
	
	function start_agent {
	     echo "Initialising new SSH agent..."
	     /usr/bin/ssh-agent | sed 's/^echo/#echo/' > ${SSH_ENV}
	     echo succeeded
	     chmod 600 ${SSH_ENV}
	     . ${SSH_ENV} > /dev/null
	     /usr/bin/ssh-add;
	}
	
	# Source SSH settings, if applicable

	if [ -f "${SSH_ENV}" ]; then
	     . ${SSH_ENV} > /dev/null
	     #ps ${SSH_AGENT_PID} doesn't work under cywgin
	     ps -ef | grep ${SSH_AGENT_PID} | grep ssh-agent$ > /dev/null || {
            start_agent;
	     }
	else
	     start_agent;
	fi

> credit: [ Joseph Reagle Jr.](http://www.cygwin.com/ml/cygwin/2001-06/msg00537.html) 	

This command starts your ssh-agent if it's not running already. 

##### Install and configure apache

If you plan on using apache to serve your projects, it has to be configured and installed. 

	$ sudo apt-get install apache2 apache2.2-common apache2-mpm-prefork apache2-utils libexpat1

Then we can install mod_wsgi, which allows you to serve python based applications from Apache. 

	$ sudo apt-get install libapache2-mod-wsgi
	
Then restart apache, and remember this is basically the "reset and get your shit together" command for apache, so you'll use it a lot in the future. 

	$ sudo service apache2 restart	

##### VirtualEnv

For best practices sake (and to save yourself a huge headache if you need to manage python package dependencies for multiple projects), do yourself a favor and install [virtualenv and virtualenvwrapper](http://virtualenvwrapper.readthedocs.org/en/latest/). This will also be used for the virtualhost to actually serve the Django projects. VirtualEnv makes a seperate changes your python path for a session, so you can have multiple environments on one computer without conflicting versions and dependencies. 

	$ sudo pip install virtualenv
	$ sudo pip install virtualenvwrapper

Then add the following to your `.bashrc` or `.profile`. 

	# virtualenv
	export WORKON_HOME=$HOME/.virtualenvs
	export PROJECT_HOME=$HOME/Devel
	source /usr/local/bin/virtualenvwrapper.sh
	
Now you can type 
	
	$ workon # list all of your virtual environments. 
	$ mkvirtualenv DevelopmentEnv # make a new environment
	$ workon DevelopmentEnv # point your python path to this new environment
	
##### Install Dependencies

Go to your project folder

	$ cd ~./mysite
	
Make sure you have an environment and workon it
	
	$ workon myprojectenv
	
Then install your dependencies from your requirements.txt file. If you don't have one, you can make one with `$ pip freeze > requirements.txt` on your home machine where you did local development. 

	$ pip install -r requirements.txt 
	


##### Mysql
Lastl, but certainly not least! Install the database.

	$ sudo apt-get install mysql-server
	$ sudo apt-get install libmysqlclient-dev
	
	$ workon YourEnvironment
	$ pip install mysql-python
	
Boom! Done. 

-----

## Step 2. Get your project on the server

Once you have a working copy of your project on your local machine, you have to actually get the files up on the server. There are lots of ways to do this, but I recommend one of the following: 

##### Github
In my opinion, the best way to get your project on the server is use Github. This obviously involves setting up the repo, so it is a little more involved, but totally worth it in terms of ROI for the future. 

	$ git clone git@github.com:BlahBlah

##### Command Line

Another option is to use the command line interface for securely transfering files to a remote machine. The following command will copy the folder at /path/to/project to the home directory of 'user' (shortcut is ~) directory on the server. Make sure to include the ~ or another path on the server. 

	$ scp -r user@your_server_up:~ /local/path/to/project/

##### Cyber Duck or Filezilla
The other option is to use a program like Cyber Duck or FileZilla to copy files over. Download the client, connect to your server with your ssh key or password, and transfer the files. 

-----


## Step 3. Prepare the project for production
In this step, we make a symbolic link for a folder in /var/www/ to the project in your home directory. This is the location that Apache looks to serve files from. This is a crucial step and it's very important to get all of the permissions and file ownership correct. 

##### Symlink the project

	$ cd /var/www/
	$ sudo mkdir mysite
	$  ln -s ~/mysite/* /var/www/mysite # this makes a link between all of the items (*) inside ~/mysite to /var/www/mysite

		
##### Ownership
Here we need to transfer ownership of the files and folders with the Django proejct to root and www-data. Root in the command below designates the user who owns the files and www-data is the group. 

	$ cd /var/www
	$ sudo chown -R root:www-data mysite/
	
##### Permissions

Setup the permissions with 775 like this:

	sudo chmod -R 775 test/

##### Setup the Django database interface (MySQL)
First, in Settings.py you need the databases setting:

	DATABASES = {
	  'default': {
	    'ENGINE':'django.db.backends.mysql',
	    'NAME': 'olr',
	    'USER': 'olrServer',
	    'PASSWORD': 'hnine5tree',
	  }
	}
	
Then, you need to make this user on the MySQL server on your server. Here we login to mysql, create the database for the project, create a user for the project, and give that user permissions. 

	$ mysql -u root -p
	
	>> create database [db name];
	>> CREATE USER 'newuser'@'localhost' IDENTIFIED BY 'password’;
	>> GRANT ALL PRIVILEGES ON [db name] . * TO 'newuser'@'localhost’; 
	>> -- (* refers to database and and table respectively)
	>> FLUSH PRIVILEGES;
	
Ensure that any time you modify the permissions for any user, you use the `FLUSH PRIVILEGES;` command. 
	
##### Static Files
As we all know, serving static files with Django can be a total pain sometimes. On a server, it is even more painful. We basically have two options here:

One: Host all of your static files on something like Amazon S3 and avoid having them on the server. 

Two: Use the `python manage.py collectstatic` command to put all your static files for your apps in one place.  This is usually mysite/static/. See [here](https://docs.djangoproject.com/en/1.6/howto/static-files/) for more info. 

-----

## Serving Your Django Projects
In this step we need to tell Apache to serve the project using a virtualhost and wsgi file. 

##### Virtualhost
To actually serve the projet, Apache needs to have a virtualhost. First, go to that directory on the filesystme:
	
	$ cd /etc/apache2/sites-available
	
Then, make your virtual host:

	$ sudo vi mysite
	
And make it look like this, but with your correct info: 

	<VirtualHost *:80>
		ServerAdmin youremail@gmail.com
		ServerName yourdomain.com
		ServerAlias your.ip.address ( or put your full domain here if you have one setup i.e. www.domain.com)
		WSGIScriptAlias / var/www/mysite/index.wsgi
	
		Alias /static/ /var/www/mysite/static/
		<Location "/static/">
			Options -Indexes
		</Location>
	</VirtualHost>

This essentially tells Apache to load the .wsgi file for this domain name for this project. 

##### WSGI File

The final step is to configure this application wsgi file. 

	import os
	import sys
	import site
	
	# Add the site-packages of the chosen virtualenv to work with
	site.addsitedir('~/.virtualenvs/mysiteEnvironment/local/lib/python2.7/site-packages')
	
	# Add the app's directory to the PYTHONPATH

	# first append the project folder path
	sys.path.append('/var/www/mysite')

	# second append the location of the app settings.py file note the case sensitive path. 
	sys.path.append('/var/www/mysite/Mysite')
	
	# Note that Mysite.settings corresponds with the folder name of the settings.py file. 
	os.environ['DJANGO_SETTINGS_MODULE'] = 'Mysite.settings'
	
	# Activate your virtual env
	activate_env=os.path.expanduser("/home/user/.virtualenvs/mysiteEnv/bin/activate_this.py")
	execfile(activate_env, dict(__file__=activate_env))

	import django.core.handlers.wsgi
	application = django.core.handlers.wsgi.WSGIHandler()
	
	
	
	
And now your done! 

## Conclusion

To conclude, this is the process I used to get a server up and running to serve my Django apps. Please let me know if I missed something or if you have any questions: [email](mailto:desmondrduggan@gmail.com). 
	

