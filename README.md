# Windup (RHAMT) Instructions

### Setup Environment

1. Install the Java SE Development Kit (JDK) version 8. We recommend using the OpenJDK or the Oracle JDK.
2. Install the RHAMT CLI.
   a. Download the ZIP archive from here - https://developers.redhat.com/download-manager/file/4.0.0/migrationtoolkit-rhamt-cli-4.0.0.offline.zip
   b. Extract the ZIP archive.
      $ unzip migrationtoolkit-rhamt-cli-4.0.0.offline.zip

### Analyze your application

Now that the RHAMT CLI is installed, you can run the CLI to analyze your web application to identify the changes necessary to migrate it from WebSphere to JBoss EAP 6.4

1. Download the application archive deployed on Websphere to your file system.
2. Run the CLI with the necessary options.
   a. Navigate into the rhamt-cli-4.0.0.Final/bin/ directory.
   b. Execute the CLI with the necessary options.
      $ ./rhamt-cli --input /path/to/simple-sample-app.ear --output /path/to/output/ --source websphere --target eap6 --packages com.td
      This command uses the following options:
      --input: Path to the application to analyze.
      --output: Output directory for the generated reports.
      --source: Source technology of the application.
      --target: Target technology for the application migration.
      --packages: List of packages to evaluate.
3. Open the generated report.
   a. The location of the report is displayed in your terminal once the execution is complete.
      Report created: /path/to/output/  
      Access it at this URL: file:///path/to/output/
   b. Open ./ in a browser.
4. Review the reports.
   a. In the Application List, take note of the story points identified for the application. This helps you assess the level of effort required to migrate this application.
   b. Click the application (ear/war) link.
   c. Click the Migration Issues link in the top navigation bar. This report shows a summary of all migration issues identified in the application.
   d. Click on an issue to show the list of files that contain the issue.
   e. Click the file name to view the file contents.
   f. The line of code affected by the issue is highlighted and information about the issue and how to resolve it are displayed.
   g. See the Review the Reports section of the RHAMT CLI Guide to learn more about examining other available reports.

### Reference Links
You may refer to the below links for more information on RHAMT CLI
RHAMT CLI Guide - https://access.redhat.com/documentation/en-us/red_hat_application_migration_toolkit/4.0/html-single/cli_guide/

RHAMT Getting Started Guide - https://access.redhat.com/documentation/en-us/red_hat_application_migration_toolkit/4.0/html-single/getting_started_guide/

RHAMT Eclipse Plugin Guide - https://access.redhat.com/documentation/en-us/red_hat_application_migration_toolkit/4.0/html-single/eclipse_plugin_guide/

RHAMT Download - https://developers.redhat.com/products/rhamt/download/


# Vault setup

## Create a Java Keystore to store sensitive strings 

### Prerequisites
The keytool command must be available to use. It is provided by the Java Runtime Environment (JRE). Locate the path for the file. In Red Hat Enterprise Linux, it is installed to /usr/bin/keytool.

keytool -genseckey -alias vault -storetype jceks -keyalg AES -keysize 128 -storepass vault22 -keypass vault22 -validity 730 -keystore /opt/jboss/security/vault/vault.keystore

A file named vault.keystore is created in the /opt/jboss/security/vault/ directory. It stores a single key, called vault, which will be used to store encrypted strings, such as passwords, for JBoss EAP 6.
  1. Configure JBoss EAP to use vault
  2. Setup a vault to store passwords
You can use the interactive session or use non-interactive commands to setup vault.

### Interactive session:
  /bin $ ./vault.sh 
  =========================================================================

  JBoss Vault

  JBOSS_HOME: /opt/jboss-as/

  JAVA: java

  VAULT Classpath: /opt/jboss-as/modules/org/picketbox/main/*:/opt/jboss-as/modules/org/jboss/logging/main/*:/opt/jboss-as/modules/org/jboss/common-core/main/*:/opt/jboss-as/modules/org/jboss/as/security/main/*
=========================================================================

**********************************
****  JBoss Vault ********
**********************************
Please enter a Digit::   0: Start Interactive Session  1: Remove Interactive Session  2: Exit
0
Starting an interactive session
Enter directory to store encrypted files (end with either / or \ based on Unix or Windows:/opt/vault/
Enter Keystore URL:/opt/vault/vault.keystore
Enter Keystore password: 
Enter Keystore password again: 
Values match
Enter 8 character salt:12345678
Enter iteration count as a number (Eg: 44):50

Make note of the following:
********************************************
Masked Password:MASK-5WNXs8oEbrs
salt:12345678
Iteration Count:50
********************************************

Enter Keystore Alias:vault
Jan 01, 2017 11:48:39 AM org.jboss.security.vault.SecurityVaultFactory get
INFO: Getting Security Vault with implementation of org.picketbox.plugins.vault.PicketBoxSecurityVault
Obtained Vault
Initializing Vault
Vault is initialized and ready for use
Handshake with Vault complete

You may add a password to the vault as shown below
/bin $ ./vault.sh

Please enter a Digit::   0: Start Interactive Session  1: Remove Interactive Session  2: Exit
0
Starting an interactive session
Enter directory to store encrypted files:EAP_HOME/vault/                        
Enter Keystore URL:EAP_HOME/vault/vault.keystore
Enter Keystore password: vault22
Enter Keystore password again: vault22
Values match
Enter 8 character salt:1234abcd
Enter iteration count as a number (Eg: 44):120
Enter Keystore Alias:vault
Initializing Vault
Oct 17, 2014 12:58:11 PM org.picketbox.plugins.vault.PicketBoxSecurityVault init
INFO: PBOX000361: Default Security Vault Implementation Initialized and Ready
Vault Configuration in AS7 config file:
********************************************
...
</extensions>
<vault>
  <vault-option name="KEYSTORE_URL" value="EAP_HOME/vault/vault.keystore"/>
  <vault-option name="KEYSTORE_PASSWORD" value="MASK-5dOaAVafCSd"/>
  <vault-option name="KEYSTORE_ALIAS" value="vault"/>
  <vault-option name="SALT" value="1234abcd"/>
  <vault-option name="ITERATION_COUNT" value="120"/>
  <vault-option name="ENC_FILE_DIR" value="EAP_HOME/vault/"/>
</vault><management> ...
********************************************
Vault is initialized and ready for use
Handshake with Vault complete

### Non - Interactive command:
You may use the below command to add passwords to the vault.
./vault.sh -b <vault block> -a <attribute name> -s <salt> -i <iterations> -k <keystore url> -x <sensitive string value> -p <keystore password> -e <directory of the vault keystore>

Or 

./vault.sh --keystore EAP_HOME/vault/vault.keystore --keystore-password vault22 --alias vault --vault-block vb --attribute password --sec-attr 0penS3sam3 --enc-dir EAP_HOME/vault/ --iteration 120 --salt 1234abcd

For more details use the following command
./vault.sh --help.

usage: vault.sh <empty> |  [-a <arg>] [-b <arg>] -c | -h | -x <arg> [-e
       <arg>]  [-i <arg>] [-k <arg>] [-p <arg>] [-s <arg>] [-v <arg>]
 -a,--attribute <arg>           Attribute name
 -b,--vault-block <arg>       Vault block
 -c,--check-sec-attr            Check whether the secured attribute already exists in the vault
 -e,--enc-dir <arg>             Directory containing encrypted files
 -h,--help                            Help
 -i,--iteration <arg>             Iteration count
 -k,--keystore <arg>           Keystore URL
 -p,--keystore-password <arg>   Keystore password
 -s,--salt <arg>                   8 character salt
 -v,--alias <arg>                 Vault keystore alias
 -x,--sec-attr <arg>            Secured attribute value (such as password) to store

Here is an example:
./vault.sh -s 12345678 -i 50 -k /opt/jboss/security/vault/vault.keystore -p password –v vault -b dataSource –a dataSourceA -x secret -e /opt/jboss/security/vault/

### Note: Ensure that the path of the vault directory in the argument –e has an ending '/'.

### Configure JBoss EAP to use vault
Use the below CLI command to add the vault configuration 
/core-service=vault:add(vault-options={"KEYSTORE_URL" => "/opt/vault/vault.keystore","KEYSTORE_PASSWORD" => "MASK-3y28rCZlcKR","KEYSTORE_ALIAS" => "vault","SALT" => "12438567","ITERATION_COUNT" => "50","ENC_FILE_DIR" => "/opt/vault/"}

This will result in the following entries in the standalone configuration
...
   <vault>
      <vault-option name="KEYSTORE_URL" value="/opt/vault/vault.keystore"/>
      <vault-option name="KEYSTORE_PASSWORD" value="MASK-3y28rCZlcKR"/>
      <vault-option name="KEYSTORE_ALIAS" value="vault"/>
      <vault-option name="SALT" value="12438567"/>
      <vault-option name="ITERATION_COUNT" value="50"/>
      <vault-option name="ENC_FILE_DIR" value="/opt/vault/"/>
    </vault>
    <management>
    ...

The following string can be used in configurations to provide an encrypted password. It should be wrapped with '${' and '}' to denote the string as a variable.
VAULT::ds_ExampleDS::password_name::1

Refer to the below example 
<subsystem xmlns="urn:jboss:domain:datasources:1.0">
    <datasources>
      <datasource jndi-name="java:jboss/datasources/ExampleDS" enabled="true" use-java-context="true" pool-name="H2DS">
       ...
        <security>
          <user-name>sa</user-name>
            <password>${VAULT::ds_ExampleDS::password_name::1}</password>
        </security>
         ...
         
 # IBM MQ in Jboss eap 6.3
 ###  Prerequisites
Before you get started, you must verify the version of the WebSphere MQ resource adapter and understand some of the WebSphere MQ configuration properties.

The WebSphere MQ resource adapter is supplied as a Resource Archive (RAR) file called wmq.jmsra-VERSION.rar. You must use version 7.5.0.0 and higher.

You must know the values of the following WebSphere MQ configuration properties. Refer to the WebSphere MQ product documentation for details about these properties.
MQ.QUEUE.MANAGER: The name of the WebSphere MQ queue manager
MQ.HOST.NAME: The host name used to connect to the WebSphere MQ queue manager
MQ.CHANNEL.NAME: The server channel used to connect to the WebSphere MQ queue manager
MQ.QUEUE.NAME: The name of the destination queue
MQ.TOPIC.NAME: The name of the destination topic
MQ.PORT: The port used to connect to the WebSphere MQ queue manager
MQ.CLIENT: The transport type

For outbound connections, you must also be familiar with the following configuration property:
MQ.CONNECTIONFACTORY.NAME: The name of the connection factory instance that will provide the connection to the remote system
###  Configure & deploy resource adapter
1. If you need transaction support with the WebSphereMQ resource adapter, you must repackage the wmq.jmsra-VERSION.rar archive to include the mqetclient.jar. You can use the following command:
[user@host ~]$ jar -uf wmq.jmsra-VERSION.rar mqetclient.jar
Be sure to replace the VERSION with the correct version number.

2. Copy the wmq.jmsra-VERSION.rar file to the EAP_HOME/standalone/deployments/ directory.

3. Add the resource adapter to the server configuration file.
a. Open the EAP_HOME/standalone/configuration/standalone-full-ha.xml file in an editor.
b. Find the urn:jboss:domain:resource-adapters subsystem in the configuration file.
c. If there are no resource adapters defined for this subsystem, first replace:
<subsystem xmlns="urn:jboss:domain:resource-adapters:1.1"/>
With this
​<subsystem xmlns="urn:jboss:domain:resource-adapters:1.1">
​    <resource-adapters>
​        <!-- <resource-adapter> configuration listed below -->
​    </resource-adapters>
​</subsystem>

d. The resource adapter configuration depends on whether you need transaction support and recovery. If you do not need transaction support, choose the first configuration step below. If you do need transaction support, choose the second configuration step.
A. For non-transactional deployments (if you don't want the resource adapter to do the transaction management for the application), replace the <!-- <resource-adapter> configuration listed below --> with the following:
<resource-adapter>
​    <archive>
​        wmq.jmsra-VERSION.rar
​    </archive>
​    <transaction-support>NoTransaction</transaction-support>
​    <connection-definitions>
​        <connection-definition 
​                class-name="com.ibm.mq.connector.outbound.ManagedConnectionFactoryImpl" 
​                jndi-name="java:jboss/MQ.CONNECTIONFACTORY.NAME" 
​                pool-name="MQ.CONNECTIONFACTORY.NAME">
​            <config-property name="hostName">
​                MQ.HOST.NAME
​            </config-property>
​            <config-property name="port">
​                MQ.PORT
​            </config-property>
​            <config-property name="channel">
​                MQ.CHANNEL.NAME
​            </config-property>
​            <config-property name="transportType">
​                MQ.CLIENT
​            </config-property>
​            <config-property name="queueManager">
​                MQ.QUEUE.MANAGER
​            </config-property>
​            <security>
​                <security-domain>MySecurityDomain</security-domain>
​            </security>
​       </connection-definition>
​    </connection-definitions>
​    <admin-objects>
​        <admin-object 
​                class-name="com.ibm.mq.connector.outbound.MQQueueProxy" 
​                jndi-name="java:jboss/MQ.QUEUE.NAME" 
​                pool-name="MQ.QUEUE.NAME">
​            <config-property name="baseQueueName">
​                MQ.QUEUE.NAME
​            </config-property>
​            <config-property name="baseQueueManagerName">
​                MQ.QUEUE.MANAGER
​            </config-property>
​       <admin-object class-name="com.ibm.mq.connector.outbound.MQTopicProxy"
​                jndi-name="java:jboss/MQ.TOPIC.NAME" pool-name="MQ.TOPIC.NAME">
​            <config-property name="baseTopicName">
​                MQ.TOPIC.NAME
​            </config-property>
​            <config-property name="brokerPubQueueManager">
​                MQ.QUEUE.MANAGER
​            </config-property>
​       </admin-object>
​      </admin-object>
​    </admin-objects>
​</resource-adapter>


B. For transactional deployments(if you want the resource adapter to do the transaction management for the application), replace the <!-- <resource-adapter> configuration listed below --> with the following:


<resource-adapter>
​    <archive>
​        wmq.jmsra-VERSION.rar
​    </archive>
​    <transaction-support>XATransaction</transaction-support>
​    <connection-definitions>
​        <connection-definition 
​                class-name="com.ibm.mq.connector.outbound.ManagedConnectionFactoryImpl" 
​                jndi-name="java:jboss/MQ.CONNECTIONFACTORY.NAME" 
​                pool-name="MQ.CONNECTIONFACTORY.NAME">
​            <config-property name="hostName">
​                MQ.HOST.NAME
​            </config-property>
​            <config-property name="port">
​                MQ.PORT
​            </config-property>
​            <config-property name="channel">
​                MQ.CHANNEL.NAME
​            </config-property>
​            <config-property name="transportType">
​                MQ.CLIENT
​            </config-property>
​            <config-property name="queueManager">
​                MQ.QUEUE.MANAGER
​            </config-property>
​           <security>
​                <security-domain>MySecurityDomain</security-domain>
​            </security>
​            <recovery>
​                <recover-credential>
​                    <user-name>USER_NAME</user-name>
​                    <password>PASSWORD</password>
​                </recover-credential>
​            </recovery>
​        </connection-definition>
​    </connection-definitions>
​    <admin-objects>
​        <admin-object 
​                class-name="com.ibm.mq.connector.outbound.MQQueueProxy" 
​                jndi-name="java:jboss/MQ.QUEUE.NAME" 
​                pool-name="MQ.QUEUE.NAME">
​            <config-property name="baseQueueName">
​                MQ.QUEUE.NAME
​            </config-property>
​            <config-property name="baseQueueManagerName">
​                MQ.QUEUE.MANAGER
​            </config-property>
​        </admin-object>
​        <admin-object class-name="com.ibm.mq.connector.outbound.MQTopicProxy"
​                jndi-name="java:jboss/MQ.TOPIC.NAME" pool-name="MQ.TOPIC.NAME">
​            <config-property name="baseTopicName">
​                MQ.TOPIC.NAME
​            </config-property>
​            <config-property name="brokerPubQueueManager">
​                MQ.QUEUE.MANAGER
​            </config-property>
​        </admin-object>
​    </admin-objects>
​</resource-adapter>
Be sure to replace the VERSION with the actual version in the name of the RAR. You must also replace the USER_NAME and PASSWORD with the valid user name and password.

e. If you don't want to pass RFH2 header in the messages exchanged with MQ broker, add the config-property "targetClient" with value "MQ" in the admin object. Below is an example.

<admin-object 
                class-name="com.ibm.mq.connector.outbound.MQQueueProxy" 
                jndi-name="java:jboss/MQ.QUEUE.NAME" 
                pool-name="MQ.QUEUE.NAME">
            <config-property name="baseQueueName">
                MQ.QUEUE.NAME
            </config-property>
            <config-property name="baseQueueManagerName">
                MQ.QUEUE.MANAGER
            </config-property>
            <config-property name="targetClient">
                MQ
            </config-property>
</admin-object>

f. If you want to change the default provider for the EJB3 messaging system in JBoss EAP 6 from HornetQ to WebSphere MQ, modify the urn:jboss:domain:ejb3:1.2 subsystem as follows:
Replace:

​<mdb>
​    <resource-adapter-ref resource-adapter-name="hornetq-ra"/>
​    <bean-instance-pool-ref pool-name="mdb-strict-max-pool"/>
​</mdb>

   With

​<mdb>
​    <resource-adapter-ref resource-adapter-name="wmq.jmsra-VERSION.rar"/>
​    <bean-instance-pool-ref pool-name="mdb-strict-max-pool"/>
​</mdb>
Be sure to replace the VERSION with the actual version in the name of the RAR.

g. Configure the ActivationConfigProperty and ResourceAdapter in the MDB code as follows:

```java
​@MessageDriven( name="WebSphereMQMDB", 
​    activationConfig =
​    {
​        @ActivationConfigProperty(propertyName = "destinationType",propertyValue = "javax.jms.Queue"),
​        @ActivationConfigProperty(propertyName = "useJNDI", propertyValue = "false"),
​        @ActivationConfigProperty(propertyName = "hostName", propertyValue = "MQ.HOST.NAME"),
​        @ActivationConfigProperty(propertyName = "port", propertyValue = "MQ.PORT"),
​        @ActivationConfigProperty(propertyName = "channel", propertyValue = "MQ.CHANNEL.NAME"),
​        @ActivationConfigProperty(propertyName = "queueManager", propertyValue = "MQ.QUEUE.MANAGER"),
​        @ActivationConfigProperty(propertyName = "destination", propertyValue = "MQ.QUEUE.NAME"),
​        @ActivationConfigProperty(propertyName = "transportType", propertyValue = "MQ.CLIENT")
​    })
​    @ResourceAdapter(value = "wmq.jmsra-VERSION.rar")
​@TransactionAttribute(TransactionAttributeType.NOT_SUPPORTED)
​public class WebSphereMQMDB implements MessageListener {
​}
```
Be sure to replace the VERSION with the actual version in the name of the RAR.         
