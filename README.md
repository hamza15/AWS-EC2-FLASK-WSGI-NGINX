# AWS-EC2-FLASK-WSGI-NGINX
Deploying a Flask application with WSGI and NGINX on an AWS EC2.

Flask as described by the original documentation is “a microframework for Python based on Werkzeug, Jinja 2 and good intentions”. There are quite a few examples on the web explaining basics of deploying Flask on your local PC, however this guide is merely a self-note/setup-guide to get you up and running in a more robust and production ready environment.

Flask’s in-built server by default is not scalable and at cannot handle multiple requests at a time. This is all great when it’s just one person accessing your web server, however for scalability you need your web server to handle multiple requests without taking ages to load.

This brings us to the solution. The idea is to break your stack into parts which can make your web application scalable and fast. This can be achieved by setting up **NGINX** as a dedicated web server which can handle multiple requests and pass them to an application server. This application server, (in our case **WSGI**) would receive those requests and pass the information on as Python objects. WSGI calls your application like a function and passes it the request object. This works great for Flask as the output from this ‘function’ is packaged into an HTTP response by WSGI and passed back to NGINX and out to the user.
 
This document covers deploying a basic flask application on an AWS EC2 instance running in a public subnet. It goes over setup for a Python Virtual Environment, setting up a sample flask application, creating an WSGI entry point, setting up your flask app to run as a service on server boot and configure NGINX to proxy requests. 

### AWS steps:

   1.	Create an AWS VPC.
   2.	Launch a subnet and create an Internet Gateway (IGW) for your subnet.
   3.	After your public subnet is ready, add a route table entry from 0.0.0.0/0 to IGW.
   4.	Create an Elastic-IP and hold on to it.
   5.	Launch an Amazon Linux EC2 Instance in your public subnet (t2-micro – You can always upgrade your AMI once you are done       with this guide).
   6.	Create a Security Group with port 80 for HTTP traffic open to 0.0.0.0/0 and port 22 for SSH open to your IP.
   7.	Upon launch, associate your Elastic IP to your EC2 Instance.
   8.	SSH to your EC2 instance and we should be ready for your application setup.

