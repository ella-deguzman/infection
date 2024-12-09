# infection

1\. Platform Specific Setup 
============================

1.1 AWS Setup
-------------

Follow the separate AWS [setup guide](https://github.com/bioe131/lab-8-ella-deguzman/blob/main/aws_instructions.md), then return here to set up linuxbrew below.

1.2 Linuxbrew for AWS
---------------------

Make sure you are using a Debian or Ubuntu distribution. Then go ahead and install linuxbrew, using the instructions below:

switch to root with:

`sudo su -`
Then run:

`passwd ubuntu`
It is going to prompt :

`Enter new UNIX password:`

Set your password to something you can remember for later, or write down. A common password choice is simply ubuntu - not very secure at all, but AWS accounts themselves can be made fairly secure.

Exit root by typing `exit`. Note: it is important to exit root, because you do not want to accidentally run future commands with administrator privileges when that might be undesirable. The subsequent command in this case will fail if run from root.

Install brew using the bash script from <https://brew.sh/>. You will be prompted to set the password you made earlier.

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

After this is complete, add brew to your execution path:

```
echo >> /home/ubuntu/.bashrc

echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> /home/ubuntu/.bashrc

eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
```

2\. Install necessary tools
===========================

2.1. Node.js
------------

Node.js is a cross-platform JavaScript runtime environment that will make is easy to run JBrowse2 command-line tools.

First, check whether Node.js is already installed by running the following. If node v20 is already installed, you can skip to the next step.

`node -v`

If Node.js is not installed, install it.

For AWS:
```
sudo apt install unzip
```
Then run 

```
# installs fnm (Fast Node Manager)
curl -fsSL https://fnm.vercel.app/install | bash
# activate fnm
source ~/.bashrc
# download and install Node.js
fnm use --install-if-missing 20
# verifies the right Node.js version is in the environment
node -v # should print `v20.18.0`
# verifies the right npm version is in the environment
npm -v # should print `10.8.2`
```

2.2. @jbrowse/cli
-----------------

Run the following command in your shell.
```
npm install -g @jbrowse/cli
```

2.3. System dependencies
------------------------

Install wget (if not already installed), apache2, samtools, and tabix.

wget is a tool for retrieving files over widely-used Internet protocols like HTTP and FTP.

apache2 allows you to run a web server on your machine.

samtools and tabix, as we have learned earlier in the course, are tools for processing and indexing genome and genome annotation files.
```
sudo apt install wget apache2
brew install samtools htslib
```

3\. Apache server setup
=======================

3.1. Start the apache2 server
-----------------------------
```
sudo service apache2 start
```
3.2. Getting the host
---------------------

In your instance summary page, there should be an "auto-assigned IP address." Your web server can be accessed at `http://ipaddress`. You don't need to provide a port.

3.3. Access the web server
--------------------------

Open a browser and type the appropriate url into the address bar. You should then get to a page that says "It works!" If you have trouble accessing the server, you can try checking your firewall settings and disabling any VPNs or proxies to make sure traffic to localhost is allowed.

3.4. Verify apache2 server folder
---------------------------------

Apache2 web servers serve files from within a root directory. This is configurable in the httpd.conf configuration file, but you shouldn't have to change it (in fact, changing the conf file is not recommended unless you know what you are doing).

For a normal linux installation, the folder should be /var/www or /var/www/html, whereas when you install on macOS using brew it will likely be in /opt/homebrew/var/www (for M1) or /usr/local/var/www (for Intel). You can run brew --prefix to get the brew install location, and then from there it is in the var/www folder.

Verify that one of these folders exists (it should currently be empty, except possibly for an index file, but we will now populate it with JBrowse 2). If you have e.g. a www folder with no www/html folder, and your web server is showing the "It works!" message, you can assume that the www one is the root directory.

Take note of what the folder is, and use the command below to store it as a command-line variable. We can reference this variable in the rest of our code, to save on typing. You will need to re-run the export if you restart your terminal session!

```
# be sure to replace the path with your actual true path!
# for example we used export APACHE_ROOT='/var/www/html'
export APACHE_ROOT='/path/to/rootdir'
```

3.5. Download JBrowse 2
-----------------------

First create a finalproj directory as a staging area. 
```
mkdir finalproj
cd finalproj
```

Next, download and copy over JBrowse 2 into the apache2 root dir, setting the owner to the current user with chown and printing out the version number. This version doesn't have to match the command-line jbrowse version, but it should be a version that makes sense.

```
jbrowse create output_folder
sudo mv output_folder $APACHE_ROOT/jbrowse2
sudo chown -R $(whoami) $APACHE_ROOT/jbrowse2
```

3.6. Test your jbrowse install
------------------------------

In your browser, now type in `http://yourhost/jbrowse2/`, where yourhost is either localhost or the IP address from earlier. Now you should see the words "It worked!" with a green box underneath saying "JBrowse 2 is installed." with some additional details.'
