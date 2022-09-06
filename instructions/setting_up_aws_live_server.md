# Setting up the Live Server on AWS
Your Production Environments (Live Servers) will be hosted on Amazon Web Services(AWS) a popular online cloud hosting platform. 

## Using AWS Academy
1. You should have already received an email to join an AWS Academy classroom. If you have not received an email reach out to an IT210 TA. 
2. Follow the instructions in the [AWS Academy Learner Lab Student Guide PDF](AWS-Academy-Learner-Lab.pdf)

> Note: If you do not have a Canvas account you will need to create one. Your BYU Canvas account will not work. 

## Setting Up Your EC2 Instance
1. On the AWS Management Console, go to the top left `Services` drop down and select `Compute` > `EC2`.
2. On the left-hand sidebar under `Instances`, select `Instances`. This dashboard is where you can start, stop, and monitor your live server. 
3. Click the orange `Launch instances` button in the top right corner. 
4. Staying on the `Quick Start` tab, scroll down and find `Ubuntu Server 20.XX LTS (HVM), SSD Volume Type` and click the blue `Select` button.
5. For the Instance Type choose `t2.micro`. Don't edit any configurations for steps 3-5 (Configure Instance, Add Storage, Add Tags)
6. On `Step 6: Configure Security Group`, click `Add Rule` and change the rule type to `HTTP`. Click the blue `Review and Launch` button.
    > Note: What we are doing here is adding a rule to the default AWS firewall to allow HTTP requests to get to our EC2 instance. Without this we would not be able to see our website because our requests would be blocked.  
7. Click the blue `Launch` button.
8. After clicking `Launch`, a pop-up will appear with information about SSH keys. **SSH** means **S**ecure **Sh**ell, which is a way to access a remote server via any terminal/command prompt as long as you have the key. An SSH key is a simple unique text document that you will store on your computer to access your EC2 instance. Treat your ssh key like a password. Do not lose your key file, do not publish your key file online anywhere. 
9. In the first dropdown select `Create a new key pair`. Name your key something useful like `IT210key`. Click `Download Key Pair` and store the key file somewhere safe on your computer. 
10. After you have downloaded your key `.pem` file, click `Launch Instances`.
11. From the `Launch Status` page you can see your instance by clicking `View Instances` in the bottom right corner. 

## Connect to Your Instance
###  Windows Users Only
In your Linux terminal you need to navigate to where you downloaded your .pem file to, this command assumes that it is in the downloads folder, if you downloaded it elsewhere adjust the last part of this command accordingly. Make sure to replace <username> with the user account name you are using.

```sh
cd /mnt/c/Users/<username>/Downloads
```

Move the SSH key from the location it was saved in the Windows file system to the Linux file system.

Open a Ubuntu terminal and run the following command. This assumes that you have downloaded the key to your downloads folder. If this is not the case, modify the file path to match where your key is saved.
```sh
cd /mnt/c/Users/<your windows user name here>/Downloads
```
```sh
mv <my ssh key file>.pem ~/mv <my ssh key file>.pem
```
```sh
cd ~
```
### Windows, Linux & Mac Users

```sh
sudo chmod 400 <my ssh key file>.pem
```

1. On the `Instances` page, find your running instance in the table and right-click on it. In that pop-up menu, click `Connect`. Find the `SSH client` tab and follow those instructions on your computer's terminal to SSH into your AWS machine using the SSH key you downloaded.
 > Note: Windows users need to run the SSH command from the Linux command line.
2. Once you are connected to the server, you should be at a prompt that looks something like this:

    ```
    ubuntu@ip-<SOME IP ADDRESS>:~$ 
    ```
## Install Apache2
To serve your website we will need to install apache2.
1. While in your SSH AWS prompt, run these two commands:
    ```
    sudo apt update -y && sudo apt upgrade -y
    ```
    ```
    sudo apt install apache2
    ```
2. You can control your server's state with `sudo service apache2 <option>`, where `<option>` is one of `{start|stop|graceful-stop|restart|reload|force-reload}`. The usages of these are fairly easy to know just by the name.
3. Run these two commands to start apache2 and make sure it is running:
    ```
    sudo service apache2 start
    ```
    ```
    sudo service apache2 status
    ```
4. You can now see if apache is working by going back to your AWS Instances page selecting your instance and copying and pasting either the `Public IPv4 address` or the `Public IPv4 DNS` into a browser. The default Apache2 page has a red banner across the top that says "It Works!". 
    > Note: Make sure to visit your site using HTTP not HTTPS because you do not have HTTPS set up.

## Clone your code

1. On GitHub.com, look at your repo and click the button that says "Clone or download" and then copy the URL in the box.

2. Navigate to the Apache WebRoot:

    ```sh
    cd /var/www
    ```
    
3. Make yourself the owner of this directory:

    ```sh
    sudo chown -R $USER . && sudo chgrp -R $USER .
    ```
    Note:  The next three instructions will be used in most of the labs to update the live server to the new website. Make note of them or remember where to find them.
4. Delete the current `html` folder that's found there:

    ```sh
    rm -rf html
    ```
    > Note: Be VERY careful using this. You can break your server if you `rm` the wrong file.

5. Clone your repo:

    ```sh
    git clone <the URL you copied from github>
    ```

6. Create a "Symbolic Link" called `html` that points to the `src` folder in the repo that you cloned:

    ```sh
    ln -s <folder of your repo>/src html
    ```
    > Note: Remove the < and > symbols when replacing the folder name. This applies to anytime < and > are used.  
    
    > Explanation: This creates a "shortcut" to your src folder. When Apache tries to access `/var/www/html`, it will be
    pointed to the `src` folder in your project now.

    __Remember this command!__ In future labs, you will need to clone more repos to your live server and point apache to the
    right directory. This involves `rm`ing the old symbolic link and making a new one using this command. This is so you
    don't have to make another `sites-available` file (which we'll cover next) and make Apache reload.
    
## Change Default Config
Apache comes with a default configuration file, and as of the time of writing this, it should be called `000-default.conf`.
You'll create your own new site config.

1. Navigate to the `sites-available` directory for Apache:

    ```sh
    cd /etc/apache2/sites-available
    ```

2. Copy the default config into a new config:

    ```sh
    sudo cp 000-default.conf it210_lab.conf
    ```

3. Learn how to open your file with a text editor
    To open your file in an editor as root, using
    ```sh
    sudoedit it210_lab.conf
    ```
    The default editor is `nano`, but `vim`, `emacs`, or `ed` are also standard options. You can change which editor `sudoedit` uses by installing the editor and then running this command, following its prompts:
    ```sh
    # Optional
    sudo update-alternatives --config editor
    ```

4. Disable the directory listing in `/etc/apache2/sites-available/it210_lab.conf` or `/etc/apache2/apache2.conf` (you'll need to search the web to find out how to do this).

    > Note: Directory listing means when you navigate to a directory in the browser without an index file
    (like the `src/css` folder, i.e. `192.168.90.<your ip>/css/`, which contains only `.css` files) it will _list_ the
    contents of the _directory_. We don't want this for your servers.

   > Be careful when editing `conf` files. Making incorrect changes could break apache and stop it from running. Make a copy of any `conf` files before you make changes so there is a backup in case something does go wrong. 

5. Make sure a user can view your website without having to specify the `index.html` file in the URL.

6. Save the new document and then run the following commands to disable the default site and enable your new one:

    ```sh
    sudo a2dissite 000-default.conf
    sudo a2ensite it210_lab.conf
    ```

7. It says you need to reload Apache, so do it:

    ```sh
    sudo service apache2 reload
    ```
## Test it out

You should be able to access your `src/index.html` file by typing in your AWS EC2 instance IP address in any browser. Typing `http://<your ip>/css` should say **Forbidden**, and `http://<your ip>/does_not_exist`
should say **Not Found**.

## Shut Down Instance
When you are not actively using your EC2 instance you need to shut it down or you will use up all your credits before the end of the semester. 
To shut your instance down from the `Instances` page, right-click your instance in the table and select `stop instance`. 

> DO NOT select `terminate instance`, this will DELETE it, not turn it off

> Note: By default, when you restart your instance to work on it again it will have a new IP address. 
