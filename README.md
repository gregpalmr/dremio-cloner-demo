# dremio-cloner-demo
A demonstration of using the Dremio cloner to backup and restore a Dremio Data Lake cluster.

---

### Introduction

TBD

### Pre-requisites

* A Windows computer or MacOS computer
* Docker Desktop for Window or MacOS
* Access to Dockerhub to pull the Dremio Opensource Edition image
* Access to this Github repository

### Download this Github repo

If you have the git utility installed on your computer you can run the command:

     git clone https://github.com/gregpalmr/dremio-cloner-demo

If you don't have the git utility installed, you can download this repo as a ZIP file by following these steps:

1. Point your Web browser to this Github repo: https://github.com/gregpalmr/dremio-cloner-demo
2. Click on the green "Code" button and select the "Download ZIP" option. 
3. Unzip the file to a folder in your user's home directory. 

Open up a Windows Powershell or MacOS terminal session and navigate to the folder to which you places the Git repo files.

     Windows

          cd \Users\<MY USER>\dremio-cloner-demo-main

     MacOS

          cd /Users/<MY USER>/dremio-cloner-demo

### Launch the first Dremio Cluster

Launch a Dremio cluster as a Docker container

     docker run -p 9047:9047 -p 31010:31010 -p 45678:45678 dremio/dremio-oss

### Install the sample data source and some test users

The Dremio cloner doesn't support encyrpted passwords, so you must create the test users and the sample data source manually. In a production environment, most likely Dremio would get usernames and passwords by integrating with an Active Directory or LDAP service, but for this demo, create the users maually.

Point your web browser to the first Dremio cluster (http://localhost:9048) and perform the following steps:

1. Register the first (admin) user with the username `admin1` and password `changeme123`.
2. Click on the Admin link in the upper right corner and then the Users link on the left.
   Add the following users: `fred`, `barney`.
3. Click on the "Add Sample Source" button to define the data source.
   (This is required because the Dremio Cloner cannot create data sources, they must already exist.)


### Restore the metadata from a previously backed up Dremio cluster

Launch a shell session into the new Docker container

     docker ps
     docker exec -it <DOCKER IMG ID> /bin/bash

Download the source code to the Dremio Cloner

     cd /tmp
     git clone https://github.com/mikhail-dremio/dremio-cloner

Go to the cloner's config directory

     cd /tmp/dremio-cloner/config

Modify the "DR" version of the "GET" Dremio Cloner config file. 

     cp ./config_dr_read.json ./config_dr_read.json.orig

     sed -i 's/<DREMIO-ADMIN-USER>/admin1/'          ./config_dr_read.json
     sed -i 's/<TARGET_JSON_FILE_NAME>/dremio-cloner.backup/'  ./config_dr_read.json
     sed -i 's/<OPTIONAL_LOG_FILE_NAME>/dremio-cloner.log/'  ./config_dr_read.json
     sed -i 's/skip/process/' ./config_dr_read.json

# Go to the Cloner's source code directory
#
cd ../src

# Run the Dremio Cloner GET operation using the modified config file
#
python dremio_cloner.py ../config/config_dr_read.json -p <admin user password>

# Display the GET results using the JSON formatter
#
cat dremio-cloner.backup | jq .

# NOTE: Install pip
#
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py && python get-pip.py

export PATH=$PATH:/var/lib/dremio/dremio/.local/bin

pip install requests 

# NOTE: Install the jq json formatting program
#
curl https://stedolan.github.io/jq/download/linux64/jq > /var/lib/dremio/dremio/.local/bin/jq && chmod +x /var/lib/dremio/dremio/.local/bin/jq

# Exit the shell from the Docker container
#
exit

# Copy the Cloner backup file from the docker image
#
docker ps
docker cp <DOCKER IMG ID>:/tmp/dremio-cloner/src/dremio-cloner.backup .

# Launch a second test Dremio cluster as a Docker container (with diff port numbers)
#
docker run -p 9048:9047 -p 31011:31010 -p 45679:45678 dremio/dremio-oss

# Point your web browser to the second Dremio cluster and:
#   1. Register the following users: admin_user, fred, barney
#   2. Click on the "Add Sample Source" button to define the data source
#      (This is required because the Dremio Cloner cannot create data sources, they must already exist)
#
https://localhost:9048

# Download the source code to the Dremio Cloner
#
docker ps
docker exec -it <DOCKER IMG ID> /bin/bash -c "cd /tmp && git clone https://github.com/mikhail-dremio/dremio-cloner"

# Copy the Cloner backup file to the second docker image
#
docker ps
docker cp dremio-cloner.backup <DOCKER IMG ID>:/tmp/dremio-cloner/src/dremio-cloner.backup

# Launch a shell session in the second Docker container
#
docker ps 
docker exec -it <DOCKER IMG ID> /bin/bash

# Go to the Cloner's config directory
#
cd /tmp/dremio-cloner/config

# Modify the "DR" version of the "PUT" config file
#
cp ./config_dr_write.json ./config_dr_write.json.orig
vi config_dr_write.json

  < set the admin username, target.filename, logging.filename configurations >

OR

cp ./config_dr_write.json ./config_dr_write.json.orig

sed -i 's/<DREMIO-ADMIN-USER>/admin1/'                    ./config_dr_write.json
sed -i 's/<SOURCE_JSON_FILE_NAME>/dremio-cloner.backup/'  ./config_dr_write.json
sed -i 's/<OPTIONAL_LOG_FILE_NAME>/dremio-cloner.log/'    ./config_dr_write.json

sed -i 's/"space.process_mode":"skip"/"space.process_mode":"create_only"/'    ./config_dr_write.json
sed -i 's/"folder.process_mode":"skip"/"folder.process_mode":"create_only"/'  ./config_dr_write.json
sed -i 's/"vds.process_mode":"skip"/"vds.process_mode":"create_only"/'        ./config_dr_write.json

sed -i 's/skip/create_only/' ./config_dr_write.json

# Go to the Cloner's source code directory
#
cd ../src

# Run the Dremio Cloner GET operation using the modified config file
#
python dremio_cloner.py ../config/config_dr_write.json -p <admin user password>

# Point your web browser to the second Dremio cluster 
#
http://localhost:9048

# Display the GET results using the JSON formatter
#
cat dremio-cloner.backup | jq .

