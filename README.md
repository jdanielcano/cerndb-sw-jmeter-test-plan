# cerndb-sw-jmeter-test-plan
JMeter Test Plan for Apache HTTP Server - Shibboleth and Oracle Weblogic Server using Kerberos Auth.

## Project outline

Jmeter is one the most popular tools used for testing web applications, among others. So, the aim of this document is to explain the configuration of Jmeter to test a web application to which has to authenticate. There are several methods to authenticate and to configure Jmeter, but in this case we explain the configuration to follow to configure Jmeter to authenticate using Kerberos.

## Prerequisites.

### Jmeter. Install.

The aim of this document is not to describe how to install JMeter, but in the [official JMeter](http://jmeter.apache.org/usermanual/get-started.html) web page you can find how to do it.

### JMeter. Kerberos pre-configuration.

Every configuration explained here about JMeter refers to the folder where the JMeter is installed. The files named here are those to configure JMeter using Kerberos authentication.

1. **jaas.conf** file. Update the parameter **useKeyTab** to allow the use of KeyTab files.

''' 
useKeyTab=true 
'''

2. **httpclient.parameters** file. Uncomment or add the following line to allow JMeter to use the pre-emptive authentication in httpClient mode. Using this kind of authentication, HttpClient will send the basic authentication response even before the server gives an unauthorized response in certain situations, thus reducing the overhead of making the connection.

'''
http.authentication.preemptive$Boolean=true
'''

3. **system.properties** file. Uncomment or add the following lines to show JMeter which files use for Kerberos authentication and as JAAS file. In this case we can use directly the Kerberos file available in **/etc/krb5.conf** or copy the content of this file into the Kerberos file available in the JMeter bin folder and named as well krb5.conf.

'''
java.security.krb5.conf=/usr/local/bin/jmeter/bin/krb5.conf
java.security.auth.login.config=/usr/local/bin/jmeter/bin/jaas.conf
'''

4. **krb5.conf** file. If we decide to use the one located in the JMeter bin folder, comment all the content of the file and copy the content of the file **/etc/krb5.conf**. It will be like this:

'''
[libdefaults]
default_realm = CERN.CH
ticket_lifetime = 25h
renew_lifetime = 120h
forwardable = true
proxiable = true

[realms]
 CERN.CH = {
   default_domain = cern.ch
     kpasswd_server = cerndc.cern.ch
       admin_server = cerndc.cern.ch
         kdc = cerndc.cern.ch

	   v4_name_convert = {
	        host = {
		         rcmd = host
			             }
				       }
				       }

				       [domain_realm]
				       cern.ch= CERN.CH
				       .cern.ch = CERN.CH
				       </pre>
'''
5. **user.properties** file. This file is used to log all the information about SSL connections, it is not mandatory, but it is advisable to add this parameters in order to record every event in case of problems.

'''
log_level.jmeter.util.HttpSSLProtocolSocketFactory=DEBUG
log_level.jmeter.util.JsseSSLManager=DEBUG
'''

6. **jmeter.properties** file. In this file just check if you are using the proper configuration files.

''' 
httpclient.parameters.file=httpclient.parameters
user.properties=user.properties
system.properties=system.properties
'''

### JMeter. Kerberos configuration.

Once the pre-configuration is ready, is time to open the JMeter application and to configure it to use Kerberos. The steps to take are the following:

1. Add a new **Thread Group**. On the **Test Plan**, right-click and select **Add**, **Threads (Users)** and click on **Thread Group**.

2. Add inside the Thread Group an element to control the cookies. On the **Thread Group**, right-click and select **Add**, **Config Element** and click on **HTTP Cookie Manager**. Keep the default configuration.

3. Add inside the Thread Group an element to control the Kerberos Authentication. On the **Thread Group**, right-click and select **Add**, **Config Element** and click on **HTTP Authorization Manager**. Change the configuration of this element.
  *   Select the element **HTTP Authorization Manager**.
  *   Push the button **Add**.
  *   Add the following information in the new line created in the box title **Authorizations Stored in the Authorization Manager**.
    *   Base URL. Leave this field empty.
    *   Username. Your nice CERN user name without the domain CERN.ch
    *   Password. Your CERN password.
    *   Domain. CERN.
    *   Realm. CERN.CH (Or the name of the realm available in the krb5.conf)
    *   Mechanism. KERBEROS.

With these last steps, the JMeter will be configured to create a test plan using the Kerberos authentication.


### JMeter. Kerberos Test Plan.

#### JMeter. Kerberos Test Plan for a back-end Apache HTTP Server - Shibboleth.

In a test plan in which we want to use Kerberos authentication for an Apache HTTP Server - Shibboleth, we have to add some intermediate steps as follow:

1. The authentication using Kerberos is going to be negotiate just the first time each user access to the test web page, so is needed to add a controller to execute the authentication just once.
  *   Add inside the Thread Group an element to control the cookies. On the **Thread Group**, right-click and select **Add**, **Logic Controller** and click on **Once Only Controller**. Keep the default configuration.

2. Add into the **Once Only Controller** the following configuration elements:
  *   To authenticate, a first access to the Kerberos authentication web page is needed. Add inside the Once Only Controller an HTTP request element to test a web page. On the **Once Only Controller**, right-click and select **Add**, **Sampler** and click on **HTTP Request**. Add the following configuration:
    *   Name: Add an intuitive name.
    *   Server Name or IP: login.cern.ch
    *   Port Number: 443
    *   Implementation: HttpClient4
    *   Protocol: https
    *   Method: GET
    *   Path: /adfs/ls/auth/integrated/?wa=wsignin1.0
    *   Check the options: **Follow Redirects**, **Use KeepAlive**.
    *   All the rest keep the default values.
  *   Access to the web page to test. Add inside the Once Only Controller an HTTP request element to test a web page. On the **Once Only Controller**, right-click and select **Add**, **Sampler** and click on **HTTP Request**.
    *   Name: Add an intuitive name.
    *   Server Name or IP: mytest.web.cern.ch
    *   Port Number: 443
    *   Implementation: HttpClient4
    *   Protocol: https
    *   Method: POST
    *   Path: /mytestweb
    *   Check the options: **Follow Redirects**, **Use KeepAlive**, **Use multipart/for-data for POST** and **Browser-compatible headers**.
    *   All the rest keep the default values.
  *   Add a XPath Extractor to extract some parameters needed for a correct authentication. On this **HTTP Request**, right-click and select **Add**, **Post-Processor** and click on **XPath Extractor**. And add the following configuration:
    *   Reference Name: variable (A random name).
    *   XPath query: //input[@type="hidden"][@name="wresult"]/@value
    *   For all the rest, keep the default values.
  *   Access to the Shibboleth for the federate solution web page. Add inside the Once Only Controller an HTTP request element to test a web page. On the **Once Only Controller**, right-click and select **Add**, **Sampler** and click on **HTTP Request**.
    *   Name: Add an intuitive name.
    *   Server Name or IP: mytest.web.cern.ch
    *   Port Number: 443
    *   Implementation: HttpClient4
    *   Protocol: https
    *   Method: POST
    *   Path: /Shibboleth.sso/ADFS
    *   Check the options: **Follow Redirects**, **Use KeepAlive** and **Browser-compatible headers**.
    *   Parameters to add:
        | Name | Value | Encode | Include Equals |
        | ---- |:-----:| :----: | :------------: |
	| wa   | wsignin1.0 | yes | yes |
	| wresult | ${variable} | yes | yes |
    *   All the rest keep the default values.

3. Add into the **Thread Group**, outside **Once Only Controller**, a new **Sampler** - **HTTP Request** with the final web page to test with the following configuration elements:
  *   Access to the web page to test with the following configuration:
    *   Name: Add an intuitive name.
    *   Server Name or IP: mytest.web.cern.ch
    *   Port Number: 443
    *   Implementation: HttpClient4
    *   Protocol: https
    *   Method: GET
    *   Path: /mytestweb/path-to-test.
    *   Check the options: **Redirect Automatically**, **Use KeeepAlive**, **Use multipart/for-data for POST** and **Browser-compatible headers**.
    *   All the rest keep the default values.

4. For reporting, statistics and status of the services purpose, it is advisable to add the following listening tools:
  *   View Result Tree. Right-click on the **Thread Group**, select **Add**, **Listener** and click on **View Result Tree**.
  *   Graph Results. Right-click on the **Thread Group**, select **Add**, **Listener** and click on **Graph Results**.
  *   Summary Report. Right-click on the **Thread Group**, select **Add**, **Listener** and click on **Summary Report**.
  *   Aggregate Report. Right-click on the **Thread Group**, select **Add**, **Listener** and click on **Aggregate Report.**.

With this configuration, the test plan will be ready to run.


#### JMeter. Kerberos Test Plan for a back-end Oracle Weblogic Server.

TO DO.

### JMeter. Running a Test Plan.
Once the configuration is ready, it is time to run the test plan, deciding previously the number of users and times we want to run the test on the web page to check.

1. Select the **Thread Group** and complete the following fields:
  *   Number of Threads (users): The maximum number of users you want to run.
  *   Ramp-Up Period (in seconds): Defines how long it takes for JMeter to ramp up from zero users to X number of threads.
  *   Loop Count: Defines how many times you want each user to run your script.

As example, the servers we are testing, have a maximum of 256 number of connections allowed, so, to make a test and to check how our application will response to an DOS attack, we define Number of Threads to 500 (users), Ramp Up to 100 (seconds) and Loop Count to 500, our test will start off with zero users and will add 5 users every second, so after 100 seconds we will have a total of 500 users running our script. That is 250000 requests to the web page in around 30 minutes, really short time frame.

2. Push the button "Play" or in the menu select "Run" --> "Start".

### JMeter. Checking the results of a Test Plan.
Once the Test Plan has been ran, to check the result of the tests, just go to the reporting components added in the point 4 of the last section. In this case it was a Summary report, a Graph result, an Aggregate report and a View Results Tree. Navigate on each one to check the results. It is possible to save this into a file.

## Conclusions.
This is just an example about how to test a web application using Kerberos authentication for an Apache HTTP Server using Shibboleth and for an Oracle Weblogic Server. In this project you can find the XLM file for each test.
Any improvement is always welcome.

## Links.

* [Apache JMeter Official Documentation](http://jmeter.apache.org/usermanual/index.html)
* [Apache JMeter Official Documentation](http://jmeter.apache.org/usermanual/get-started.html)
