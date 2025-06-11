
## Configure Web Instance ## 



We now need to install all of the necessary components needed to run our front-end application. Again, start by installing NVM and node :

```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
source ~/.bashrc
nvm install 16
nvm use 16
```

1. Now we need to download our web tier code from our s3 bucket:

````
cd ~/
aws s3 cp s3://BUCKET_NAME/web-tier/ web-tier --recursive
````

2. Navigate to the web-layer folder and create the build folder for the react app so we can serve our code:

```
cd ~/web-tier
npm install 
npm run build
```

* **NGINX** can be used for different use cases like load balancing, content caching etc, but we will be using it as a web server that we will configure to serve our application on port 80, as well as help direct our API calls to the internal load balancer.

```
sudo amazon-linux-extras install nginx1 -y
```

3. We will now have to configure NGINX. Navigate to the Nginx configuration file with the following commands and list the files in the directory:

```
cd /etc/nginx
ls
```

4. You should see an nginx.conf file. We’re going to delete this file and use the one we uploaded to s3. Replace the bucket name in the command below with the one you created for this workshop:

```
sudo rm nginx.conf
sudo aws s3 cp s3://BUCKET_NAME/nginx.conf .
```

Then, restart Nginx with the following command:

```
sudo service nginx restart
```

5. To make sure Nginx has permission to access our files execute this command:

```
chmod -R 755 /home/ec2-user
```
And then to make sure the service starts on boot, run this command:

```
sudo chkconfig nginx on
```
6. Now when you plug in the public IP of your web tier instance, you should see your website, which you can find on the Instance details page on the EC2 dashboard. If you have the database connected and working correctly, then you will also see the database working. You’ll be able to add data. Careful with the delete button, that will clear all the entries in your database.

![image](https://static.us-east-1.prod.workshops.aws/public/32bb8fc8-8b71-4fd4-89a6-f69a91cc4458/static/part5/WebPage2.png) , 
