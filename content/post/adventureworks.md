---
author: "Leoren Tanyag"
date: 2020-01-03
linktitle: Importing Adventureworks using Azure Data Studio (via local SQL Server)
title: Importing Adventureworks using Azure Data Studio (via local SQL Server)
categories: [ "Development", "howto" , SQL]
tags: ["adventureworks", "sql", "howto"]
weight: 9
---

If you’re like me, and is tying to follow a tutorial about SQL which is using Adventureworks on Mac… or just simply wanna play with a good sample DB. Here’s a step by step guide on how to set it up.

### Prerequisites
* You will need [Docker](https://docs.docker.com/docker-for-mac/install/)
* Install [Azure Data Studio](https://docs.microsoft.com/en-us/sql/azure-data-studio/download?view=sql-server-ver15)
* Download [Adventureworks](https://docs.microsoft.com/en-us/sql/samples/adventureworks-install-configure?view=sql-server-2017) (any of the OLTP downloads would work)
    

### Run SQL Server
* This is using this instruction - https://docs.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker?view=sql-server-ver15&pivots=cs1-bash

Pull SQL Server
```
sudo docker pull mcr.microsoft.com/mssql/server:2017-latest
```

Run SQL Server locally.
```
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=<YourStrong@Passw0rd>"  -p 1433:1433 --name sql1 -d mcr.microsoft.com/mssql/server:2017-latest
```

See running containers
```
docker ps
```
![image](/img/adventureworks/1.jpg)

You now have a SQL Server running locally

### Connect to your SQL Server using Azure Data Studio
* Click on `New Connection`
    ![image](/img/adventureworks/2.jpg)
        or if you’ve used it before, under Connections, next to SERVERS, click the New Connection Icon
    ![image](/img/adventureworks/3.jpg)
    
* Fill it in:
    * Server: localhost
    * Username: sa
    * Password: (what you put in <YourStrong@Passw0rd>) above when you ran the docker container
    ![image](/img/adventureworks/4.jpg)

### Import AdventureWorks.bak 
* Download AdventureWorks2017.bak (or whatever year) from
    * https://docs.microsoft.com/en-us/sql/samples/adventureworks-install-configure?view=sql-server-2017
* Copy it over to your docker container
    * Check your container ID and copy over your file into the mssql directory in the container. 
       In this case, my file saved in my `~/Downloads` directory

    ```
    docker ps
    docker cp ~/Downloads/AdventureWorks2017.bak <containerID>:/var/opt/mssql/data/
    ```
* Back to Azure Data Studios, click on your localhost connection, and click Restore in the Dashboard
![image](/img/adventureworks/5.jpg)

* In General > Source. Select Restore from: Backup file. And in Backup file path, click (…)
![image](/img/adventureworks/6.jpg)

* You should be able to see AdventureWorks under data
![image](/img/adventureworks/7.jpg)

* Hit **Restore**
* You now should be able to see your AdventureWorks db under Databases. If not, right click on Databases and click “Refresh”.
![image](/img/adventureworks/8.jpg)

* You can click on the AdventureWorks DB and click New Query on the dashboard
![image](/img/adventureworks/9.jpg)


Happy Learning!





