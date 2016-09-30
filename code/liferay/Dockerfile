FROM mdelapenya/liferay-portal:6.2-ce-ga6-tomcat-hsql
MAINTAINER Manuel de la Peña <manuel.delapenya@liferay.com>

COPY portal-ext.properties $LIFERAY_HOME/portal-ext.properties
COPY setenv.sh $CATALINA_HOME/bin/setenv.sh
COPY ojdbc6.jar $CATALINA_HOME/lib/ext/ojdbc6.jar
