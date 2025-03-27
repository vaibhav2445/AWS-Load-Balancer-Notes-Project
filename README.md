# AWS-Load-Balancer-Notes-Project

Project Description:
This project demonstrates the implementation of an AWS Application Load Balancer (ALB) to efficiently route traffic between multiple EC2 instances based on URL path rules. It enhances high availability, security, and scalability for web applications by distributing requests across multiple Availability Zones (AZs).

Project Starts Here :- 
Step 1:- Create 2 Ec2 instances in 2 different Availability zones (AZ's) within same region.
EC2 --> Give Name as "Test EC2 1a" 
	--> AMI 
		-->KP 
			--> In Netowrk setting select subnet as per 1st AZ for e.g 
              		subnet = subnet-0603aa0dd1018f970 AZ :- ap-south-1a 
			--> Select Security Group (SG )We have given Same SG  to both instances named as "ec2sg".
              		 ---> Add User data (given below)
                                #!/bin/bash
				yum update -y
				yum install httpd -y
				systemctl start httpd
				systemctl enable httpd
				echo "Hello World from $(hostname -f)" > /var/www/html/index.html

Launch Instance named as "Test EC2 1a"

Create another instance by name "Test EC2 1b"
by repeating above steps same and just select different subnet
  for eg. subnet-006d8c52d39e28865 and Availability zone  "ap-south-1b" and add same user data.
  
and launch the instance called "ap-south-1b".
And check the instance are running properly or not by copying their public ip addresses in new tabs.

Step 2 :- 
Go to Load Balancers in EC2 dashboard 
Creat Load Balancer --> 
   Select "Application Load Balancer " 
      --> Give Name "DemoALB" 
	--> Select Scheme "Internet Facing"
            -->  Select Load balancer IP address type as "Ipv4" 
              --> Network Mapping Keep VPC Default  and select all "Availability Zones and 
                     subnets"Here we are selecting all -->Select Security Group for LB  For now select 
                     anyone here we selected "launch-wizard-1" will do configuration for it later 
                 -->Listeners and routing (Here explain Listeners and routing Concepts  in short) --> 
                      For Protocol HTTP Port 80 (Explain What is port ? in short ) Select Default Action 
                      by selecting target group -->
              Click "Create Target Group " (First explain What is Target Group In Load Balancers ?) --> 
                ----> (Explain Target Group in short )Specify group details Select "Instances"
                     ------>  Give Name as "TG1"
                            ----> Select protocol ,port , VPC and Protocol version
                                ---->  Health checks  (what is  Health Checks? in short)
                                           Health check protocol
                                           Health check path
                                    ----> Click "Next"
                                      ----- > Register targets 
                                      ---------> Select above created two instances and explain "Ports for the 
                                                       selected instances"  Ports for routing traffic to the selected 
                                                        instances. 
                                                         Click "Include as pending below"
                                                        
                                           ----------->Click  "Create Target Group"
Now go back to "Listeners and routing" in LB and refresh and add TG1 as target group for HTTP
------> Summary Check all Configuration of LB 
       -----> Click "Create Load Balancer"

Now we have two security groups (SG), one is on load balancer and one is on instances 
Explain their responsibilities in short.

Now we have to configure SG of instances so that they will only the traffic coming from SG of 
load balancer "DemoALB".

Go to EC2 Dashoard ---->
    --->Select SG herer "ec2sg"
       ---> Edit Inbound Rules
          ---> Add Rule
             ----> Type "All Traffic" --> Source "Custom"--> Select SG of Load Balancer i.e "launch- 
                       wizard-1" 
                       and delete above all 3 rules (explain the reason in short)
                        --> Click "Save Rules"
Now you can check the "Target Group" Details to see healthy targets now it will be two 
You can also check "DemoALB" Load balancer to check status "Active or Provisioning "


Copy the "DNS Name" of Load Balancer and hit the URL

Hit the URL Again to check on which EC2 the traffic is going 

Now you can see that the traffice is distributing equally between two EC2 instances.

Now you can the traffic is shuffling between two instances which are created in different AZ's but in same region.

This is called as High availability which is a feature AWS Load Balancer.
Now you just to provide the URL of "DemoALB" to the users of application.
we only configured the one Target Group to our Load Balancer which has only Listner Rule which performs actions 
"Forward to target group
TG1 : 1 (100%)
Target group stickiness: Off"
(Explain Above In short)

Now we will some Advanced Use Cases for Load Balancer : - 
Go to EC2 Dashboard ---->
    ----->Select EC2 "Test EC2 1b"
          -----> Connect To EC2 (need to Assign SSH connection again in SG because we deleted firstly )
               -----> Use Following Commands
                           cd /var/www/html
                           ls -lstr
                           sudo mkdir login
                           sudo mv index.html login/

Here we just changed the Path of "Index.html" of  "Test EC2 1b" .
Now In Target Group Dashboard you can we see in "TG1" the health check will fail for EC2 "Test EC2 1b".

You can see on below image Load Balancer don't know that we changed the path of our web application (index.html) t0 "/var/www/html/login/index.html" for EC2 "Test EC2 1b" but still its redirecting you to old path "/var/www/html"


Now we have to Intelligence to the Load Balancer that if we accessed ou Web app from new path then it will redirect to correct Ec2.                         
Now From TG1 target group Deregister the unhealthy EC2 which is "Test EC2 1b" from "TG1".    
Explain Draining In AWS Loadbalancer ? (in SHort)

Create New Target Group --->
   --->  Name as "loginTG"            
     ---> keep IP address type = IPv4
       -----> VPC = HTTP1
            -----> Health Checks 
                              The associated load balancer periodically sends requests, per the settings 
                               below, to the registered targets to test their status.
                               Health check path
                               ---> "Health check protocol" as "HTTP"
                                   ---> Use the default path of "/" to perform health checks on the root, or 
                                            specify a custom path if preferred.
                                          Now here specify the new path as "/login/"
                           ----> Click "Next"
Now Include EC2 "Test EC2 1b" to "Include as pending below" 
 ----> Click "Create Target Group"
Now you have Target Groups "TG1"  which has "Test EC2 1a "And "loginTG" which has "Test EC2 1b"
Now we have to create a New Listener Rule for Our Load Balancer for path "/login/"
Go To Listener  ---> 
----> Add Rule
    ----> Add Name 
     -------> Define rule conditions 
              -----> Add Condition 
                    ----> Select "Path"
                        -----> Add Path
                                 -------> Next
    ---------> Define rule actions
          ------> Actions --> Forward to Target Group
                   ---> Select "Target Group" as "loginTG"
                    ------> Click "Next" 
                        
    ----------> Set rule priority 
            --------> Add Priority. Here we are adding "1".
                ------> Click "Next" 
                        ------> Review and create

Now we have 2 rules in a Listener and rules will get executed from top to bottom.

Now Hit the URL of Load Balancer "DemoALB" 
Now you see load balancer redirecting us to EC2 "Test EC2 1a"

Now Hit the URL of Load Balancer "DemoALB"  and add "/login/" at the end .
Now you see load balancer redirecting us to EC2 "Test EC2 1b"

To get More Clear Idea of this you can Resource Map fpr "DemoALB" Load Balancer

So the ALB uses Round Robin Algorithm.
Explain Round Robin Algorithm regarding to the Application load balancer in AWS.



                               


