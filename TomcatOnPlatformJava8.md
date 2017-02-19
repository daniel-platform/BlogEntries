# Setup

## Download Software

### Get Tomcat

Download a version of Apache Tomcat.  These instructions use 8.5
http://tomcat.apache.org/download-80.cgi

The tar.gz binary file is all you need.

## Project

Create a new directory to hold your project files and the Tomcat application server's binaries.

```
PROJECT_DIR=~/Projects/TomcatApp
mkdir -p $PROJECT_DIR/tomcat $PROJECT_DIR/base/bin $PROJECT_DIR/.platform
git init $PROJECT_DIR
```

Extract Tomcat into a temporay folder.  Move the `bin` and `lib` folders to you're project's tomcat folder.

```
cd /tmp
tar -zxf ~/Downloads/apache-tomcat-8.5.11.tar.gz 
mv /tmp/apache-tomcat-8.5.11/bin /tmp/apache-tomcat-8.5.11/lib $PROJECT_DIR/tomcat/
```

For the purpose of this tutorial we will put in place the sample applications

```
mv /tmp/apache-tomcat-8.5.11/webapps /tmp/apache-tomcat-8.5.11/conf $PROJECT_DIR/base/
```

## YAML Files

Create a basic `.platform.app.yaml` file:

```
cd $PROJECT_DIR
edit .platform.app.yaml
```

Paste in the following


```
name: tomcat
type: java:8
disk: 5120

mounts:
  "/base/work": "shared:files/work"
  "/base/logs": "shared:files/logs"
  "/base/temp": "shared:files/temp"
  "/base/conf/Catalina/localhost": "shared:files/conf_catalina_localhost"

web:
  commands:
    start: "sh ./start.sh"
```

Then drop-in a basic `.platform/routes.yaml`

```
edit .platform/routes.yaml
```

Paste in the following

```
"http://{default}/":
   type: upstream
   upstream: "tomcat:http"
```

For now just an empty `.platform/services.yaml` will do
```
touch .platform/services.yaml
```

## Scripts

The purpose of the startup script is to set the enviromental variables to let tomcat know were it's located and where to load the apps from.  

Place this script in `base/bin/setenv.sh`

```
#!/bin/sh

export CATALINA_OPTS="$CATALINA_OPTS -Dport.http=${PORT}"
```

Place this script in the root of your project directory as `start.sh`

```
#!/bin/sh

export CATALINA_HOME=${PLATFORM_APP_DIR}/tomcat
export CATALINA_BASE=${PLATFORM_APP_DIR}/base
$CATALINA_HOME/bin/catalina.sh run
```

## Listen Port Configuration

Edit the file `base/conf/server.xml` locate following lines and modify the listen port

```
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```

to

```
<Connector port="${port.http}" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```

Alternativly, you can try this `sed` command:

```
sed -i '' -e 's/Connector port="8080"/Connector port="${port.http}"/' base/conf/server.xml
```

## Save

```
git add .
git commit -m "Tomcat Example"
```

# How to Use

Place your application's code in the `base/webapps` folder. Then add, commit, and push to your plaform.sh project.
For example

```
git remote add platform <YOUR PROJECT's REPO>
git push -u platform master
```
