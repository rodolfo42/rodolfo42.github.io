---
layout: post
title: Apache Tomcat 7 service script
date: 2013-05-27
comments: true
---

So, here's my Tomcat service init script. It was adapted from many others throughout the web, which when didn't gave me syntax errors, didn't make use of tomcat's PID file or just didn't work at all.

<!-- more -->

> Replace `CATALINA_HOME` with your own tomcat install path

## The script

{% highlight bash %}
#!/bin/bash
### BEGIN INIT INFO
# Provides: tomcat
# Defalt-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Description: Apache Tomcat 7
### END INIT INFO

# base dir of tomcat installation #
CATALINA_HOME=/opt/tomcat/

# full path for file which will hold the PID #
# used by 'status' #
CATALINA_PID=/var/run/tomcat.pid
export CATALINA_PID

status=0

case $1 in
  start)
    sh $CATALINA_HOME/bin/startup.sh
  ;;

  stop)
    sh $CATALINA_HOME/bin/shutdown.sh
  ;;

  restart)
    sh $CATALINA_HOME/bin/shutdown.sh
    sh $CATALINA_HOME/bin/startup.sh
  ;;

  status)
    if ! [ -z "$CATALINA_PID" ]; then
      if [ -r $CATALINA_PID ]; then
        pid=`cat "$CATALINA_PID"`
        if ! [ -z $pid ]; then
          echo "tomcat (pid  $pid) is running..."
        else
          echo "tomcat is stopped"
        fi
      else
        echo "tomcat is stopped"
      fi
    else
      echo "No PID file found for tomcat..."
      echo "(variable \$CATALINA_PID is not defined)"
      status=1
    fi
  ;;

esac
exit $status
{% endhighlight %}

## How to use it

First, download the script contents and copy the file to `/etc/init.d` (as root):

    $ wget https://gist.github.com/rodolfo42/5654188/raw/tomcat.sh
    $ sudo cp tomcat.sh /etc/init.d/tomcat

> Don't forget to strip the extension as above

Now, to use it:

    # service tomcat (start|stop|status)

To enable it on system start-up (just need to add it, the run levels are already in the script):

    # chkconfig --add tomcat

Or, on Debian-based systems:

    # update-rc.d tomcat 20 2 3 4 5

Hope it helps you as it helps me.
