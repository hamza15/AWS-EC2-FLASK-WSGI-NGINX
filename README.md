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

### Flask Configuration Steps:

   1.	Install pip, python-devel, gcc and nginx packages with the following command:

      	>  sudo yum install python-pip python-devel gcc nginx

   2.	Create a Python Virtual Environment to isolate your Flask application from other Python files. Run the following command:
       >  sudo pip install virtualenv
   
   3. Make a directory and change to it:
       >  mkdir ~/app-flask
       >  cd ~/app-flask
       
   4. Create a virtual enviornment inside your Flask application and active it:
       >  virtualenv app-flask
       >  source virtualenv/bin/activate
       
   5. Install flask and uwsgi:
       >  pip install uwsgi flask
       
   6. Create a sample app with Flask. Create a file in your directory called hello.py and copy the hello.py file provided in this repo.
   
   7. Execute the file with python:
       >  python hello.py
       
   8. Open a web browser and naviagte to your Elastic Ip Address. **You should see "Hello there!"**    
   
   9. Next, create a WSGI entry point so your uWSGI application server can interact with your application. Call this file wsgi.py and copy the wsgi.py file from this repo. 
   
      **NOTE: Before testing uWSGI Server configuration, open port 8000 on your Security Group in AWS. Without that port open, this will not work. You can close it after you test.**
   
   10. Test your uWSGI Server configuration by providing an available socket and specifying a protocol:
       >  uwsgi --socket 0.0.0.0:8000 --protocol=http -w wsgi
       
   11. We can deactivate the virtual environment since the rest of the commands are not specific to this project.
   
   12. Create a uWSGI Configuration file for a prod-ready environment. Copy the wsgi.ini file from this repo and place it in your project directory. 
   
       **We don't specify a protocol in this config file because by NGINX speaks the binary protocol natively. WSGI uses the binary protocol byd default.**
   
   13. It's important to setup your Flask application to be automatically started at server boot. This provides an **Always-on** attribute, which coupled with other infrastructural changes, like Load Balancing and Auto-scaling can create a full production workload for your application. To get started navigate to /etc/systemd/system/ directory and create a file called app-flask.service inside. Copy the app-flask.service file from this repo.
   
       **If you are using a user other than ec2-user, change the username accordingly.**
       
   14. Start the uWSGI service and enable it so your application starts at boot.
       >  sudo systemctl start app-flask
       >  sudo systemctl enable app-flask
       
   15. Configure NGINX to pass requests to the socket uWSGI is configured at. This can be done by updating the nginx.conf file. Add a **server block** right above the server block already configured and copy the nginx.conf file server block to your project nginx.conf file. Make sure to keep the rest of the file as default.
   
   **NOTE: UPDATE THE 'SERVER NAME' by replacing the Elastic-IP address with your AWS Elastic-IP.**
   
   16. For nginx to have access to your application, add nginx to the ec2-user group so it can access the socket file. Also change permissions on teh home directory:
       >  sudo usermod -a -G ec2-user nginx
       >  chmod 710 /home/ec2-user
       
   17. Test your nginx conifguration for syntax errors with the following command:
       >  sudo nginx -t
       
   18. If the syntax passes, start and enable Nginx so it can start at boot:
       >  sudo systemctl start nginx
       >  sudo systemctl enable nginx
       
   19. Although at this point you can navigate to the your Elastic IP on a web browser and should see it load up, however in some cases you may see 502 bad gateway message from NGINX. Navigate to the /var/log/nginx/error.log and check for errors. If you see:
       >  unix:/home/ec2-user/app-flask/flask.sock failed (2: No such file or directory) while connecting to upstream
       
   In the above case run the uWSGI config file to start your WSGI application server, so a socket file is created in your app-flask directory:
       >  uwsgi --ini wsgi.ini
       
   20. Navigate to your Elastic-IP with your web broswer and you should see it is up and running now! Your Flask application is now running with WSGI conifgured as an application server and NGINX as a web server.     
       
    
   
   
