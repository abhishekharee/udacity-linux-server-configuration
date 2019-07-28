# Project: Linux Server Confugration

This README documents the process of deploying the Podcast Web App stored in [this](https://github.com/abhishekharee/udacity-item-catalog) repository to a Linux Virtual Private Server ("VPS"), and securing that server against bad actors.

## Creating VPS

In order to make the web app accessible on the internet, a server, accessible via an IP address, is needed to 'serve' the web app to users on the internet. Rather than setting up a physical server, a _virtual_ server can be set up instead. VPS services are provided by various **cloud provider**s. This README documents using a VPS supplied by **AWS Lightsail** ("AWS"). If an alterantive VPS provider is being used, ensure that it is set up as an Ubuntu 16.04 LTS server, and skip ahead to the next section. Else, follow the following steps:

1. Sign up for an AWS account [here](https://portal.aws.amazon.com/billing/signup?nc2=h_ct&src=header_signup&redirect_url=https%3A%2F%2Faws.amazon.com%2Fregistration-confirmation#/start).

2. After logging into the AWS portal, click the 'Build using virtual servers With Lightsail' link to access the Amazon Lightsail portal.

3. Click 'Create Instance'. Then, select 'Linux/Unix', and ensure to select the 'OS Only' option. Select 'Ubuntu 16.04 LTS'. Select the cheapest tier, provide a name for the server, and hit 'Create instance'. After a few seconds, this new instance should be up and running.

4. Click the server text in order to access certain server settings. Then, click 'Networking'. Finally, click 'Create static IP', and 'attach' it to the server. This will ensure that the IP address of the VPS (and therefore, web app) will remain fixed, improving its ability to be found on the internet.

## Logging into the VPS

Configuring and deploying the VPS will require SSH-ing into it from a client machine. The following steps describe how to SSH into an AWS VPS, assuming the above steps have been completed. If the ability to SSH into an Ubuntu VPS has already been established, skip ahead to the next section. Else, follow the following steps:

1. After logging into AWS, click 'Account'. Then, click 'SSH keys' and download the private key to your local machine.

2. Open up a command prompt and change directory to the location of where the file has been downloaded. Then, run:
    
    `chmod 644 <AWS key>`

    Where '<AWS key>' should be replaced with the filename of the key that was downloaded.

3. Finally, in the same directory, run

    `ssh ubuntu@<IP address> -p 22 -i <AWS key>`

    Where '<IP address>' should be replaced with the public IP address of the AWS VPS, and '<AWS key>' should be replaced with the same key described in the above step.

    _Note: the -p 22 in the above command ensures the client connects to the VPS using port 22, and the -i flag ensures the SSH application uses the .pem file as credentials._

Once logged into the VPS, run:

`sudo apt-get update && sudo apt-get upgrade`

This will update the package lists on the AWS VPS, and upgrade any applications that have been updated by package-maintainers. Respond to any prompts that come out of this process, accordingly.

## Creating a new user - specific to the Linux Server Configuration Project

For the purposes of the Udacity Nanodegree, a new super user, 'grader', needs to be created. The following steps describe the process of setting up this super user, and can be repurosed to create any other user, whether a super user or not. If this super user is not necessary for various reasons, skip ahead to the next section. Else, follow the following steps:

1. In the VPS terminal, run

    `sudo adduser grader`
    
    A password for 'grader' will be requested. Provide one, and proceed to filling out the various prompts for metadata with the relevant information. _(Note: This is the only step needed to create a new user. If a user needs to be given super user privileges, proceed with steps 2 and 3 below.)_

2. When the user is created, run

    `sudo vim /etc/sudoers.d/grader`
    
    This will open up a vim application, where permissions will be entered to enable sudo privileges for the 'grader' user.

3. Type 'i' to enter into 'INSERT' mode, so edits can be made to the file. Then, write:

    `grader ALL=(ALL) NOPASSWD`
    
    Finally, hit the 'Esc' key, and type `:wq` and hit the 'Enter' key. This will save the file in the relevant directory, and allow the VPS to identify the grader user as a 'super user'.

## Securing Linux server configuration and enabling firewall

Now that the server has been set up, it must be secured. Specifically, all ports on the server need to be closed except for port 2200 (for SSH), port 80 (for HTTP), and port 123 (for NTS). This will restrict traffic and entry points into the server and go some way in preventing bad actors from disrupting the web app or its host server. This README covers achieving this goal using In oreder to do this, perform the following steps:

1. While being logged in as a user with super user privileges, run:

    `sudo ufw deny all incoming`
    
    This will ensure the ufw application ('the firewall') will, when active, block all incoming traffic to the server.

2. Then, run:

    `sudo ufw allow all outgoing`
    
    This will ensure the firewall will, when active, allow the server to send traffic out of any port.

3. Then, run:

    `sudo ufw allow 2200/tcp`
    
    This will ensure TCP/IP connections will be permitted into port 2200. _(Note: This README proceeds on the basis that this will be the SSH port. Usually, the SSH port is set up on port 22.)_

4. Next, run:

    `sudo ufw allow http`
    
    This will ensure that HTTP connections can be made with the server. This will open up port 80 - the standard port number for HTTP connections.

5. Next, run:

    `sudo ufw allow ntp`
    
    This will ensure NTP connections can be made with te server. This will open up port 123 - the standard port number for NTP connections.

6. **OPTIONAL BUT VERY IMPORTANT STEP** AWS has its own firewall settings, modifiable from the AWS portal. These port settings supersede those that will be set by the ufw application. If these settings aren't modified to mirror the settings described in steps 1 through 5 above, there is a high risk that _all_ SSH connections into the VPS will be blocked. Therefore, navigate to the 'Networking' tab of the AWS portal specific to the VPSm and add/delete port settings so that TCP/IP connections are enabled for port 2200 and 80, and UDP comnnections are enabled for port 123. As a precaution, do all of this while you are still logged into the VPS via SSH. _(Note: this step is likely necessary for other VPS providers.)_

7. Back in the SSH terminal, run:

    `sudo vim /etc/ssh/sshd_config`
    
    This will open up a vim application that will allow edits to the SSH configuration. For a VPS set up on AWS Lightsail, there will be a line in this file labelled 'Port 22'. Move the cursor to this line, hit 'i' to enter into INSERT mode, and then edit this line to read to 'Port 2200'. Then, hit the 'Esc' key, type ':wq', and then hit the 'Enter' key. These steps ensure the VPS is configured to only permit SSH connections on port 2200.

8. Next, run:

    `sudo service ssh restart`
    
    This will activate the edited SSH configuration saved in the immediately-preceding step.

9. Finally, run:

    `sudo ufw enable`

    This activates the ufw application, and raises a firewall that will go some way to protect the VPS against bad actors. The specification of the ufw firewall can be checked by running:

    `sudo ufw status`

## Setting up VPS to run FlaskApp

The 'Podcast Web App' depends on a number of libraries and packages that are not available by default on a newly-created Ubuntu 16 instance. These libraries and packages will therefore need to be installed.

1. Run:

    ```
    sudo apt-get install flask
    sudo apt-get install Flask
    ```
    
    These commands will install Flask on the VPS.

2. Run:

    ```
    sudo apt-get install python
    sudo apt-get install python-pip
    ```

    These commands will install python 2.7 (the language in which the Podcast Web App is written), and an installer that will enable the installation of various libraries used by the Podcast Web App.

3. Run:

    `sudo apt-get install sqlite3 libsqlite3-dev`

    This command will install SQLite - the database engine that the Podcast Web App uses to manage data.

4. Run:

    `sudo pip install oauth2client`

    This command will install the libraries needed to authenticate users using OAuth 2.0 protocols. In the case of the Podcast Web App, this will enable Google as an OAuth 2.0 provider.

5. Run:

    ```
    sudo pip install httplib2
    sudo pip install requests
    ```
    
    These commands will install various packages used by the Podcast Web App in order to function as a web server.

6. Run:

    `sudo pip install sqlalchemy`

    This command will install the SQL Alchemy package, which allows movmeent of data between object oriented languages (in this case, Python) and a database system (in this case, SQLite).

7. Run:

    ```
    sudo apt-get install apache2
    sudo apt-get install sudo apt-get install libapache2-mod-wsgi python-dev
    ```

    These commands will install Apache, which will be the application that actually runs the Flask App (the Podcast Web App), and the relevnat packages needed to run the WSGI module.

## Setting up for Flask, and porting over Podcast Web App code

Assuming the above steps have been completed, the VPS should be ready to host a Flask application. Complete the following steps:

1. Run:

    `sudo apt-get install git`

    This will install git on the VPS (although it is likely pre-installed when creating the Ubuntu VPS instance).

2. Change directory to `/var/www/`, and run:

    `sudo mkdir /var/www/FlaskApp/`

   This will create the directory where the files supporting the Podcast Web App will be stored.

3. Change directory to `/var/www/FlaskApp/`, and run:

    `sudo git clone https://github.com/abhishekharee/udacity-item-catalog.git FlaskApp`

    This will clone the git directory that stores the files needed to run the Podcast Web App, and store those files in a FlaskApp folder. The contents of the git directory will now be in `/var/www/FlaskApp/FlaskApp`.

4. Run:

    `sudo rm -R /var/www/FlaskApp/FlaskApp/.git`

    This will delete the link beween the server web app and the git repository from which files were cloned.

5. Run:

    ```
    sudo python podcast_database_setup.py
    sudo python podcast_database_load.py
    ```

    This will initialise the database used to drive the Podcast Web App and seed it with 'starter' information. If executed correctly, the contents of the database will be printed on the screen, and a new file, `podcastepisodes.db` will have been created in the `/var/www/FlaskApp/FlaskApp/` directory.

## Amending Podcast Web App code for serving on VPS

The Podcast Web App, as stored on the git repository from which files were cloned, was designed to run on a local machine. Some changes are needed to enable this app to run on the VPS. The following steps presume a working knowledge of vim.

1.  The Podcast Web App uses Google's OAuth2.0 API to authenticate users. However, the `client_secret.json` file stored in the git repository was set up for use on a local machine only. To enable OAuth2.0 on the Web App _as served on the VPS_, the following steps were performed:

    - Signed into the [Google Developers Console](https://console.developers.google.com).
    - Clicked 'Credentials', then clicked the  pencil icon next to the 'Podcast Web App' entry. Then:
        - Added the public IP address of the VPS to 'Authroized JavaScript origins'
        - Added the public IP address followed by `.xip.io` to 'Authorized redirect URIs' (as Google does not allow naked IP addresses to be added)
        - Clicked 'Save'
    - Clicked 'OAuth consent screen', then:
        - Added the `xip.io` address to 'Authorized  domains' (again, Google does not allow naked IP addresses to be added)
    - Downloaded updated JSON file.

2. Run:
    
    `sudo vim /var/www/FlaskApp/FlaskApp/podcast_webapp.py`

    Change line 9 from `from podcast_database_setup import Base, User, Podcast, Episode` to:

    `from FlaskApp.podcast_database_setup import Base, User, Podcast, Episode`

    This will enable the Podcast Web App to source relevant files from their location on the VPS, rather than from some default, local location.

    Change line 25 from `open('client_secret.json', 'r').read())['web']['client_id']` to:

    `open('/var/www/FlaskApp/FlaskApp/client_secret.json', 'r').read())['web']['client_id']`

    This will ensure the Podcast Web App, when run, will look for the `client_secret.json` file as stored on the VPS. While the client-ID should not have changed as a result of step 1 above, it is worth double-checking that this has not changed.

    Change line 28 from  `engine = create_engine('sqlite:///podcastepisodes.db')`: to:

    ```
   engine = create_engine(
    'sqlite:////var/www/FlaskApp/FlaskApp/podcastepisodes.db',
    connect_args={'check_same_thread': False})
    ```

    This will enable the Podcast Web App to source the database as stored at its location on the VPS.

    Change line 417 from `app.run(host='0.0.0.0', port=5000)` to:

    `app.run()`

    This will ensure the Podcast Web App will run on the server from which it is called (rather than on a local machine, only).

    After the above changes are made, save and exit.


2. Run:

    `sudo vim /var/www/FlaskApp/FlaskApp/client_secret.json`

    This contains the client secret as configured for the Podcst Web App to be run locally. Delete the contents of this file and replace with the contents of the new JSON file downloaded in the immediately-preceding step.

2. Run:

    `sudo mv /var/www/FlaskApp/FlaskApp/podcast_webapp.py /var/www/FlaskApp/FlaskApp/__init__.py`

    This will rename the `podcast_webapp.py` file so that it can be run by Apache on the VPS, using the WSGI module.

3. Run:

    `sudo vim /var/www/FlaskApp/flaskapp.wsgi`

    Paste the following into this file (in vim):

    ```
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/FlaskApp/")

    from FlaskApp import app as application
    application.secret_key = 'super_secret_key'
    ```

    This will enable the WSGI mod to call the Podcast Web App (i.e. the `__init__.py` file). This WSGI mod will be what Apache 'activates' when serving the App at the IP address of the VPS.

    Save and exit.

4. Run:

    `sudo vim /etc/apache2/sites-available/FlaskApp.conf`

    Paste the following into this file (in vim):

    ```
    <VirtualHost *:80>
                ServerName <IP address>
                ServerAdmin <email>
                ServerAlias <IP address>.xip.io
                WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
                <Directory /var/www/FlaskApp/FlaskApp/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/FlaskApp/FlaskApp/static
                <Directory /var/www/FlaskApp/FlaskApp/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```
    With the '<>' tags above being replaced appropriately.

    This will 'tell' Apache to run the WSGI module defined in step 3 above, when the Apache service is activated.

    Save and exit.

## Deploy Podcast Web App and activate Apache service

The above steps, when properly executed, will have ensured all files are in their proper condition to be served on the internet. The following steps should be executed in order to enable those files to be used as a functioning web app accessible on the internet:

1. Run:

    `sudo a2dissite defualt`

    This command will disable the default Apache 'site', which is a simple HTML page that explains how the Apache application works.

2. Run:
    ```
    sudo a2enmod wsgi 
    sudo a2ensite FlaskApp
    ```

    This command will enable the Flask App (i.e. the Podcast Web App) to run via Apache on the VPS.

3. Run:

    `sudo service apache2 restart`

    This command will restart the Apache service so that it runs the Flask App (i.e. the Podcast Web App) on the VPS' IP address.

## References

- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
- https://github.com/jungleBadger/-nanodegree-linux-server-troubleshoot/blob/master/missing_client_secret/README.md
- https://github.com/jungleBadger/-nanodegree-linux-server-troubleshoot/blob/master/OAuth_login/README.md
- https://stackoverflow.com/questions/14238665/can-a-public-ip-address-be-used-as-google-oauth-redirect-uri
