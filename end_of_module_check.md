# Add an EC2 instance with ELB and auto-scaling

## Introduction

In this exercice we want to add an EC2 instance to be able to manage a bigger amount of trafic and improve our performance. The goal of the exercise is to have a functional URL on the internet that displays Hello followed by your name and the hostname of the instance.

To do that we are going to add a [ELB](https://aws.amazon.com/elasticloadbalancing/) that is going to be the one in charge of distribute the traffic accross our instances.

Also we will add an [auto-scaling group](https://aws.amazon.com/documentation/autoscaling/) with 2 availability zones.
This way we ensure that if we have 2 instances one on each availability zone, and an [Availability Zone](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-regions-availability-zones) goes down and our instance terminated, AWS will automatically start a new instance in the other availability zone so we don't decrease our performance.
Also we will create some rules to add more instances if our 2 instances are overloaded (example: using 20% of cpu for the last 5 minutes), you can add whatever rule you want.

> There are deliberately missing steps in this exercise, rely on what has been seen in previous courses as well as AWS documentation, to make your application functional.

## Create a Load Balancer

Elastic Load Balancing automatically distributes incoming application traffic across multiple targets, such as Amazon EC2 instances, containers, and IP addresses. When you are running applications in production, you typically will use multiple instances so if one fails, your application can still work. The Load Balancer will get the traffic, and will forward it to the instances that serve your app. You can more about this [here](https://aws.amazon.com/elasticloadbalancing/).

1. Go to **EC2** under **Compute** section.
2. On left menu select **Load Balancers** under **LOAD BALANCING**.
3. Click **Create Load Balancer**.

## Create Auto Scaling Group

Production applications need to be ready to tolerate a growing number of users at the same time. For example, if you get published in a popular blog, you may receive many more users that you had expected in a short period of time, and your application may crash because it's not able to sustain all the incoming traffic.

Amazon provides [Auto Scaling Groups](https://docs.aws.amazon.com/autoscaling/latest/userguide/AutoScalingGroup.html) as way to build a more robust application which can handle increasing loads. Using these, you can setup rules (scaling policies) so more instances serving your application

To create an Auto Scaling Group, first we need to create a [Launch Configuration](http://docs.aws.amazon.com/autoscaling/latest/userguide/LaunchConfiguration.html), which is basically a template that specifies properties of the instances that will be launched.

### Create Launch Configuration

1. Go to **EC2** under **Compute** section.
2. On left menu select **Launch Configuration** under **AUTO SCALING**.
3. Click **Create launch configuration**.
4. Look for Ubuntu Server (make sure it say Free tier eligible) and click Select.
5. Select `t2.micro` and then click on **Next: Configure Details**.
6. On **Advanced Settings**, there you have to select **As text** in **User data** and paste this bash script:

    ```sh
    #!/bin/bash

    apt-get update
    apt-get install -y apache2
    cat <<EOF > /var/www/html/index.html
    <html>
    <body>
    <h1>Hello Your Name</h1>
    <p>hostname is: $(hostname)</p>
    </body>
    </html>
    EOF
    systemctl restart apache2
    systemctl enable apache2
    ```

7. Click **Create launch configuration** and select the key pair to used to `ssh` into future instances.

Now that we have our **Launch configuration** we can create our **Auto Scaling Group**.

### Create an Auto Scaling Group

1. Go to **EC2** under **Compute** section.
2. On left menu select **Auto Scaling Groups** under **AUTO SCALING**.
3. Click: **Create Auto Scaling group**.
4. Select: `your auto-scaling-group` and then click **Next Step**.
5. On **Group name** put the same as in Launch configuration.
6. **Group size:** 2. At least we will have some redundancy form the start!
7. Click **Next: Configure scaling policies**.
8. Select: **Use scaling policies to adjust the capacity of this group**.
9. Configure it to scale between 2 and 4 instances.
10. Pick `Average CPU Utilization` as metric (imagine your app was compute intensive). In Target value, set something like 20%.
11. Click **Review**.
12. Click **Create Auto Scaling group**.

## Test and send the results

Retrieve the URL of the Load Balancer, test the operation, you should see a blank page displaying Hello followed by your name. If it works send the URL to my EPSI email address.

If it doesn't work, it's time to debug, remember to check your target groups, your routes, your security groups, etc.
