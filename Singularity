Bootstrap: docker
From: centos:centos7.5.1804

%help
Singularity container for the 0.9.4 snapshot of AsterixDB

This environment utilizes the config and log files within your home directory.
If you want to change the config of AsterixDB, use the files in /opt/asterix/asterixdb-files/conf.
Logs for the program are located in /opt/asterix/asterixdb-files/logs.
Data files for the database are located in /opt/asterix/asterixdb-files/data.

Files located in the asterix user home directory under /opt/asterix can be changed freely, however
any other files located within the singularity instance are immutable.

%post
    yum -y update

    # Installs the requirements for building AsterixDB
    yum -y install git
    yum -y install java-1.8.0-openjdk-devel
    yum -y install centos-release-scl
    yum -y install rh-maven35

    # Clones the AsterixDB repository to /opt
    cd /opt
    git clone https://github.com/apache/asterixdb.git

    # Sets the environment variables for maven
    export PATH="/opt/rh/rh-maven35/root/usr/bin:${PATH:-/bin:/usr/bin}"
    export MANPATH="/opt/rh/rh-maven35/root/usr/share/man:${MANPATH}"
    export PYTHONPATH="/opt/rh/rh-maven35/root/usr/lib/python2.7/site-packages${PYTHONPATH:+:}${PYTHONPATH:-}"
    export JAVACONFDIRS="/opt/rh/rh-maven35/root/etc/java${JAVACONFDIRS:+:}${JAVACONFDIRS:-}"
    export XDG_CONFIG_DIRS="/opt/rh/rh-maven35/root/etc/xdg:${XDG_CONFIG_DIRS:-/etc/xdg}"
    export XDG_DATA_DIRS="/opt/rh/rh-maven35/root/usr/share:${XDG_DATA_DIRS:-/usr/local/share:/usr/share}"

    # Builds AsterixDB
    cd /opt/asterixdb
    mvn clean package -DskipTests

    # Chowns the /opt/asterixdb folder to user with uid and gid 8889
    chown -R 8889:8889 /opt/asterixdb

%environment
    umask 022

######################
#        Apps        #
######################


#### CCSTART APP ####


%apphelp ccstart
USAGE: singularity run --app ccstart <container path> [global options...]

A singularity app that starts the AsterixDB cluster controller within the singularity image.

GLOBAL OPTIONS:
    -c <path>         Specifies a path to the cluster controller configuration file.
                      Default path is /opt/asterix/asterixdb-files/conf/cc.conf

    -l <path>         Specifies a path to log stdout. Default path is /opt/asterix
                      /asterixdb-files/logs/cc-service.log

%apprun ccstart
#!/bin/bash

conf_file=""
log_file=""

while getopts ":c:l:h" opt
  do
    case ${opt} in
      c )
        if [[ -f $OPTARG ]] ; then
          conf_file=$OPTARG
        else
          echo "ERROR: Configuration file $OPTARG does not exist"
          exit 1
        fi
        ;;
      l )
        file_name="$(basename $OPTARG 2> /dev/null)"
        file_path="$OPTARG"
        file_directory=${file_path%"$file_name"}
        if [[ -d $file_directory ]] ; then
          log_file=$OPTARG
        else
          echo "ERROR: Directory $OPTARG does not exist"
          exit 1
        fi
        ;;
      h )
        echo "USAGE: singularity run --app ccstart <container path> [global options...]"
        echo ""
        echo "A singularity app that starts the AsterixDB cluster controller within the singularity image."
        echo ""
        echo "GLOBAL OPTIONS:"
        echo "    -c <path>         Specifies a path to the cluster controller configuration file."
        echo "                      Default path is /opt/asterix/asterixdb-files/conf/cc.conf"
        echo ""
        echo "    -l <path>         Specifies a path to log stdout. Default path is /opt/asterix"
        echo "                      /asterixdb-files/logs/cc-service.log"
        exit 0
      ":" )
        echo "USAGE: singularity run --app ccstart <container path> -$OPTARG <path>"
        exit 1
        ;;
      \? )
        echo "ERROR: Invalid argument -$OPTARG"
        exit 1
        ;;
  esac
done

cd /opt/asterix

if [[ -z "$conf_file" && -z "$log_file" ]] ; then
  echo "No options specified, starting with default configuration file at /opt/asterix/asterixdb-files/conf/cc.conf and logging to /opt/asterix/asterixdb-files/logs/cc-service.log"
  echo "Starting the AsterixDB CCService..."
  nohup "/opt/asterixdb/asterixdb/asterix-server/target/asterix-server-0.9.4-SNAPSHOT-binary-assembly/bin/asterixcc" -config-file "/opt/asterix/asterixdb-files/conf/cc.conf" >> "/opt/asterix/asterixdb-files/logs/cc-service.log" 2>&1 &
  exit 0

elif [[ -n "$conf_file" && -n "$log_file" ]] ; then
  echo "Using the configuration file at $conf_file"
  echo "Writing logs to $log_file"
  echo "Starting the AsterixDB CCService..."
  nohup "/opt/asterixdb/asterixdb/asterix-server/target/asterix-server-0.9.4-SNAPSHOT-binary-assembly/bin/asterixcc" -config-file "$conf_file" >> "$log_file" 2>&1 &
  exit 0

else
  if [[ -n "$conf_file" ]] ; then
    echo "Using the configuration file at $conf_file"
    echo "Writing logs to /opt/asterix/asterixdb-files/logs/cc-service.log"
    echo "Starting the AsterixDB CCService..."
    nohup "/opt/asterixdb/asterixdb/asterix-server/target/asterix-server-0.9.4-SNAPSHOT-binary-assembly/bin/asterixcc" -config-file "$conf_file" >> "/opt/asterix/asterixdb-files/logs/cc-service.log" 2>&1 &
    exit 0

  else
    echo "Starting with default configuration file at /opt/asterix/asterixdb-files/conf/cc.conf"
    echo "Writing logs to $log_file"
    echo "Starting the AsterixDB CCService..."
    nohup "/opt/asterixdb/asterixdb/asterix-server/target/asterix-server-0.9.4-SNAPSHOT-binary-assembly/bin/asterixcc" -config-file "/opt/asterix/asterixdb-files/conf/cc.conf" >> "$log_file" 2>&1 &
    exit 0
  fi
fi

#### NCSTART APP ####

%apphelp ncstart
USAGE: singularity run --app ncstart <container path> [global options...]

A singularity app that starts the AsterixDB node controller within the singularity image.

GLOBAL OPTIONS:
    -c <path>        Specifies a path to the node controller configuration file.
                     Default runs the node controller without a configuration file.

    -l <path>        Specifies a path to log stdout. Default path is /opt/asterix
                     /asterixdb-files/logs/nc-service.log

%apprun ncstart
#!/bin/bash

conf_file=""
log_file=""

while getopts ":c:l:h" opt
  do
    case ${opt} in
      c )
        if [[ -f $OPTARG ]] ; then
          conf_file=$OPTARG
        else
          echo "ERROR: Configuration file $OPTARG does not exist"
          exit 1
        fi
        ;;
      l )
        file_name="$(basename $OPTARG 2> /dev/null)"
        file_path="$OPTARG"
        file_directory=${file_path%"$file_name"}
        if [[ -d $file_directory ]] ; then
          log_file=$OPTARG
        else
          echo "ERROR: Directory $OPTARG does not exist"
          exit 1
        fi
        ;;  
      h )
        echo "USAGE: singularity run --app ncstart <container path> [global options...]"
        echo ""
        echo "A singularity app that starts the AsterixDB node controller within the singularity image."
        echo ""
        echo "GLOBAL OPTIONS:"
        echo "    -c <path>         Specifies a path to the node controller configuration file."
        echo "                      Default runs the node controller without a configuration file."
        echo ""
        echo "    -l <path>         Specifies a path to log stdout. Default path is /opt/asterix"
        echo "                      /asterixdb-files/logs/nc-service.log"
        exit 0
      ":" )
        echo "USAGE: singularity run --app ncstart <container path> -$OPTARG <path>"
        exit 1
        ;;
      \? )
        echo "ERROR: Invalid argument -$OPTARG"
        exit 1
        ;;
  esac
done

cd /opt/asterix

if [[ -z "$conf_file" && -z "$log_file" ]] ; then
  echo "No options specified, defaulting to starting without a configuration file and logging to /opt/asterix/asterixdb-files/logs/nc-service.log"
  echo "Starting the AsterixDB NCService..."
  nohup "/opt/asterixdb/asterixdb/asterix-server/target/asterix-server-0.9.4-SNAPSHOT-binary-assembly/bin/asterixncservice" >> "/opt/asterix/asterixdb-files/logs/nc-service.log" 2>&1 &
  exit 0

elif [[ -n "$conf_file" && -n "$log_file" ]] ; then
  echo "Using the configuration file at $conf_file"
  echo "Writing logs to $log_file"
  echo "Starting the AsterixDB NCService..."
  nohup "/opt/asterixdb/asterixdb/asterix-server/target/asterix-server-0.9.4-SNAPSHOT-binary-assembly/bin/asterixncservice" -config-file "$conf_file" >> "$log_file" 2>&1 &
  exit 0

else
  if [[ -n "$conf_file" ]] ; then
    echo "Using the configuration file at $conf_file"
    echo "Writing logs to /opt/asterix/asterixdb-files/logs/nc-service.log"
    echo "Starting the AsterixDB NCService..."
    nohup "/opt/asterixdb/asterixdb/asterix-server/target/asterix-server-0.9.4-SNAPSHOT-binary-assembly/bin/asterixncservice" -config-file "$conf_file" >> "/opt/asterix/asterixdb-files/logs/nc-service.log" 2>&1 &
    exit 0

  else
    echo "Starting without a configuration file..."
    echo "Writing logs to $log_file"
    echo "Starting the AsterixDB NCService..."
    nohup "/opt/asterixdb/asterixdb/asterix-server/target/asterix-server-0.9.4-SNAPSHOT-binary-assembly/bin/asterixncservice" >> "$log_file" 2>&1 &
    exit 0
  fi
fi
