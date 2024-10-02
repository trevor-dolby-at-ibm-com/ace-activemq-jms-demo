# ace-activemq-jms-demo
ACE with ActiveMQ, SSL, and user/pw auth

## Flows

The current test flow is as follows:

![image](https://github.com/user-attachments/assets/b8367770-e159-4f8a-844c-d7414f58e2c9)

and relies on an ACE server that has the `ActiveMQJARs` shared library deployed to it with server.conf.yaml 
set to tell the server to use the shared library for shared classes (so the JMS nodes can find the JARs):
```
additionalSharedClassesDirectories: '{ActiveMQJARs}'
```

Note that this will only work for a server that has been configured to use Java11 or later due to the ActiveMQ
JAR build process, so for ACE v12 the following command may be needed:
```
ibmint specify jre --work-dir /home/tdolby/IBM/ACET12/ace-activemq-jms-demo-workspace/TEST_SERVER --version 11
```

The JMSPolicy must be customized with the correct hostname for ActiveMQ and deployed to the same
server, with any user/pw information added also with a command such as 
```
mqsisetdbparms -w /home/tdolby/IBM/ACET12/ace-activemq-jms-demo-workspace/TEST_SERVER -n jms::queueConnectionFactory -u user -p password
```

To test the resulting configuration, the URL `http://localhost:7800/jmsOutput` can be access from
a browser or curl to make the flow put a message to `second.queue`.

## Notes

- ACE v13 defaults to Java17 so there should be no need to specify a JRE
- Java17 allows later clients than 5.18.5; see https://activemq.apache.org/components/classic/download/ for details.
- ActiveMQ JARs in this repo should be replaced with originals from https://mvnrepository.com/artifact/org.apache.activemq/activemq-client or equivalent

## Server setup

This repo is not intended to be definitive on the server configuration, but simple servers can be 
configured quickly. 

conf/activemq.xml needs
```
<transportConnector name="openwire" uri="ssl://0.0.0.0:61616?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
```
and
```
      <plugins>
        <simpleAuthenticationPlugin anonymousAccessAllowed="false">
          <users>
            <authenticationUser username="system" password="manager" groups="users,admins"/>
            <authenticationUser username="user" password="password" groups="users"/>
            <authenticationUser username="guest" password="password" groups="guests"/>
          </users>
        </simpleAuthenticationPlugin>
      </plugins>
```

and the SSL keystore can be specified with
```
export _JAVA_OPTIONS="-Djavax.net.ssl.keyStore=/home/ubuntu/activemq/apache-activemq-6.1.3/conf/cedars.p12 -Djavax.net.ssl.keyStoreType=PKCS12 -Djavax.net.ssl.keyStorePassword=changeit"
```
