# Terraform_Aws_Autoscaling_Group_And_Aws_Load_Balancer

We are a preferred AWS consultant and offers the best cloud AWS consulting service. Our AWS-certified expert consultants conduct a thorough review and evaluation of your existing IT infrastructure and service interaction model to provide top-notch solutions.
<h3><a href="https://www.ecomstreet.com/aws-consulting-services-company/" target="_blank">Schedule your free consultation today!</a></h3> <br/>

As soon as you learn how to manage basic network infrastructure in AWS using Terraform (see “Terraform recipe – Managing AWS VPC – Creating Public Subnet” and “Terraform recipe – Managing AWS VPC – Creating Private Subnets”), you want to start creating auto-scalable infrastructures.

# Auto Scaling Groups

Usually, Auto Scaling Groups are used to control the number of instances executing the same task like rendering dynamic web pages for your website, decoding videos and images, or calculating machine learning models.

Auto Scaling Groups also allows you to dynamically control your server pool size – increase it when your web servers are processing more traffic or tasks than usual, or decrease it when it becomes quieter.

In any case, this feature allows you to save your budget and make your infrastructure more fault-tolerant significantly.

Let’s build a simple infrastructure, which consists of several web servers for serving website traffic. In the following article, we’ll add RDS DB to our infrastructure.

Our infrastructure will be the following:

<img width="521" alt="c1" src="https://user-images.githubusercontent.com/115148205/197935178-eae2d1f9-9cba-405f-947d-c925604c0199.PNG">

# Setting up VPC.
Let’s assemble it in a new main.tf file. First of all, let’s declare VPC, two Public Subnets, Internet Gateway and Route Table (we may take this example as a base):

<img width="862" alt="1" src="https://user-images.githubusercontent.com/115148205/197942198-00ecb773-1830-4f85-bbcd-6a532720527f.PNG">
<img width="826" alt="2" src="https://user-images.githubusercontent.com/115148205/197942308-c7728a52-8880-4d47-8c22-361e953e5595.PNG">

Next, we need to describe the Security Group for our web-servers, which will allow HTTP connections to our instances:

<img width="816" alt="9" src="https://user-images.githubusercontent.com/115148205/197945281-f507351a-5e17-4fd5-bd81-dfd8c3d28e78.PNG">


# Launch configuration

As soon as we have SecurityGroup, we may describe a Launch Configuration. Think of it like a template containing all instance settings to apply to each new launched by Auto Scaling Group instance. We’re using aws_launch_configuration resource in Terraform to describe it

Most of the parameters should be familiar to you, as we already used them in aws_instance resource.

<img width="807" alt="10" src="https://user-images.githubusercontent.com/115148205/197946813-c7559169-c561-4378-b3ac-4aa5ac90177e.PNG">

# The new ones are a user_data and a lifecycle:

# user_data :- 
is a special interface created by AWS for EC2 instances automation. Usually this option is filled with scripted instructions to the instance, which need to be executed at the instance boot time. For most of the OS this is done by cloud-init.

# lifecycle :
special instruction, which is declaring how new launch configuration rules applied during update. We’re using create_before_destroy here to create new instances from a new launch configuration before destroying the old ones. This option commonly used during rolling deployments.

The user-data option is filled with a simple bash-script, which installs the Nginx web server and puts the instance’s local IP address to the index.html file, so we can see it after the instance is up and running

# Load Balancer
Before we create an Auto Scaling Group, we need to declare a Load Balancer. There are three Load Balances available for you in AWS right now:

# Elastic or Classic Load Balancer (ELB):
previous generation of Load Balancers in AWS.

# Application Load Balancer (ALB):
operates on application network layer and provides reach feature set to manage HTTP and HTTPS traffic for your web applications.

# Network Load Balancer (NLB):
operates on connection layer and capable for handling millions of requests per second.

For simplicity, let’s create an Elastic Load Balancer in front of our EC2 instances (I’ll show how to use other types of them in future articles). To do that, we need to declare aws_elb resource.

<img width="821" alt="11" src="https://user-images.githubusercontent.com/115148205/197948490-68227126-7726-4787-9769-efa3f7dffb5a.PNG">
<img width="817" alt="12" src="https://user-images.githubusercontent.com/115148205/197948782-9e74621d-b580-4b66-bc8c-94fae32fa791.PNG">

Here we’re setting up Load Balancer name, it’s own Security Group, so we could make traffic rules more restrictive later if we want to.

We’re specifying 2 subnets, where our Load Balancer will look for (listener configuration) launched instances and turned on cross_zone_load_balancing feature, so we could have our instances in different Availability Zones.

And finally, we’ve specified health_check configuration, which determines when Load Balancer should transition instances from healthy to unhealthy state and back depending on its ability to reach HTTP port 80 on the target instance.

If ELB can not reach the instance on the specified port, it will stop sending traffic.

# Auto scaling group

Now we’re ready to create an Auto Scaling Group by describing it using aws_autoscaling_group resource:





  










