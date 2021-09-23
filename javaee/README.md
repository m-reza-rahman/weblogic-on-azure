# Basic Java EE CRUD Application
This is a basic Java EE 7 application used throughout the WebSphere on Azure demos. It is a simple CRUD application. It uses Maven and Java EE 7 (JAX-RS, EJB, CDI, JPA, JSF, Bean Validation).

We use Eclipse but you can use any Maven and WebSphere capable IDE. We use PostgreSQL but you can use any relational database such as Db2, SQL Server or MySQL.

## Setup

* IBM Installation Manager
* Repository: http://www.ibm.com/software/repositorymanager/com.ibm.websphere.BASE.v90
* WebSphere 9.
* Install [the 2020-06 release of Eclipse for Enterprise Java Developers](https://www.eclipse.org/downloads/packages/release/2020-06/r/eclipse-ide-enterprise-java-developers) (this is the latest Eclipse IDE version that supports Java SE 8).
* Make sure to use the IBM JDK.
* Run Eclipse as admin.
* * Marketplace -> "websphere" -> IBM WebSphere Application Server V9.x Developer Tools
* Install WebLogic 12.2.1.3 (note - not the latest version) using the Quick Installer by downloading it from [here](https://www.oracle.com/middleware/technologies/weblogic-server-downloads.html).
* Download this repository somewhere in your file system (easiest way might be to download as a zip and extract).
* You will need an Azure subscription. If you don't have one, you can get one for free for one year [here](https://azure.microsoft.com/en-us/free).

## Start Managed PostgreSQL on Azure
We will be using the fully managed PostgreSQL offering in Azure for this demo. Below is how we set it up.

* Go to the [Azure portal](http://portal.azure.com).
* The steps in this section use `<your suffix>`. The suffix could be your first name such as "reza".  It should be short and reasonably unique.
* Select 'Create a resource'. In the search box, enter and select 'Azure Database for PostgreSQL'. Hit create. Select a single server.
* In "Resource group" select "Create new" and enter websphere-cafe-db-group-`<your suffix>`.
* Specify the Server name to be websphere-cafe-db-`<your suffix>`.
* Specify the location to be a location close to you.
* Leave the Version at its default.
* In Compute + Storage click "Configure Server" then choose Basic.
   * Set vCore to the minimum.
   * Set Storage to the minimum.
   * Click 'OK'
* Specify the Admin username to be postgres. 
* Specify the Password to be Secret123! (do not forget the exclamation point). 
* Hit 'Review+create', then 'Create'. It will take a moment for the database to deploy and be ready for use.
* In the portal, go to 'All resources'. Enter `<your suffix>` into the filter box and press enter.
* Find and click on websphere-cafe-db-`<your suffix>`. 
* Under Settings, open the connection security panel.
   * Toggle "Allow access to Azure services" to "Yes".
   * Toggle "Enforce SSL connection" to "DISABLED". 
   * Hit Add client IP. This allows connection to the database from the IP you are currently using to access Azure.  As a precaution, verify the IP entered is actually your IP.  You can do this by googling "what is my ip".  Click Save.

## Setting Up WebLogic
The next step is to get the application up and running. Follow the steps below to do so.
* Start Eclipse.
* Go to the 'Servers' panel, secondary click. Select New -> Server
* Select Oracle -> Oracle WebLogic Server Tools. Click next. Accept the license agreement, click 'Finish'.  Eclipse may ask to be restarted.  If so, comply with the request.
* After the Eclipse WebLogic adapters are done installing, go to the 'Servers' panel again, secondary click. 
   * Select New -> Server -> Oracle -> Oracle WebLogic Server. 
   * Choose the defaults and hit 'Next'. 
   * Enter where you have WebLogic installed.  Even though the dialog says "WebLogic home" you may have to enter the full path to the `wlserver` directory.
   * Enter where the Oracle JDK is installed.  Click next. 
   * For the domain directory, hit Create -> Create Domain. 
   * For the domain name, specify 'domain1'. Hit 'Finish' to add the new server to Eclipse.  If Eclipse asks to create a master password hint, do so.  Consider using `<your suffix>` for the questions and answers.
   * Copy the PostgreSQL driver to where you have WebSphere installed under AppServer/lib. This file is located in the javaee/server directory where you downloaded the application code.
* Go to the 'Servers' panel, secondary click on the registered WebSphere instance and select Start if the server is not started already.  If the server does not start, put aside this workshop and troubleshoot why the server did not start.  Once the server is successfully started from Eclipse, you may continue.

## Connect WebLogic to the PostgreSQL Server

* Once WebSphere starts up, go to https://localhost:9043/ibm/console/ and log onto the console.  
   * Click on Resources -> JDBC -> Data sources. Select the scope to be the server. Select New. 
   * Enter the name as 'WebSphereCafeDB' and the JNDI name as 'jdbc/WebSphereCafeDB'. Click next.
   * Select 'Create new JDBC provider' and click next.
   * Select the database type as 'User-defined' and the implementation class as 'org.postgresql.ds.PGConnectionPoolDataSource'. Click next.
   * Enter the full path to the PostgreSQL driver (for example: /opt/IBM/WebSphere/AppServer/lib/postgresql-42.2.23.jar). Click next.
   * Accept the defaults and click next until you click Finish.
   * Click 'Save' to sync with the master configuration. 
   * Go to Resources -> JDBC -> Data sources. Click 'WebSphereCafeDB'. Click on 'Custom properties'.
   * Change the URL property to be:
   ```
   jdbc:postgresql://websphere-cafe-db-`<your suffix>`.postgres.database.azure.com:5432/postgres?user=postgres@websphere-cafe-db-`<your suffix>`&password=Secret123!
   ```
   * Click 'Save' to sync with the master configuration.
   * Go to Resources -> JDBC -> Data sources. Click 'WebSphereCafeDB'. Click on 'Test connection'.
   * Make sure the test succeeds. If it does not, put these instructions aside, troubleshoot and resolve the issue.  Once the connection successfully tests, you may continue. You will see a warning about the GenericDataStoreHelper being used. You can safely ignore this warning.

## Open websphere-cafe in the IDE
* Get the websphere-cafe application into the IDE. In order to do that, go to File -> Import -> Maven -> Existing Maven Projects.  Click Next
* Then browse to where you have this repository code in your file system and select javaee/websphere-cafe and click "Open".  
* Accept the rest of the defaults and click "finish".
* Once the application loads, you should do a full Maven build by going to the application and secondary clicking -> Run As -> Maven install.
   * You must see `BUILD SUCCESS` in the Eclipse console in order to proceed.  If you do not, troubleshoot the build problem and resolve it.  Once the application has successfully built, you may continue.

## Deploying the Application

* It is now time to run the application. Secondary click the application -> Run As -> Run on Server.
   * Select the local WebSphere instance.
   * Make sure to select "Always use this server when running this project" and click "Finish". Just accept the defaults and wait for the application to finish deploying.
* Once the application runs, Eclise will open it up in a browser. The application is available at http://localhost:7001/websphere-cafe.

## Exploring the Application

The application is composed of:

- **A RESTFul service*:** protocol://hostname:port/websphere-cafe/rest/coffees

	- **_GET by Id_**: protocol://hostname:port/websphere-cafe/rest/coffees/{id} 
	- **_GET all_**: protocol://hostname:port/websphere-cafe/rest/coffees
	- **_POST_** to add a new element at: protocol://hostname:port/websphere-cafe/rest/coffees
	- **_DELETE_** to delete an element at: protocol://hostname:port/websphere-cafe/rest/coffees/{id}
	
- **A JSF Client:** protocol://hostname:port/websphere-cafe/index.xhtml

Feel free to take a minute to explore the application.

## Cleaning Up

Once you are done exploring all aspects of the demo, you should delete the websphere-cafe-db-group-`<your suffix>` resource group. You can do this by going to the portal, going to resource groups, finding and clicking on websphere-cafe-db-group-`<your suffix>` and clicking delete. This is especially important if you are not using a free subscription! If you do keep these resources around (for example to begin your own prototype), you should in the least use your own passwords and make the corresponding changes in the demo code.
