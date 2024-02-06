# Multi stack tomcat project with AWS Service


 ![](https://i.imgur.com/LpIwm8b.png)

1. First we create the Security Group in our AWS account
	- Let's start with Application security group
        	So here we will open SSH connection and port 8080 for our Tomcat
        	![](https://i.imgur.com/iRx5pe7.png)
       	 
	- Now let's create a security group for all the other services to communicate between each other, that include tomcat, Memcache, RabbitMQ. Note that it's only allowed from the same security group where they do belong.
        	![](https://i.imgur.com/w4dzuYx.png)
       	 
	- Lets create the security group for Load Balancer. So it simply allow port 80 and 443 from anywhere.
        	![](https://i.imgur.com/JvnyyFV.png)
       	 
	- Final view
        	![](https://i.imgur.com/07kBcmj.png)
       	 
2. Then we create or use or Key pair to login in the machine
3. Now launch a EC2 instance for MySQL DB server using Redhat linux
	- Take a RedHat AMI base on any linux and put the user data script from the `userdata` folder in the source. So this basically install the MySQL server create the DB and do the configuration part
	- Make sure you will attach a public IP and same Key pair that you created last time
	- Use the `vprofile-backend-sg` as security group
    
4. Launch another EC2 instance for the RabbitMQ server using Redhat linux
	- Again, take a RedHat AMI base from any linux and put the user data script from the `userdata` folder in the source. So this basically install the RabbitMQ and do the configuration part
	- Make sure you will attach a public IP and same Key pair that you created last time
	- Use the `vprofile-backend-sg` as security group
    
5. Launch another EC2 instance for the Memcache server using Redhat linux
	- Again, take a RedHat AMI base from any linux and put the user data script from the `userdata` folder in the source. So this basically install the Memcache and do the configuration part
	- Make sure you will attach a public IP and same Key pair that you created last time
	- Use the `vprofile-backend-sg` as security group
	- So by now you have 3 EC2 instance running
    
	![](https://i.imgur.com/BQikNya.png)
    
6. Route53. We need this service because our internal stack needs to communicate between each other that are running in different machines, and those machines have dynamic IP that we can't hard code in the application. So if you see the application property file.
    	![](https://i.imgur.com/6E5lNMR.png)
   	 
	Ok, create a private hosted zone and add the following CNAME records
    	![](https://i.imgur.com/ZyfHYNN.png)
   	 
	Please note here i add private IP for my EC2 machine of MySQL, RabbitMQ, Memcache

7. Now create the Tomcat server using Ubuntu linux
	- Take a Ubuntu AMI base on any linux and put the user data script from the `userdata` folder in the source. So this basically install the Tomcat9 and do the configuration part
	- Make sure you will attach a public IP and same Key pair that you created last time
	- Use the `vprofile-backend-sg` as security group
	- Now we need to bring the source artifact that we will host here. But before that we need to build it and make it in `.war` format. So you can do it locally or spin a EC2 instance to build the `.war` artifact. I do it locally in my VM and `scp` the artifact into this app server
	- If you want to do it locally check my other [project](https://github.com/roychandrasekhar/multi-stack-tomcat-project-in-vagrant).
	- Let's assume you have scp in the `/tmp/` folder
	- ssh into the machine
    	```bash
    	sudo su
    	sudo systemctl stop tomcat
    	sudo rm -rf /usr/local/tomcat/webapps/ROOT*
    	sudo cp vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
    	sudo chown tomcat.tomcat /usr/local/tomcat/webapps -R
    	sudo systemctl stop tomcat
    	sudo systemctl restart tomcat
    	```
    
8. Test the Application server using the public IP of the Tomcat server
	![](https://i.imgur.com/tXXrrmu.png)
	Make sure you will add port 8080 for now.
    
    
9. Now create the Target group
	- Create a new target group as given below in the screenshot
	![](https://i.imgur.com/EPXCZnP.png)
    
	- Once done add your `vprofile-app01` as target
	![](https://i.imgur.com/OzlkCGh.png)
    
10. Now create the Application Load Balancer
	- That will listen on port 80 and forward the request to our target group
	![](https://i.imgur.com/dTFkdd0.png)
    
	- Once its get provisioned copy the DNS name of the load balancer and check in your browser
	![](https://i.imgur.com/2nrIc59.png)
    
11. Now finally you can purchase/book a domain and in the name server to host this application with some simple URL name.


