# __Springboot jar 패키징 후 실행 방법__

## __1. $ java -jar SpringbootSample.jar__

- 위와 같이 실행하면 ssh 연결이 끊어질때 프로그램도 같이 종료된다.

​

## __2. $ java -jar SpringbootSample.jar &__

- 백그라운드로 실행은 되지만 실행한 사용자가 로그아웃하면 프로그램은 역시 종료된다.

​

## __3. $ nohup java -jar SpringbootSample.jar &__

- 사용자가 로그아웃해도 백그라운드로 실행된다.

​

3번으로 하면 실행은 되지만 프로그램 종료를 자동적으로 하기가 어렵다. 

종료를 하기 위해서는 pid 를 알아야한다.

따라서 쉘스크립트를 작성하여 서비스에 등록하는 방법으로 문제를 최종적으로 해결하였다.

​

## __4. 쉘스크립트 작성 후 서비스 등록__

- /etc/init.d/webService 로 등록하여 service webService start/stop/restart 명령을 사용

```shell
"#!/bin/sh

SERVICE_NAME=example

PATH_TO_JAR=/home/test/SpringbootSample.jar

PID_PATH_NAME=/var/run/example.pid

JAVA_OPTS=""-server -Dspring.profiles.active=dev""

case $1 in

start)

echo ""Starting $SERVICE_NAME ...""

if [ ! -f $PID_PATH_NAME ]; then

nohup java $JAVA_OPTS -jar $PATH_TO_JAR /tmp 2>> /dev/null >> /dev/null &

echo $! > $PID_PATH_NAME

echo ""$SERVICE_NAME started ...""

else

echo ""$SERVICE_NAME is already running ...""

fi

;;

stop)

if [ -f $PID_PATH_NAME ]; then

PID=$(cat $PID_PATH_NAME);

echo ""$SERVICE_NAME stoping ...""

kill $PID;

echo ""$SERVICE_NAME stopped ...""

rm $PID_PATH_NAME

else

echo ""$SERVICE_NAME is not running ...""

fi

;;

restart)

if [ -f $PID_PATH_NAME ]; then

PID=$(cat $PID_PATH_NAME);

echo ""$SERVICE_NAME stopping ..."";

kill $PID;

echo ""$SERVICE_NAME stopped ..."";

rm $PID_PATH_NAME

echo ""$SERVICE_NAME starting ...""

nohup java $JAVA_OPTS -jar $PATH_TO_JAR /tmp 2>> /dev/null >> /dev/null &

echo $! > $PID_PATH_NAME

echo ""$SERVICE_NAME started ...""

else

echo ""$SERVICE_NAME is not running ...""

fi

;;

esac

"
```
