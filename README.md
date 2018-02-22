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
