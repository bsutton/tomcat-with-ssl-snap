# tomcat-with-ssl snap part

Tomcat-with-ssl is a remote part that you can use in your own snap to deploy an Apache Tomcat server, your web app and an SSL certificate.

This remote part takes what is perhaps annd an SSL certificate.

 unusual path, in that it has Tomcat expose port 443 directly rather than requiring a web server such as Apache.

The part is designed to facilitate deploying a SINGLE webapp under Tomcat.

This part is intended to included in your own snap using the 'after:' clause;

The end result is a secure web server that takes two lines to install and configure:

sudo snap install yourwebapp
sudo yourwebapp.getcert <youremail> <fqdn>

You can start/stop the tomcat service via:
sudo snap start yourwebapp
sudo snap stop yourwebapp

For full documentation see:

https://forum.snapcraft.io/t/remote-part-tomcat-with-ssl/4264

The default build is based on tomcat 8.5 but you can easily modify the version.

Note: snaps MUST be built on ubuntu 16.04 (or using lxd with a 16.04 image).

# Note: currently if a certificate renewal occurs tomcat will be restarted without warning.

# Credit where credit due
I based my work on the snap part created by:
Michael Hall <mhall119@ubuntu.com>
https://github.com/mhall119/tomcat-snap-part.git

#Lets Encrypt
The part uses ‘certbot’ from Lets Encrypt to obtain a certificate and renew that certificate as required.

One current weakness with this implementation is that it checks for an expiring certificate every 15 minutes. When your certificate needs to be renewed, it will renew it immediately AND RESTART Tomcat. This is likely to be problematic for a serious webapp as random restarts are usually not welcome. I will look at improving this in a later release.

# How to use this part
You would normally use this snap part from within your own snap 

    name: orion-monitor # you probably want to 'snapcraft register <name>'
    version: '0.1' 
    summary: A java web app.
    description: |
      My web app does good things.
      
    grade: devel # must be 'stable' to release into candidate/stable channels
    confinement: devmode # use 'strict' once you have the right plugs and slots
  
    apps:
      tomcat:
        command: tomcat-launch
        daemon: simple
        plugs: [network, network-bind]
        
         # used to ran the certbot renewal process.
      cron:
        command: cron
        daemon: simple
        plugs: [network, network-bind]

      # Used to obtain a certificate post install
      getcert:
        command: getcert
        plugs: [network, network-bind]
  
    parts:
      my-webapp:
        plugin: maven  # assuming you use maven to build your java code.
        source: git@github.org:yourusername/yourwebapp.git
        maven-options:
          [-DskipTests=true]
        organize:
          # rename the generated war so its available from the / rather than requiring a context name
          war/irrigation-1.0-SNAPSHOT.war : webapps/ROOT.war
        after: [tomcat-with-ssl]


# installing
To install your snap
    
    sudo snap install <your snap>
  
Tomcat will start on port 8080. 

# obtain a certificate

You now need to obtain a certificate:
Run:

    <yoursnap>.getcert <your email address> <fqdn> true
  
True is the default for the final parameter so can be left off.

# use a staging certificate during testing
Running getcert obtains a Lets Encrypt certificate.
By default it will get a live production certificate.

During testing of your app installation process you may want to run with a Lets Encrypt 'Staging' certificate
as Lets Encrypt has a daily request limit on live certificates (of around five).

To use a staging certificate add 'false' to the end of the line:

    <yoursnap>.getcert <your email address> <fqdn> false
  
# Certbot
getcert uses Lets Encrypts cert bot to obtain the certificates:

Certbot logs to: 

    /root/snap/<yoursnap>/current/letsencrypt/logs/letsencrypt.log
  
# certificate renewal
The snap installs its own 'cron' process which runs every fifteen minutes to check for an expired certificate.

If the renewal cron detects an expiry certificate it will automatically renew it.
When it renews the certificate it will restart tomcat so it picks up the new certificate.

Currently this process can trigger at ANY time during the day!

The cron process logs to:

    /root/snap/<yoursnap>/current/cron/renew.log
  
# Tomcat

You can start/stop/restart tomcat via:

    sudo snap start|stop|restart <yoursnap>.tomcat
  
## Tomcat Logs

Tomcat logs are contained within the snap and can be found at:

    /var/snap/<yoursnap>/current/logs



touched.
