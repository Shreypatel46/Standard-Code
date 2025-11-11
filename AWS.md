# EC2 WITH AUTO SCALE

 Network settings:

·        Under Firewall (security groups), click Create security group.

·        Security group name: my-web-security-group.

·        Description: Allows HTTP and SSH traffic.

·        Inbound security groups rules: You need to add a rule to allow web traffic. Click Add security group rule.

·        Type: Select HTTP.

·        Source type: Select Anywhere. This allows anyone on the internet to see your web page.

·        The default SSH rule should already be there. Leave it.

Auto Scaling Setting :

1)Configure Network:

·        VPC: Leave the default VPC selected.

·        Availability Zones and subnets: Select at least two different subnets from the list (e.g., one ending in us-east-1a and another in us-east-1b

2) Configure Scaling:

For this lab, you can skip the Load Balancer section. Select No load balancer. Click Next.

Group size: This is where you define how many instances to run.

·        Desired capacity: 2 (Start with two instances).

·        Minimum capacity: 1 (Never have fewer than one instance).

·        Maximum capacity: 4 (Never have more than four instances).

Scaling policies

·        Select Target tracking scaling policy.

·        Metric type: Choose Average CPU utilization.

·        Target value: 50.

# Serverless compute : S3 + lambda

Create a Function:

·        Click the Create function button

·        Select the Author from scratch option.

·        Function name: myS3FileLogger.

·        Runtime: Select Python 3.11 (or a similar recent Python version). The "runtime" is the programming language environment.

·        Architecture: Leave the default x86_64.

·        Permissions: Every Lambda function needs permission to run and interact with other AWS services

·        Expand the "Change default execution role" section

·        Select Create a new role with basic Lambda permissions. This automatically creates a role that allows your function to write logs, which is all we need for now.

·        Click Create function

we connect the two services, telling S3 to trigger our Lambda function.

·        Add a Trigger: In your Lambda function's main page, in the "Function overview" diagram, click + Add trigger.

·        Configure the Trigger:

·        Select a source: In the dropdown, choose S3.

·        Bucket: Select the bucket you created earlier (yourname-midterm-lambda-uploads).

·        Event type: Leave the default All object create events. This means the function will fire whenever a new object is put into the bucket

·        Check the acknowledgment box at the bottom that says I acknowledge that using the same S3 bucket for input and output is not recommended...

·        Click Add.

# Auto Load Balancer Lab4

**Phase -1**

**Step-1 Create a Security Group** :

Inbound rules: Click Add rule. We need two rules

Rule 1 (SSH):

·        Type: Select SSH.

·        Source: Select My IP. AWS will automatically fill in your current IP address. This is a security best practice, ensuring only you can manage the server.

Rule 2 (HTTP):

·        Type: Select HTTP.

·        Source: Select Anywhere (0.0.0.0/0). This allows anyone on the internet to access our web pages.

**Step 2: Create a ec2 Launch Template**

Advanced details section - User data box

```#!/bin/bash

yum update -y

yum install -y httpd

systemctl start httpd

systemctl enable httpd

echo "Hello from $(hostname -f)" > /var/www/html/index.html
```

**Step 3: Launch Instances in Different Availability Zones**

Click the Actions button and select Launch instance from template.

We'll now configure the first instance.

·        Number of instances: Keep this at 1.

·        Network settings: Look for the Subnet setting. Click the dropdown menu. You will see a list of subnets, each corresponding to a different Availability Zone (e.g., us-east-1a, us-east-1b, etc.). Select any one of them (e.g., us-east-1a).

Click Launch instance.

You'll be taken to a success screen. Click on the instance ID link to view your new instance in the "Instances" dashboard. You can rename it WebApp-Server-1 for clarity.

Repeat the process for the second instance:

·        Go back to Launch Templates, select your WebApp-Template again

·        Click Actions -> Launch instance from template.

·        This time, under Network settings -> Subnet, CHOOSE A DIFFERENT AVAILABILITY ZONE from the one you chose for the first instance (e.g., select us-east-1b).

·        Click Launch instance.

Go to your "Instances" dashboard. You should now see two instances running. You can rename the second one WebApp-Server-2.

**Phase -2**

**Step 1: Create a Target Group**

Basic configuration:

·        Target group name: WebApp-TG.

·        Protocol: HTTP, Port: 80. This means the ALB will forward traffic to your instances on port 80.

·        VPC: Ensure your default VPC is selected.

Register targets:

·        You will see the two instances you created (WebApp-Server-1 and WebApp-Server-2).

·        Check the box for each instance.

·        Click the Include as pending below button.

**Step 2: Create the Application Load Balancer**

Network mapping:

·        VPC: Your default VPC should be selected.

·        Mappings: This is critical for high availability. You must select at least two Availability Zones. Choose the exact same two AZs where your EC2 instances are running (e.g., us-east-1a and us-east-1b).

Security groups:

·        Remove the default security group.

·        Click Create new security group. A new browser tab will open.

·        Security group name: WebApp-ALB-SG.

·        Description: Allows web traffic to the ALB.

·        Inbound rules: Click Add rule.

·        Type: HTTP.

·        Source: Anywhere (0.0.0.0/0).

·        Click Create security group.

·        Go back to the load balancer creation tab. Click the refresh icon next to the security group dropdown and select your new WebApp-ALB-S

Listeners and routing:

·        This section defines how the ALB handles incoming requests

·        The default listener should be Protocol: HTTP and Port: 80.

·        For the Default action, click the dropdown and select your WebApp-TG target group.

**Step 3: Update Your EC2 Security Group**

·        Go back to Security Groups in the EC2 menu.

·        Select your instance security group, WebApp-SG.

·        Click the Inbound rules tab, then click Edit inbound rules.

·        Find the HTTP rule where the Source is Anywhere (0.0.0.0/0) and click Delete.

·        Click Add rule.

·        Type: HTTP.

·        Source: Instead of an IP range, click in the custom source box and start typing WebApp-ALB-SG. Select the security group ID for your ALB's security group.

·        Click Save rules.

**Phase -3**

**Step 1: Configure Health Check Settings**

·        the AWS EC2 Dashboard, go to your Target Group, WebApp-TG.

·        Select the Health checks tab.

·        Click the Edit button.

·        We will adjust some settings to make our system react faster for this demonstration.

·        Health check path: This should be / by default, which is correct. The ALB will request the index.html page.

Advanced health check settings:

·        Healthy threshold: 2. The number of consecutive successful checks required to consider an instance healthy.

·        Unhealthy threshold: 2. The number of consecutive failed checks to consider an instance unhealthy.

·        Timeout: 5 seconds. How long to wait for a response before it's considered a failure.

·        Interval: 10 seconds. How often to perform a health check on each instance.

**Step 2: Simulate an Instance Failure**

·        Go to the EC2 -> Instances dashboard.

·        Select one of your instances (e.g., WebApp-Server-1).

·        Click the Connect button at the top right

·        In the connection options, select the EC2 Instance Connect tab and click Connect. A new browser tab with a command line terminal will open.

·        Once you're connected, you are "inside" your server. Type the following command to stop the Apache web server and press Enter:

·        sudo systemctl stop httpd

·        observe  health status

·        For recovery : sudo systemctl start httpd

**Phase-4**

**Step 1: Prepare the Environment**

·        Go to the EC2 -> Instances dashboard.

·        Select both of your running instances (WebApp-Server-1 and WebApp-Server-2).

·        Click Instance state -> stop instance.

·        If you now go to your Target Group, you will see that it has zero instances, and they are all in the "draining" state.

**Step 2: Create the Auto Scaling Group**

·        In the EC2 Dashboard, scroll to the bottom of the left-hand menu and click Auto Scaling Groups (under "Auto Scaling").

·        Click Create Auto Scaling group.

_Step 1: Choose launch template or configuration_

·        Auto Scaling group name: WebApp-ASG.

·        Launch template: Select your WebApp-Template from the dropdown menu.

·        Click Next.

_Step 2: Choose instance launch options_

·        Network (VPC): Your default VPC should be selected.

·        Availability Zones and subnets: Select the same two subnets in different Availability Zones that you've been using all along (e.g., us-east-1a and us-east-1b). The ASG needs to know where it's allowed to create new servers.

·        Click Next.

_Step 3: Configure advanced options_

·        Load balancing: Select Attach to an existing load balancer.

·        Choose Choose from your load balancer target groups and select your WebApp-TG

·        Health checks: Check the box for Enable ELB health checks. This tells the ASG to also consider the load balancer's health checks when determining if an instance is healthy.

·        Click Next.

_Step 4: Configure group size and scaling policies_

Group size: This is where you set the desired capacity.

·        Desired capacity: 2. The ASG will immediately try to launch 2 instances.

·        Minimum capacity: 1. The ASG will never terminate instances if it would go below 1.

·        Maximum capacity: 4. The ASG will never create more than 4 instances

Scaling policies: This is the brain of the ASG.

·        Select Target tracking scaling policy.

·        Metric type: Average CPU utilization.

·        Target value: 50. This means we want our servers to average around 50% CPU usage.

·        Click Next.

Click Create Auto Scaling group.

**Phase-5**

**Section A: Configure Path-Based Routing**

**Step 1: Modify the ALB Listener Rules**

1.      Navigate to your Load Balancer (WebApp-ALB), and click on the Listeners tab.

2.      You will see a listener for HTTP:80. Click the View/edit rules link for this listener.

3.      You are now on the rules management page. You'll see one existing rule, the [Default] one. We will insert new rules that get evaluated before the default.

4.      Click the + icon (Insert Rule) on the menu bar.

Create the Authentication Rule:

·        Click Add condition -> Path.

·        Enter the path pattern /auth/*. The * is a wildcard, so this rule will match any URL like /auth/login, /auth/user, etc.

·        Click the checkmark to confirm the condition.

·        Click Add action -> Return fixed response.

·        Response code: 20

·        Content-Type: text/plain

·        Response body: This is the Authentication Service.

·        Click the checkmark to confirm the action.

·        At the top, make sure the rule's priority is set to 1 (or the lowest number).

Create the Order Processing Rule:

·        Click the + icon (Insert Rule) again.

·        Condition: Add a Path condition for /order/*.

·        Action: Add a Return fixed response action.

·        Response code: 200

·        Content-Type: text/plain

·        Response body: This is the Order Processing Service.

·        Set this rule's priority to 2.

Finally, click Save at the top. Your rule set should now have three rules: priority 1 for /auth/*, priority 2 for /order/*, and the last one for the default action.

Finally test /auth & /order path by visiting

**Section B: Integrate with Route 53 for Global Routing**

**Step 1: Create a Hosted Zone**

·        In the AWS Console, search for and navigate to Route 53.

·        On the left menu, click Hosted zones.

·        Click Create hosted zone.

·        Domain name: Enter the domain name you own (e.g., my-aws-project.com).

·        Type: Select Public hosted zone.

·        Click Create hosted zone.

**Step 2 Create a DNS Record to Point to the ALB**

·        Click on your newly created hosted zone to view its details.

·        Click Create record.

·        Record name: You can leave this blank to point your root domain (e.g., my-aws-project.com) or enter a subdomain like app.

·        Record type: A – Routes traffic to an IPv4 address and some AWS resources.

·        Enable the Alias toggle.

Route traffic to:

·        In the first dropdown, choose Alias to Application and Classic Load Balancer.

·        In the second dropdown, choose the AWS Region you are using (e.g., us-east-1).

·        In the third dropdown, your load balancer WebApp-ALB should appear. Select it.

Click Create records.

**To make your application truly global and use this feature, you would have to:**

·        Deploy a Second Stack: Replicate your entire setup (EC2 instances in an ASG, and an ALB) in a different AWS region, for example, eu-west-1 (Ireland).

·        Create a Second DNS Record: Go back to Route 53 and create a second record with the exact same name (e.g., app.my-aws-project.com).

·        Configure Latency Routing: For this second record, you would again toggle Alias, but this time point it to the new ALB in the Ireland region. Most importantly, for both the first and second records, you would set the Routing policy to Latency.

# Analyse API activity with cloud trail

STEP 1: -

Go to CloudTrail Console

Choose:

•       Trail name: MySecurityTrail

•       Apply trail to All regions o Store logs in a new S3 bucket

•       Enable CloudWatch Logs Integration (optional)

•       Enable management events log

•       Create trail

STEP 2: -

Perform a few activities in your AWS account:

• Launch or stop an EC2 instance.

• Create an S3 bucket

STEP 3: -

• Go back to CloudTrail.

• Click Event history on the left menu.

• We will now see a list of recent events from our account.

STEP 4: -

Click on any event to view full details for following

•       Who performed the action

•       Time of action

•       Source IP address

•       Affected resource

# Configure Cloud Watch for EC2

**Step 1: Launch an EC2 Instance**

•       Security group: Allow SSH (port 22), and optionally HTTP (port 80)

•       Click Launch Instance

**Step 2: Install and Configure CloudWatch Agent**

Connect to the EC2 instance using SSH, then run:

•       sudo yum install amazon-cloudwatch-agent -y

•       This installs the agent.

•       sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-configwizard

•       This  generate config file with question in shell

Question expected answer

•       Install agent as root? → yes

•       Format: json

•       Collect CPU stats? → yes

•       Collect memory metrics? → yes

•       Collect disk metrics? → yes

•       Log file collection? → no (optional)

•       Do you want to turn on StatsD daemon -> NO

•       Do you want to monitor metrics from CollectD -> NO

•       Do you want to monitor any host metrics -> YES

•       Do you want to monitor cpu metrics per core -> YES

•       Do you want to add ec2 dimensions into all of your metrics if the info is available -> NO

•       Do you want to aggregate ec2 dimensions (InstanceId) -> NO

•       Would you like to collect your metrics at high resolution (sub-minute resolution) -> 60sec

•       Which default metrics do you want -> Basic

•       Do you have any existing cloudwatch log agent -> NO

•       Do you want to monitor any log file -> YES

**Step 3: Start the CloudWatch Agent**

Once configuration is done, start the agent with:

sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl

-a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json

-s

This fetches our monitoring configuration and starts sending metrics from

our EC2 instance to CloudWatch.

**Step 4: Verify Metrics in CloudWatch**

•       Go to AWS Console → CloudWatch → Metrics

•       Click EC2 metrics

•       We should see:

·        CPU Utilization

·        mem_used_percent (Memory)

·        disk_used_percent (Disk)

**Step 5: Create a CloudWatch Alarm**

•       Go to CloudWatch Console → Alarms → Create Alarm

•       Select metric: CPU Utilization

•       Set condition:

o   Greater than 70%

o   For 2 consecutive 5-minute periods

•       Notification:

o   Create or select SNS Topic

o   Add our email to receive alerts

•       Click Create alarm

Test the Alarm

•       Go to shell in ec2 ,

•       Sudo yum install stress -y

•       Stress  --cpu 1 –timeout 60
