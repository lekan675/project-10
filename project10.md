## LOAD BALANCER SOLUTION WITH NGINX AND SSL/TLS


### Task

This project consists of two parts:

1.  Configure Nginx as a Load Balancer
 
  
2.  Register a new domain name and configure secured connection using SSL/TLS certificates

Your target architecture will look like this:

![](./images/NGNIX.jpg)


## CONFIGURE NGINX AS A LOAD BALANCER

You can either uninstall Apache from the existing Load Balancer server, or create a fresh installation of Linux for Nginx.

1.  Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it `Nginx LB` (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 – this port is used for secured HTTPS connections)

  ![](./images/From%20the%20Actions%20dropdown%20menu%2C%20select%20Networking%20and%20then%20Manage%20IP%20addresses.jpg)
  

2.  Update `/etc/hosts` file for local DNS with Web Servers’ names (e.g. `Web1` and `Web2`) and their local IP addresses

    ![](./images/vi%20etc%20hosts.jpg)


3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers

Update the instance and Install Nginx

`sudo apt update`

`sudo apt install nginx`

![](./images/update%20and%20install%20ngnix.jpg)

Configure Nginx LB using Web Servers’ names defined in /etc/hosts

Hint: Read this blog to read about `/etc/host`

Open the default nginx configuration file

`sudo vi /etc/nginx/nginx.conf`


insert following configuration into http section

 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }

![](./images/edit%20default%20nginx%20configuration%20file.jpg)

#comment out this line
#       include /etc/nginx/sites-enabled/*;

Restart Nginx and make sure the service is up and running

`sudo systemctl restart nginx`

`sudo systemctl status nginx`

![](./images/restart%20ngnix.jpg)

REGISTER A NEW DOMAIN NAME AND CONFIGURE SECURED CONNECTION USING SSL/TLS CERTIFICATES

In order to get a valid SSL certificate, I registered a new domain name. 

1. I purchased the domain name - `onimoleoflagos.com` from Azure.

![](./images/domain.jpg)

2. Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP

  ![](./images/Associate%20ip.jpg)


3. Update A record in your registrar to point to Nginx LB using Elastic IP address

  ![](./images/hosted%20dns%20record.jpg)

  Restart the web service on your webservers

    `sudo systemctl restart httpd`

    `sudo systemctl status httpd`

    ![](./images/restart%20webserver1.jpg)

    ![](./images/restart%20webserver2.jpg)

    Check that the Web Servers can be reached from the browser using new domain name using HTTP protocol – http://www.onimoleoflagos.com

  ![](./images/test%20the%20url%20on%20the%20website.jpg)

4. Configure Nginx to recognize your new domain name

![](./images/edit%20default%20nginx%20configuration%20file.jpg)

5. Install certbot and request for an SSL/TLS certificate

  Make sure snapd service is active and running

  `sudo systemctl status snapd`

  ![](./images/Make%20sure%20snapd%20service%20is%20active%20and%20running.jpg)

  Install certbot

  `sudo snap install --classic certbot`

  ![](./images/Install%20Certbot.jpg)

  Request your certificate (just follow the certbot instructions – you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from nginx.conf file)

  `sudo ln -s /snap/bin/certbot /usr/bin/certbot`

  `sudo certbot --nginx`

  ![](./images/You%20can%20test%20renewal%20command%20in%20dry-run%20mode.jpg)

  
  Test secured access to your Web Solution by trying to reach `https://onimoleoflagos.com`

![](./images/Test%20secured%20access%20to%20your%20Web%20Solution%20by%20trying%20to%20reach.jpg)

6. Set up periodical renewal of your **SSL/TLS** certificate

  By default, **LetsEncrypt** certificate is valid for **90 days**, so it is recommended to renew it at least every **60 days** or more frequently.

  You can test renewal command in dry-run mode

  `sudo certbot renew --dry-run`

  ![](./images/sudo%20certbot%20renew%20--dry-run.jpg)

  Best practice is to have a scheduled job that to run renew command periodically. Let us configure a `cronjob` to run the command twice a day.

To do so, lets edit the crontab file with the following command:

`crontab -e`

![](./images/crontab.jpg)

Add following line:

`* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1`

![](./images/edit%20the%20crontab%20file%20with%20the%20following%20command.jpg)


