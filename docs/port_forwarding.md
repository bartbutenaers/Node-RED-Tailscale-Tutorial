# Avoid port forwarding

The easiest way to access your Node-RED system via internet, is by setting up port forwarding.  But that is a really ***bad idea***, which we will try to explain below.

By default all the ports on your modem/router will be closed to make sure nobody can access devices on your home network from the internet.  As a result, you also won't be able yourself to access Node-RED from the internet.  

To explain how port-forwarding can give you access to Node-RED from the internet, we will explain how you can access Node-RED.  The Node-RED ExpressJs webserver is listening on one of more ports for incoming requests:
+ Node-RED listens by default on port 1880 a.o. for incoming requests about the flow editor and dashboard.
+ The node-red-contrib-google-smarthome node listens to port 3001 for incoming voice commands from the Google servers.
+ And so on...

You can access directly these ports from within you local home network.  However to allow access from the internet on these ports, you need to open ports on your modem/router and forward the requests to these ports on the host device (e.g. Raspberry Pi) where Node-RED is running:

![image](https://github.com/bartbutenaers/Node-RED-security-basics/assets/14224149/2e85f777-7fed-4fd0-aa63-e5124d04993a)

This setup is very easy and works fine: you can access your Node-RED web application with your browser, and you can send voice commands to Node-RED with your Google Home device.

However as soon as a port has been opened on your modem/router, malicious bots will start scanning these ports within minutes.  Once an open port has been detected by these bots, hackers will know about it and have direct access to your Node-RED logon screen.

You will find on the Node-RED Discourse community some best practices to make your port forwarding setup a bit more secure:
+	Always make sure you have at least activated basic authentication, i.e. a logon screen where you have to enter username and password.
+	Use a difficult to guess username and password, instead of well-known usernames like 'pi' or 'admin'.  That way hackers not only have to guess the password, but they also have to guess the username.
+	Use a different port instead of 1880, especially a not well-know port.  That way hackers that are focussed on Node-RED systems (and thus scanning for port 1880), won’t find your open port that easily.
+	Use SSL/TLS (i.e. https) to make sure that the data you send across the internet is encrypted, so that hackers who intercept your data cannot read it.
+	Limit the list of IP addresses that can access Node-RED.
+	Use client certificates to authenticate your own devices in Node-RED.
+ And so on…

This are all very good guidelines, which we will be discussed in more detail later on!  

However those guidelines won’t solve all our problems, because the hackers have direct access to your Node-RED login screen:
+	Hackers use brute force algorithms to guess your password, by trying a massive amount of different passwords.
+	Hackers can determine (a.o. based on the source of your logon screen) which NodeJs and ExpressJs webserver versions you are running, and look for security vulnerability issues within that version.  That allows them to exploit these issues to gain access to your system.
+	Hackers can start a DDOS attack (Denial of service) by sending a massive amount of http requests from a large farm of hacked machines to your Raspberry.  That way you will not be able to access your Node-RED anymore, and Node-RED can even crash.
+	…

Since a lot of Node-RED users (using port forwarding) have been hacked already, it is adviced to avoid using port forwarding!  As a result, ALL ports should be closed on your modem/router!  
