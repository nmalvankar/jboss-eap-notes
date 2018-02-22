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
