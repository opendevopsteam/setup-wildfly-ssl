# setup-wildfly-ssl
🛠 Setup WildFly SSL

DevOps should configure SSL support on WildFly application servers for security reasons. The following steps describe how to configure HTTPS on local server for the web application:

**Step 1:**

*Generate a keystore and self-signed certificate*

Ensure that Java is installed and setup on `JAVA_HOME` properly as JRE keytool will be used for this purpose.

Switch to a command-line and execute the following command as shown below:

`keytool -genkey -alias mycert -keyalg RSA -keystore mycert.keystore -validity 365`

The aforementioned command has some default sets, and also prompts the developer to enter additional information as shown below:

```
What is your first and last name?
  [Unknown]:  Orestis Pantazos
What is the name of your organizational unit?
  [Unknown]:  Open DevOps
What is the name of your organization?
  [Unknown]:  opendevops.dev
What is the name of your City or Locality?
  [Unknown]:  Athens
What is the name of your State or Province?
  [Unknown]:  Greece
What is the two-letter country code for this unit?
  [Unknown]:  GR
Is CN=localhost, OU=Open DevOps, O=opendevops.dev, L=Athens, ST=Greece, C=GR correct?
  [no]:  yes
```

**Step 2:**

The command generates **mycert.keystore** file in the folder that you are currently working. Copy this to your WildFly config directory (`%JBOSS_HOME%/standalone/config`)

**Step 3:**

*Configure the additional WildFly Security Realm*

The next step is to configure the new keystore as a server identity for SSL in the WildFly security-realms section of the `standalone.xml`. You can insert the source code after `<management>` tag and also inside `<security-realms>` tag in the XML file.

```xml
<management>
    <security-realms>
        <security-realm name="UndertowRealm">
            <server-identities>
                <ssl>
                    <keystore path="mycert.keystore" relative-to="jboss.server.config.dir" keystore-password="secret" alias="mycert" key-password="secret"/>
                </ssl>
            </server-identities>
        </security-realm>
```

**Step 4:**

*Configure Undertow Subsystem for SSL*

If the default-server is running, add the https-listener to the undertow subsystem:

```xml
<subsystem xmlns="urn:jboss:domain:undertow:1.2">
    <server name="default-server">
        <https-listener name="https" socket-binding="https" security-realm="UndertowRealm"/>
```

Replace only the word `UndertowRealm` with the previous one for https listener in the given namespace into `security-realm="..."`.

**Step 5:**

SSL port of the current instance is already for connection in `https://localhost:8443/`. Otherwise, the SSL port can be changed to 443 as default port number in the end/bottom of the file as shown below:

```xml
<socket-binding-group name="standard-sockets" default-interface="public" port-offset="${jboss.socket.binding.port-offset:0}">
    <socket-binding name="ajp" port="${jboss.ajp.port:8009}" />
    <socket-binding name="http" port="${jboss.http.port:8080}" />
    <socket-binding name="https" port="${jboss.https.port:443}" />
```

> READY 
