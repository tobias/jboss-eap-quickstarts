cmt: Container Managed Transactions (CMT) 
=========================================
Author: Tom Jenkinson  
Level: Intermediate  
Technologies: EJB, CMT  
Summary: The `cmt` quickstart demonstrates Container-Managed Transactions (CMT), showing how to use transactions managed by the container.  
Target Product: JBoss EAP  
Source: <https://github.com/jboss-developer/jboss-eap-quickstarts/>  

## What is it?

The `cmt` quickstart demonstrates using transactions managed by the container in Red Hat JBoss Enterprise Application Platform. It is a fairly typical scenario of updating a database and sending a JMS message in the same transaction. A simple MDB is provided that prints out the message sent but this is not a transactional MDB and is purely provided for debugging purposes.

Aspects touched upon in the code:

1. XA transaction control using the container managed transaction annotations
2. XA access to a PostgreSQL database using the JPA API
3. XA access to a JMS queue

After users complete this quickstart, they are invited to run through the following quickstarts:

1. [jts](../jts/README.md) - The JTS quickstart builds upon this quickstart by distributing the CustomerManager and InvoiceManager
2. [jts-distributed-crash-rec](../jts-distributed-crash-rec/README.md) - The crash recovery quickstart builds upon the [jts](../jts/README.md) quickstart by demonstrating the fault tolerance of JBoss EAP.

_Note: This quickstart uses a `*-ds.xml` datasource configuration file for convenience and ease of database configuration. These files are deprecated in JBoss EAP and should not be used in a production environment. Instead, you should configure the datasource using the Management CLI or Management Console. Datasource configuration is documented in the [Administration and Configuration Guide](https://access.redhat.com/documentation/en-US/JBoss_Enterprise_Application_Platform/) for Red Hat JBoss Enterprise Application Platform._


### What are container managed transactions?

Prior to EJB, getting the right incantation to ensure sound transactional operation of the business logic was a highly specialised skill. Although this still holds true to a great extent, EJB has provided a series of improvements to allow simplified transaction demarcation notation that is therefore easier to read and test. 

With CMT, the EJB container sets the boundaries of a transaction. This differs from BMT (bean managed transactions) where the developer is responsible for initiating and completing a transaction via the methods begin, commit, rollback on a <code>javax.transaction.UserTransaction</code>.

### What makes this an example of container managed transactions?

Take a look at <code>org.jboss.as.quickstarts.cmt.ejb.CustomerManagerEJBImpl</code>. You can see that this stateless session bean has been marked up with the @javax.ejb.TransactionAttribute annotation.

The available options for this annotation are as follows:

* Required - As demonstrated in the quickstart. If a transaction does not already exist, this will initiate a transaction and complete it for you, otherwise the business logic will be integrated into the existing transaction
* RequiresNew - If there is already a transaction running, it will be suspended, the work performed within a new transaction which is completed at exit of the method and then the original transaction resumed. 
* Mandatory - If there is no transaction running, calling a business method with this annotation will result in an error
* NotSupported - If there is a transaction running, it will be suspended and no transaction will be initiated for this business method
* Supports - This will run the method within a transaction if a transaction exists, alternatively, if there is no transaction running, the method will not be executed within the scope of a transaction 
* Never - If the client has a transaction running and does not suspend it but calls a method annotated with Never then an EJB exception will be raised.


System requirements
-------------------

The application this project produces is designed to be run on Red Hat JBoss Enterprise Application Platform 7 or later. 

All you need to build this project is Java 8.0 (Java SDK 1.8) or later and Maven 3.1.1 or later. See [Configure Maven for JBoss EAP 7](https://github.com/jboss-developer/jboss-developer-shared-resources/blob/master/guides/CONFIGURE_MAVEN_JBOSS_EAP7.md#configure-maven-to-build-and-deploy-the-quickstarts) to make sure you are configured correctly for testing the quickstarts.


Use of EAP7_HOME
---------------

In the following instructions, replace `EAP7_HOME` with the actual path to your JBoss EAP installation. The installation path is described in detail here: [Use of EAP7_HOME and JBOSS_HOME Variables](https://github.com/jboss-developer/jboss-developer-shared-resources/blob/master/guides/USE_OF_EAP7_HOME.md#use-of-eap_home-and-jboss_home-variables).


Configure the PostgreSQL Database for Use with this Quickstart
--------------------------------------------------

This quickstart requires the PostgreSQL database. Instructions to install and configure PostgreSQL can be found here: [Configure the PostgreSQL Database for Use with the Quickstarts](https://github.com/jboss-developer/jboss-developer-shared-resources/blob/master/guides/CONFIGURE_POSTGRESQL.md#configure-the-postgresql-database-for-use-with-the-quickstarts)

_Note_: For the purpose of this quickstart, replace the word `QUICKSTART_DATABASE_NAME` with `cmt-quickstart-database` in the PostgreSQL instructions.

1. Be sure to start the PostgreSQL database. Unless you have set up the database to automatically start as a service, you must repeat the instructions "Start the database server" for your operating system every time you reboot your machine.
2. Follow the instructions to [Add the PostgreSQL Module to the JBoss EAP Server](https://github.com/jboss-developer/jboss-developer-shared-resources/blob/master/guides/CONFIGURE_POSTGRESQL.md#add-the-postgresql-module-to-the-red-hat-jboss-enterprise-application-platform-server).
3. Follow the instructions to [Configure the PostgreSQL Driver in the JBoss EAP Server](https://github.com/jboss-developer/jboss-developer-shared-resources/blob/master/guides/CONFIGURE_POSTGRESQL.md#configure-the-postgresql-driver-in-the-red-hat-jboss-enterprise-application-platform-server).


Start the JBoss EAP Server with the Full Profile
---------------

1. Open a command prompt and navigate to the root of the JBoss EAP directory.
2. The following shows the command line to start the server with the full profile:

        For Linux:   EAP7_HOME/bin/standalone.sh -c standalone-full.xml
        For Windows: EAP7_HOME\bin\standalone.bat -c standalone-full.xml
 

Build and Deploy the Quickstart
-------------------------

1. Make sure you have started the JBoss EAP server with the PostgreSQL driver
2. Open a command prompt and navigate to the root directory of this quickstart.
3. Type this command to build and deploy the archive:

        mvn clean install wildfly:deploy

4. This will deploy `target/jboss-cmt.war` to the running instance of the server.

Access the application 
---------------------

The application will be running at the following URL: <http://localhost:8080/jboss-cmt/>.

You will be presented with a simple form for adding customers to a database.

After a user is successfully added to the database, a message is produced containing the details of the user. An example MDB will dequeue this message and print the following contents:

    Received Message: Created invoice for customer named:  Tom

When the same customer name is given, a duplicate warning is given and no JMS-message is send to cause the above message.

The customer name should match: letter & '-', else an error is given. This is to show that a 'LogMessage' entity is still stored in the database thanks to the ```@TransactionAttribute(TransactionAttributeType.REQUIRES_NEW)```
that the method logCreateCustomer in the EJB LogMessageManagerEJB is decorated with. 


Server Log: Expected warnings and errors
-----------------------------------

_Note:_ You will see the following warning in the server log. You can ignore this warning.

    WFLYJCA0091: -ds.xml file deployments are deprecated. Support may be removed in a future version.

You also see the following errors. This is because Hibernate attempts to drop the table and constraints before they are created and without checking first to see if they exist. This issue, <https://hibernate.atlassian.net/browse/HHH-9545>, is fixed in the 4.3 version of Hibernate.  You can ignore these errors.

    HHH000389: Unsuccessful: drop sequence hibernate_sequence
    Sequence "HIBERNATE_SEQUENCE" not found; SQL statement: drop sequence hibernate_sequence [90036-168]


Undeploy the Archive
--------------------

1. Make sure you have started the JBoss EAP server as described above.
2. Open a command prompt and navigate to the root directory of this quickstart.
3. When you are finished testing, type this command to undeploy the archive:

        mvn wildfly:undeploy


Run the Quickstart in Red Hat JBoss Developer Studio or Eclipse
-------------------------------------
You can also start the server and deploy the quickstarts or run the Arquillian tests from Eclipse using JBoss tools. For general information about how to import a quickstart, add a JBoss EAP server, and build and deploy a quickstart, see [Use JBoss Developer Studio or Eclipse to Run the Quickstarts](https://github.com/jboss-developer/jboss-developer-shared-resources/blob/master/guides/USE_JBDS.md#use-jboss-developer-studio-or-eclipse-to-run-the-quickstarts) 

_NOTE:_ Within JBoss Developer Studio, be sure to define a server runtime environment that uses the `standalone-full.xml` configuration file.


Debug the Application
------------------------------------

If you want to debug the source code of any library in the project, run the following command to pull the source into your local repository. The IDE should then detect it.

        mvn dependency:sources

