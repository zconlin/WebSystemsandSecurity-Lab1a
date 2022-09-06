# Lab 1A - Development and Production Environments

### Overview

Before you start building your application, you need an environment on your computer that makes it easy to see your app as you develop it.

With basic HTML, CSS, and JavaScript, you can just open the files in your browser by dragging the HTML page into an open browser window.

However, soon you'll be developing in PHP, C#, and frameworks such as Node.js and React, so you'll want a better solution, because you can't just open those files in a browser.

### Functionality

- Setup development environment
- Setup production environment
- Serve a web page

### Concepts

- Development vs Production Environments
- Configurations with web servers
- Working with Linux servers
- Working with Remote Servers

### Technologies

- Docker
- HTML
- Linux
- Apache

## Step 1: Download VSCode

**Visual Studio Code** is a text editor with _TONS_ of plugins and helpers, including some built-in plugins like [emmet abbreviations](https://docs.emmet.io/abbreviations/) and [Intellisense](https://code.visualstudio.com/docs/editor/intellisense). VSCode is built with HTML and CSS, which makes it incredibly easy to create plugins, which is why there are so many even though it's only been out for a few years.

You can use VSCode for almost any technology. We'll use it for most of our projects.

1. Go to the [VSCode Website](https://code.visualstudio.com/) and download the proper version for your operating system

2. During the install, if you're installing on Windows, make sure to check the following boxes:

    - Add to PATH
    - Add 'open with code' to the context menu
    - Associate file extensions with code

3. Once installed, explore the `Extensions` tab, the boxy looking icon on the left-side dock

4. For instance, look up "CSS Formatter" and install the extension of that name from Martin Aeschlimann

5. Create a folder for your project on your computer, open it in Code, and start hacking!

Another important feature of VSCode is the Git integration. If you open a git repo in VSCode, it will sense that automatically and show any changes in the git tab (the one that looks like a little node tree). There's also a `debug` tab, but you need a debug profile to use it.

Any time you need to find some kind of VSCode command, press <kbd>F1</kbd> **OR** <kbd>Ctrl-Shift-p</kbd>/<kbd>Cmd-Shift-p</kbd> to open the "Command Pallette". Type "Theme" and you'll see options for changing the editor's theme, or the file icon theme. My favorite file icon theme is `vscode-icons` which you can find in the Extensions tab.

## Step 2: Download Docker

**Docker** is a system that allows you to develop with all of the packages you need without actually installing those packages. It's possible to create a Docker Container with everything you need pre-installed, whether it's PHP or Node.js, and use it to develop as if you were developing on a real server.

If you are on a Windows machine you will most likely have to use WSL2 to get Docker to work properly on your computer. Follow [this tutorial](https://www.padok.fr/en/blog/docker-windows-10) to do so. If you have a Mac or Linux machine you do not need to worry about this.

We'll be using Docker to set up our DEVELOPMENT environment.

1. [Follow this link](https://hub.docker.com/signup) and create an account

2. Download Docker Desktop, and update it if necessary, also a reboot might be required

3. Install the [Docker Extension for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker)

> Note: You might want to make sure Docker Desktop doesn't run automatically on boot, as it takes quite a few resources to get started.

Explore the VSCode extension. You might not understand everything that you're looking at, but when you set up your project, it will make more sense. Once the Docker installation is complete, the extension should show that it was successfully connected, but no items found.

Basically, **Containers** are your Virtual Machines, but they don't come with a full operating system. **Images** are what the containers are built on. You can find all kinds of containers on [Docker Hub](http://hub.docker.com). **Registries** are where you can add credentials. **Networks** are virtual adapters for your Containers. **Volumes** are local storage locations for your containers, because when containers aren't running, none of the data inside it persists.

> WARNING: DO NOT INSTALL DOCKER ON YOUR LIVE SERVER. **YOU DON'T NEED IT THERE. IT WILL JUST MAKE YOUR LIFE MISERABLE.**

## Step 2.1: Install git

This step if for Windows users only.

In order for Visual Studio Code to integrate with GitHub, you’ll need to install the git package onto your system:

- [View the download link here](https://git-scm.com/download/win)

> Lab computers might have to install 32-bit exe.

When choosing your default editor, this doesn’t matter, but I chose Use Visual Studio Code as Git’s default editor. This only affects config files, I think, so you won’t be using this option. Install using recommended: Git from the command line and also from 3rd-party software.

[More information about version control](https://code.visualstudio.com/docs/editor/versioncontrol)

If you do not have a GitHub account already, sign up for one [here](https://github.com).

## Step 3: Set up your project

In order to see your project running, you need to _serve_ it. We'll be downloading a static file server called Apache, via a Container that's managed by Docker. (DO NOT actually download Apache. All you need is the Docker Container.) You can use the `httpd` official Apache docker image. Apache is a static file server. Below are instructions for using it.

1. Clone your repository to your computer, and open the folder in VSCode.
> Note: This repository that you are cloning from is also the repository that you will push your code to when you push to Github.

2. Create a new folder called `src` in the project root directory. Inside the `src` folder, create two more folders, called `css` and `js`. 

    - The `src` folder is where your project files will go. Any HTML, CSS, and JavaScript should be here.
    - In the `css` folder create a file called `style.css` in the `js` folder create a file called `script.js`. You do not have to put anything in these files in this lab but in future labs we will use a similar file structure. 
    - Back in the `src` folder, make a file called `index.html` and paste the following into that file:

    ```html
    <!DOCTYPE html>
    <html>
    <head>
      <title><!-- add a good title --></title>
    </head>
    <body>
      <!-- add a main header -->
      <!-- add an unordered list -->
        <!-- add 3 'tasks' to that list -->
    </body>
    </html>
    ```

    > Note: The `<title>` inside the `<head>` is a special element. Browsers will take the content here and stick it on the tab at the top of the browser. We'll be able to see this once we have Docker serving our files.

3. Create a new file called `docker-compose.yml` in the root of the project (i.e. NOT inside the new `src` folder) and paste in the following:

    ```yml
    web:
        image: httpd
        container_name: it210_lab_1_apache
        ports:
            - 80:80
        volumes:
            - ./src:/usr/local/apache2/htdocs
    ```

    > Explanation: `docker-compose` is a tool that allows us to give settings to a docker image before running it.

    > Here, we're telling Docker that it will create a container using the `httpd` image, and call it `it210_2_apache`. It should forward any connections on port `80` of your localhost to port `80` on your container. The first number is the localhost port, and the second number is the port on the container. Apache is already set to listen on port 80, but if you were running multiple servers, you could see them all by associating them with different ports by changing the first number (e.g. `1000:80` could be seen on `http://localhost:1000` in your browser).

    > Then we tell Docker to mount the `src` folder we created to the `/usr/local/apache2/htdocs` folder on the image. When we make updates to these files on our local machine, we will be able to see those changes immediately.

4. Press <kbd>Ctrl-`</kbd> to open the integrated terminal

    - The default on windows machines is PowerShell, but if you have the Windows Subsystem for Linux and you've downloaded Ubuntu from the Microsoft Store, you can use that as your integrated terminal, or you can even use the Git Bash, which some people look down on, but others think it's great. For Macs, this will open a normal terminal within your editor.

5. Run the following command:

    ```sh
    docker-compose up -d
    ```

6. You can see the running server in the Docker tab in VSCode under Containers

7. Open a web browser to `http://localhost` and see your website!

8. To take down the server, run the following command:

    ```sh
    docker-compose down
    ```

## Step 4: Get Some Basic HTML Going

Add the following to the HTML:

1. An appropriate title

2. An `h1` tag

3. An undordered list

4. 3 tasks on the list

## Step 5: Add a favicon

Let's take a second and get familiar with the most indispensable tool for web development: the inspector. The inspector shows the DOM as the browser sees it, as well as the console, which will tell you of any JavaScript errors. In later labs we'll learn how to inspect local storage, cookies, Vue resources, and more. What we're going to look at right now is the Networks tab.

1. Open the inspector (<kbd>Ctrl-Shift-i</kbd> / <kbd>Cmd-Opt-i</kbd>)
2. Open the Networks tab (you may need to hit the <kbd>>></kbd> button if your screen is small)
3. Perform a hard refresh (<kbd>Ctrl-Shift-r</kbd> / <kbd>Cmd-Shift-r</kbd>)
4. Note the red resource, `favicon.ico`, which has code 404 Not Found. The browser automatically looks for this resource on a hard refresh, but does not look for it again on a normal refresh (<kbd>Ctrl-r</kbd> / <kbd>Cmd-r</kbd>).

With this in mind, find or create an icon image, and stick it in your `src` folder, right next to your `index.html`. Name it `favicon.ico`. It should automatically load on a hard refresh now (200 instead of 404 in Networks tab), and you should be able to see it in the tab bar next to where the title is displayed.

## Step 6: Commit and Push to GitHub

You now have one place (a folder on your computer) where your code lives. Eventually there will be 3:

1. Your Development Environment (your personal computer or a lab computer)

2. GitHub

3. Your Production Environment (a computer somewhere in the Crabtree that you access via SSH)

You created the repo on GitHub already (the BYU-ITC-210 repo), and should see the status and any changes you made in the Source Control tab on the left edge of the VSCode window. VSCode knows how to keep track of changes because of a `.git` directory inside your project root folder. If you can't see the `.git` folder, it may be because your operating system settings are set to hide hidden files and folders. Use Google to find out how to show hidden files and folders on your device.

The root of our project is where you find the `docker-compose.yml` and the `src` folder.

We've created some files and folders inside our project, and you should see them appear in the Source Control tab, under the section __CHANGES__. Hit the <kbd>+</kbd> next to each new file, which automatically calls `git add` on that file. Now, hit the check at the top of the tab, which calls `git commit`. It will prompt you for a message. Take a second and give it a useful name, for example: `'add new files'`. Next, inside the three-dot menu at the top of the tab, click `Push`. This calls `git push`, and after verifying that you are a GitHub user with appropriate permissions on this repository, if you go back to GitHub, you'll see all of the code you wrote stored in the cloud!

## Step 7: Set up your Production Environment

Follow [these instructions](https://gist.github.com/210TAs/660cd61f0210fdd0645be91ecc58969b) on setting up your live server.

# Tips

## `The current user must be in the 'docker-users' group to use Docker Desktop.` 

When multiple people use the same lab computer, then each user must be added to the `docker-users` group. Windows has good documentation about how you can add yourself to the `docker-users` group which can be found [Here](https://docs.microsoft.com/en-us/visualstudio/containers/troubleshooting-docker-errors?view=vs-2019#docker-users-group).

# Passoff

- [ ] Bring up local server page
- [ ] Site has proper structure
- [ ] Site has good title
- [ ] Site has favicon
- [ ] Files are properly pushed to GitHub

## If Live Servers have been set up:

- [ ] Your site works on your live server
- [ ] Live server page brings up `index.html` without any extra path in the URL
- [ ] Live server has directory listing disabled
- [ ] Stop and Restart Apache server
- [ ] Open the apache error log and review contents

# Writeup Questions

- What is the purpose of using Docker containers?
- Why is it useful to have both a development environment and a live server environment?
- What is the purpose of using a code versioning tool (i.e. git)?

