---
layout: post
title:  "Optimizely Foundation on a Mac"
---

### Introduction

With all of the exciting stuff going on at Optimizely, perhaps the biggest thing for developers is our move to .NET 5, which provides a ton of benefits, performance being one of them.  The other thing that has developers excited is being able to code in any operating system.  I’m on a Mac and have successfully gotten Foundation up and running locally.  I’m excited to share the steps I took to hopefully accelerate the time it takes for you to start working with Optimizely on your Mac.  There have been some great blog posts written that share how to load the solution across multiple different platforms.  I thank those authors for giving me a great head start and I encourage you to explore those as well.

I will start by listing some prerequisites that you will require, which will be a one-time install.  Once we get the prerequisites out of the way, I’ll walk you through the steps I added to my setup script (setup.sh).  Note that I installed Docker for Windows in order to host SQL Server.  If you will be using SQL Azure instead, you can skip that step.

### Prerequisites
Before getting to the solution, some prerequisites are required on your Mac. This section will quickly walk you through those.

### .NET 5
You will need .NET 5, which you can download directly from the [Microsoft website](https://web.archive.org/web/20220703115658/https://dotnet.microsoft.com/download/dotnet/5.0).

### NPM
You will need to install Node and NPM in order to build the CSS.  I prefer using [homebrew](https://web.archive.org/web/20220703115658/https://brew.sh/) as my package manager on my Mac. **Note that the version of Node seems to make a difference and I have found version 12 seems to work well**. Once installed, you can install Node and NPM in a Terminal window, as follows

    brew update
    brew install node@12
    
Check to make sure both Node and NPM are installed:

    node -v
    npm -v

### Visual Studio For Mac
Take advantage of the [latest release](https://web.archive.org/web/20220703115658/https://visualstudio.microsoft.com/vs/mac/preview/) of Visual Studio for Mac in order to maintain that rich IDE experience.

### Docker Desktop
Although the code will be running directly on the Mac, the database will be running within a Docker container, which will require [Docker Desktop](https://web.archive.org/web/20220703115658/https://docs.docker.com/desktop/mac/install/) on your Mac.

### Azure Data Studio
[Azure Data Studio](https://web.archive.org/web/20220703115658/https://docs.microsoft.com/en-us/sql/azure-data-studio/download-azure-data-studio?view=sql-server-ver15) is optional but a nice way to interact with your database hosted in Docker.

### Installation
#### Run the script
That’s all we need for prerequisites.  Let’s get to the code!  You can find my setup.sh script in the Git repo below, which will handle everything in this section.  If you’re not patient enough to read through the rest of this post, you can follow these simple steps.

1. Open a Terminal window
2. Create a folder where you will clone the foundation-mvc-cms repo
            
            mkdir ~/Documents/Code/core-cms
            
3. Switch to the new folder

            cd ~/Documents/Code/core-cms

4. Clone the net5-mac branch of my forked repo

            git clone -b net5-mac https://github.com/chrisvaf/foundation-mvc-cms.git
            
5. Switch to the foundation-mvc-cms folder

            cd foundation-mvc-cms
            
6. Run setup.sh (if you are creating your own script, don’t forget to make it executable --> chmod 755 setup.sh).

            ./setup.sh

#### What the script does
So, what does the script do?  Let’s walk through the different steps.  The first thing it does is prompt for an app name, which is required, and the SQL Server instance which defaults to localhost.  It then installs a couple of tools.  

First is the Optimizely .NET Templates which can be used to create new projects.  .Net 5 provides developers the ability to create new project templates using the dotnet new command.  There are many templates out there to help scaffold different types of .Net 5 applications.  For Optimizely, we provide empty cms and empty commerce projects.

We also also install the .NET CLI tool which provides some nice goodies to do things like create admin users and databases, as well as update the connection string in appsettings.json.  

Finally, we ensure the Optimizely Nuget feed is added as well, so that all packages can be restored.

.NET 5 includes an https dev certificate and we want to trust the cerificate by running the following command:

    dotnet dev-certs https --trust.  
    
Note that this will prompt you for your Mac password.  If you're going to be running this script multiple times you can comment this line out after the first time.

The next thing we do is build the solution using dotnet build.  Once completed, we check to see if the image for the latest version of SQL Server 2019 is available in Docker.  If not, it adds it while also creating a new container called sql_server_optimizely. If it does exist, a check is made to see if a container named sql_server_optimizely has already been added.  The same container can be used to host databases from multiple different sites – no need for more than one.

Once SQL Server is in place, we run 

    npm ci 
    
and 
    
    npm run dev.

The last step is to provision the database.  We first check for the existence of the database and if found, delete the backup history and drop the database.  We then call 

    dotnet-episerver create-cms-database 
    
to provision the new database for our site.

If all goes well, the script ends and we can now load our site!  Note than I initially called dotnet run on the project within the script, however there doesn’t seem to be a way to load the browser, and on top of that I kept getting prompted for my login keychain password.  Instead, open up Visual Studio Preview and load the project.  **F5** will kick off the browser.  Be patient on first load as the site gets provisioned.  You can monitor the database tables in Azure Data Studio to see the tables get provisioned.  If you’re as lucky as me, you should have a running Foundation site within a few minutes.

Hope this helps getting things going on your Mac.  If you encounter any issues or if I left out any prerequisites feel free to reach out and let me know.

Happy coding!
