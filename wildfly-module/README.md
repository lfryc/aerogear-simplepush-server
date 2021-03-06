# Aerogear Simple Push Server WildFly module
This project contains a WildFly subsystem for AeroGear's SimplePush Server.

## Building
From the root folder of this project run the following command:

    mvn package

## Installing
Copy the module produced by ```mvn package``` to the _modules_ directory of the application server.

    cp -r wildfly-module/target/module/org $WILDFLY_HOME/modules/system/layers/base/

For JBoss Application Server 7.x, the following command will copy the modules to the correct location:

    cp -r wildfly-module/target/module/org $AS7_HOME/modules/

## Configuring WildFly


Next, start your server :

    ./standalone.sh

Finally, run the follwing WildFly command line interface script:

    $WILDFLY_HOME/bin/jboss-cli.sh --file=src/main/resources/wildfly-config.cli
    
The above script will add the SimplePush extension/subsystem using an in-memory datastore. Please see the _datastore_ section
below for instructions to add a different datastore.
 
If you inspect the server console output you should see the following message:

    08:56:13,052 INFO  [org.jboss.aerogear.simplepush.subsystem.SimplePushService] (MSC service thread 1-3) SimplePush Server binding to [/127.0.0.1:7777]

## OpenShift
Deploying on OpenShift can be done using the [OpenShift Cartridge](https://github.com/aerogear/openshift-origin-cartridge-aerogear-push). This
cartridge can be installed directly using [OpenShift](https://www.openshift.com/)s web interface.

### Client URL
By default an application created using the OpenShift cartridge can be accessed by using the following urls:

For _secured_ connections:

    https://{APP}-{NAMESPACE}-rhcloud.com:8443/simplepush

For _unsecured_ connections:

    http://{APP}-{NAMESPACE}-rhcloud.com:8000/simplepush

**NOTE:** It is recommended that you always use _secured_ connections.


## Configuration options
The wildfly-config.cli script will add the configuration elements to the running server. But not all configuration options
will be present and you might want to add or update existing ones.   
This section goes through all of the configuration options available.  

    <subsystem xmlns="urn:org.jboss.aerogear.simplepush:1.0">
        <server 
            socket-binding="simplepush-socket-binding" 
            datasource-jndi-name="java:jboss/datasources/SimplePushDS" 
            password="936agbbhh6ee99=999333"
            useragent-reaper-timeout="604800000"
            endpoint-prefix="update"
            endpoint-tls="true"
            endpoint-ack-interval="60000"
            endpoint-socket-binding="simplepush-notify"
            notifier-max-threads="8"
            sockjs-prefix="simplepush"
            sockjs-cookies-needed="true"
            sockjs-url="http://cdn.sockjs.org/sockjs-0.3.4.min.js"
            sockjs-session-timeout="5000"
            sockjs-heartbeat-interval="25000"
            sockjs-max-streaming-bytes-size="131072"
            sockjs-tls="false"
            sockjs-keystore="/simplepush.keystore"
            sockjs-keystore-password="password"
            sockjs-websocket-enable="true" 
            sockjs-heartbeat-interval="18000" 
            sockjs-protocols="push-notification">
            <datastore>
                <jpa datasource-jndi-name="java:jboss/datasources/TestDS" persistence-unit="SimplePushPU"/>
            </datastore>
        </server>
    </subsystem>

#### socket-binding 
This is the name of a socket-binding configured in the _socket-binding-group_ section in a WildFly configuration xml file.  


#### password
This should be a password that will be used to generate the server private key which is used for  encryption/decryption
of the endpoint URLs that are returned to clients upon successful channel registration.

#### useragent-reaper-timeout  
This is the amount of time which a UserAgent can be inactive after which it will be removed from the system.
Default is 604800000 ms (7 days).

#### endpoint-prefix
The prefix for the the notification endpoint url. This prefix will be included in the endpointUrl returned to the client to enabling them to send notifications.

#### endpoint-tls
Configures Transport Layer Security (TLS) for the notification endpointUrl that is returned when a UserAgent/client registers a channel. 
Setting this to _true_ will return a url with _https_ as the protocol.

#### endpoint-ack-interval
This is the interval time for resending un-acknowledged notifications. Default is 60000 ms.

#### endpoint-socket-binding
This is the name of a socket-binding configured in the _socket-binding-group_ section in a WildFly configuration xml file. 
This information is used to configure the host and port that will be returned as the notification endpoints that backend servers can 
use to send notifications to a channel. The can be useful on OpenShift where the host and port that the server binds to might 
not be available to external clients.
The configuration for this socket-binding could look like this:

    <interfaces>
        <interface name="external">
            <inet-address value="domain1.com"/>
        </interface>
    </interfaces>

    <socket-binding-group name="standard-sockets" default-interface="public" port-offset="${jboss.socket.binding.port-offset:0}">
        <socket-binding name="simplepush" port="7777"/>
        <socket-binding name="simplepush-notification" interface="external" port="7777"/>
    </socket-binding-group>


#### notifier-max-threads
This is the maxium number of threads that will be used for handling notifications.

#### sockjs-prefix
The prefix/name, of the SockJS service. For example, in the url _http://localhost/simplepush/111/12345/xhr_, _simplepush_ is the prefix. 

#### sockjs-cookies-needed
This is used by some load balancers to enable session stickyness. Default is true.

#### sockjs-url
The url to the sock-js-<version>.json. This is used by the 'iframe' protocol and the url is replaced in the script 
returned to the client. This allows for configuring the version of sockjs used.  
Default is _http://cdn.sockjs.org/sockjs-0.3.4.min.js_.

#### sockjs-session-timeout
A timeout for inactive sessions. Default is 5000 ms. 

#### sockjs-heartbeat-interval
Specifies a heartbeat interval. Default is 25000 ms.

#### sockjs-max-streaming-bytes-size
The max number of bytes that a streaming transport protocol should allow to be returned before closing the connection, 
forcing the client to reconnect. 
This is done so that the responseText in the XHR Object will not grow and be come an issue for the client. Instead, 
by forcing a reconnect the client will create a new XHR object and this can be see as a form of garbage collection.
Default is 131072 bytes.

#### sockjs-tls
Specified whether Transport Layer Security (TLS) should be used by the SockJS layer.
Default is false.

#### sockjs-keystore
If _tls_ is in use then the value of this property should be a path to keystore available on the classpath of the subystem.

#### sockjs-keystore-password
If _tls_ is in use, then the value of this property should be the password to the keystore specified in _keystore_.

#### sockjs-websocket-enable
Determines whether WebSocket support should be enabled for the server.

#### sockjs-websocket-heartbeat-interval
A heartbeat-interval for WebSockets. This interval is separate from the normal SockJS heartbeat-interval and might be 
required in certain environments where idle connection are closed by a proxy. It is a separate value from the hearbeat 
that the streaming protocols use as it is often desirable to have a much larger value for it.

#### sockjs-websocket-protocols
Adds the specified comma separated list of protocols which will be returned to during the HTTP upgrade request as the header 'WebSocket-Protocol'. 
This is only used with raw WebSockets as the SockJS protocol does not support protocols to be specified by the client yet.

#### datastore
The datastore can be used to configure the datastore which should be used. Currently, in-memory, jpa, redis, and couchdb are supported.

Redis:  
The [Redis datastore](https://github.com/aerogear/aerogear-simplepush-server/tree/master/datastores/redis) can be configured by replacing the content of the datastore element of the simplepush subsystem:

    <datastore>
        <redis host="localhost" port="6379"/>
    </datastore>
    
CouchDB:  
The [CouchDB datastore](https://github.com/aerogear/aerogear-simplepush-server/tree/master/datastores/couchdb) can be configured by replacing the content of the datastore element of the simplepush subsystem:  

    <datastore>
        <couchdb url="http://127.0.0.1:5984" database-name="simplepush"/>
    </datastore>
    
InMemory:    
The [InMemory datastore](https://github.com/aerogear/aerogear-simplepush-server/tree/master/datastores/in-memory) can be configured by replacing the content of the datastore element of the simplepush subsystem:  

    <datastore>
        <in-memory/>
    </datastore>
    
JPA:   
The [JPA datastore](https://github.com/aerogear/aerogear-simplepush-server/tree/master/datastores/jpa) can be configured by replacing the content of the datastore element of the simplepush subsystem:  

    <datastore>
        <jpa datasource-jndi-name="java:jboss/datasources/TestDS" persistence-unit="SimplePushPU"/>
    </datastore>
    
To use JPA you also need a datasource. In this case we will give an example of adding a MySql datasource.  
Create a database and database user:  

    $ mysql -u <user-name>
    mysql> create database simplepush;
    mysql> create user 'simplepush'@'localhost' identified by 'simplepush';
    mysql> GRANT SELECT,INSERT,UPDATE,ALTER,DELETE,CREATE,DROP ON simplepush.* TO 'simplepush'@'localhost';
    
Add a datasource for the SimplePush database:   
The module for mysql can be found in ```src/main/resources/modules/com/mysql```. 
Copy this module to WildFlys modules directory:

    cp -r src/main/resources/modules/com $WILDFLY_HOME/modules/
    
We also need the mysql driver copied to this module:

    mvn dependency:copy -Dartifact=mysql:mysql-connector-java:5.1.18 -DoutputDirectory=/$WILDFLY_HOME/modules/com/mysql/jdbc/main/
    