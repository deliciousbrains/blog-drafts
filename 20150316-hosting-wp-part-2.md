# Hosting WordPress Yourself Part 1 – Installing Nginx, PHP and MySQL

In part 1 of this series on Hosting WordPress Yourself, I took you through the initial steps to setting up and securing a virtual server. In this post I will guide you through the process of setting up Nginx, PHP-FPM and MySQL, which will form the foundations of a working web server.

Before moving on, you will need to open a new SSH connection to the server, if you haven’t already:

![SSH](https://s3.amazonaws.com/cdn.deliciousbrains.com/content/uploads/2015/03/Image-1-SSH.jpeg)

## Installing Nginx

Although the Ubuntu package manager (Apt-Get) has access to an Nginx repository, it doesn’t have the _fastcgi_cache_purge_ module compiled-in, which will be required later in this series. Luckily, [rtCamp](https://rtcamp.com) have created a pre-compiled version which contains the _fastcgi_cache_purge_ module. To add the repository and install Nginx, enter the following commands:

```
sudo add-apt-repository ppa:rtcamp/nginx -y
sudo apt-get update
sudo apt-get install nginx-custom -y
```

Once complete, you can confirm that Nginx has been installed with the following command:

```
nginx -v
```

Additionally, when visiting your server’s FQDN (fully qualified domain name) in the browser, you should see an Nginx welcome page.

![Welcome to nginx](https://s3.amazonaws.com/cdn.deliciousbrains.com/content/uploads/2015/03/Image-1-Nginx.jpeg)

Now that Nginx has successfully installed it’s time to perform some basic configuration. Out of the box Nginx is pretty well optimized, however there are a few basic adjustments to make. But, before opening the configuration file, you need to determine  your server’s number of CPU cores and the open file limit.

Enter the following commands and make note of the return values:

```
grep processor /proc/cpuinfo | wc -l
ulimit -n
```

Next, open the Nginx configuration file, which can be found at _/etc/nginx/nginx.conf_: 

```
sudo nano /etc/nginx/nginx.conf
```

![nginx.conf](https://s3.amazonaws.com/cdn.deliciousbrains.com/content/uploads/2015/03/Image-2-Nginx.jpeg)

I’m not going to list every configuration directive but I am going to briefly mention those that you should change.

Start by setting the **user** to the _username_ that you’re currently logged in with. This will make managing file permissions much easier in the future, but this is only advantageous when running a single user access server.

The **worker_processes** directive determines how many workers to spawn per server. The general rule of thumb is to set this to the amount of CPU cores your server has available. In my case, this is _1_.

The events block contains 2 directives, the first **worker_connections** should be set to your server’s open file limit. This tells Nginx how many simultaneous connections can be opened by each _worker_process_. Therefore, if you have 2 CPU cores and an open file limit of 1024, your server can handle 2048 connections per second. Connections however doesn’t directly equate to the number of users that can be handled by the server, as the majority of web browsers open at least 2 connections per request. The **multi_accept** directive should be uncommented and set to _on_, which informs each _worker_process_ to accept all new connections at a time, opposed to accepting one new connection at a time.

Moving down the file you will see a _http_ block. The first directive to change is the **keepalive_timeout**, which by default is set to 65. **keepalive_timeout** determines how many seconds a connection to the client should be kept open before it’s closed by Nginx. This directive should be lowered as you don’t want idle connections sitting there for up to 65 seconds if they can be utilised by new clients. I have set mine to _15_.

For security reasons you should uncomment the **server_tokens** directive and ensure it is set to _off_. This will disable emitting the Nginx version number in error messages and response headers.

Underneath **server_tokens** add the **client_max_body_size** directive and set this to the maximum upload size you require in the WordPress Media Library. I chose a value of _64m_.

Further down the _http_ block you will see a section dedicated to Gzip compression. By default, Gzip is enabled but is disabled for Microsoft Internet Explorer 6, however you should tweak these values further for better handling of static files. First, you should uncomment the **gzip_proxied** directive and set it to _any_, which will ensure all proxied request responses are gzipped. Secondly, you should uncomment the **gzip_comp_level** and set it to a value of _2_. This controls the compression level of a response and can have a value in the range of 1 - 9, however setting this too high can have a negative effect on CPU usage. Finally, you should uncomment the **gzip_types** directive, leaving the default values in place. This will ensure that JavaScript, CSS and other file types are gzipped in addition to the HTML file type.

That’s the basic Nginx configuration dealt with, hit _CTRL X_ followed by _Y_ to save the changes. For the changes to take effect you must restart Nginx, however before doing so you should ensure that the configuration file contains no errors by issuing the following command:

```
sudo nginx -t
```

If everything looks ok, go ahead and restart Nginx:

```
sudo service nginx restart
```

![Restart nginx](https://s3.amazonaws.com/cdn.deliciousbrains.com/content/uploads/2015/03/Image-3-Nginx.jpeg)

If everything worked out ok, you should still be able to see the Nginx welcome page when visiting the FQDN in the browser.

![Welcome to nginx](https://s3.amazonaws.com/cdn.deliciousbrains.com/content/uploads/2015/03/Image-1-Nginx.jpeg)


## PHP-FPM

Just like Nginx, the Apt-Get repository does contain a PHP repository, however it isn’t the most up-to-date so we will use one maintained by [Ondřej Surý](https://launchpad.net/~ondrej/+archive/ubuntu/php5-5.6). Add the repo and update the package lists using the following commands:

```
sudo apt-add-repository ppa:ondrej/php5-5.6 -y
sudo apt-get update
```

Then install PHP:

```
sudo apt-get install php5-fpm php5-common php5-mysqlnd php5-xmlrpc php5-curl php5-gd php5-cli php-pear php5-dev php5-imap php5-mcrypt
```

After the installation has completed, confirm that PHP has installed correctly:

```
php5-fpm -v
```

![PHP version number](https://s3.amazonaws.com/cdn.deliciousbrains.com/content/uploads/2015/03/Image-1-PHP.jpeg)

Now that PHP has installed you need to configure the user and group that the service will run under. Because the server isn’t designed for a shared hosting environment, it’s ok to run a single PHP pool under your user account. Open the default pool configuration file:

```
sudo nano /etc/php5/fpm/pool.d/www.conf
```

Change the following lines, replacing _www-data_ with your _username_:

```
user = www-data
group = www-data
listen.owner = www-data
listen.group = www-data
```

Next, you should adjust the php.ini to increase the WordPress maximum upload size. Both this and the **client_max_body_size** directive within Nginx must be changed in order for the new maximum upload limit to take effect. Open the php.ini file:

```
sudo nano /etc/php5/fpm/php.ini
```

Change the following lines, to match the value you assigned to the **client_max_body_size** directive when configuring Nginx:

```
upload_max_filesize = 2M
post_max_size = 8M
```

Hit _CTRL X_ and _Y_ to save the configuration. Before restarting PHP, check that the configuration file syntax is correct:

```
sudo php5-fpm -t
```

![PHP configuration test](https://s3.amazonaws.com/cdn.deliciousbrains.com/content/uploads/2015/03/Image-2-PHP.jpeg)

If the configuration test was successful, restart PHP using the following command:

```
sudo service php5-fpm restart
```

![Restart PHP](https://s3.amazonaws.com/cdn.deliciousbrains.com/content/uploads/2015/03/Image-3-PHP.jpeg)

Now that Nginx and PHP have been installed, you can confirm that they are both running under the correct user by issuing the _top_ command:

```
top
```

If you hit _SHIFT M_ the output will be arranged my memory usage which should bring the nginx and php5-fpm processes into view. You should notice a few occurrences of both nginx and php-fpm. Both processes will have one instance running under the root user (this is main process that spawns each worker) and the remainder should be running under the username you specified.

![Running processes](https://s3.amazonaws.com/cdn.deliciousbrains.com/content/uploads/2015/03/Image-4-PHP.jpeg)

If not, go back and check the configuration, and ensure that you have restarted both the Nginx and PHP-FPM services.

## MySQL

The final package to install is MySQL, which requires very little configuration compared to Nginx and PHP. That’s not to say that MySQL doesn’t have many configurable options, however MySQL tuning is far beyond the scope of this post and the default settings should serve you well to begin with.

To install MySQL, issue the following command:

```
sudo apt-get install mysql-server
```

You’ll be prompted to enter a root password, which should be complex, as described in the previous post. 

![MySQL installation](https://s3.amazonaws.com/cdn.deliciousbrains.com/content/uploads/2015/03/Image-1-MySQL.jpeg)

Once MySQL has installed, you need to setup the default system tables:

```
sudo mysql_install_db
```

The last step is to secure MySQL. Luckily, there’s a built in script which will prompt you to change a few insecure defaults:

```
sudo mysql_secure_installation
```

You will be prompted to enter the root password you created in the previous step and will be given the option to change this password if you are not happy with it.

![MySQL change root password](https://s3.amazonaws.com/cdn.deliciousbrains.com/content/uploads/2015/03/Image-2-MySQL.jpeg)

A number of additional prompts will appear, you should accept the default values by hitting _ENTER_.

![MySQL security prompts](https://s3.amazonaws.com/cdn.deliciousbrains.com/content/uploads/2015/03/Image-3-MySQL.jpeg)

## Catch All Server Block

The last thing to address in this post is to remove the default server block from Nginx. Currently, when you visit the server’s FQDN in a web browser you should see the Nginx welcome page. However, unless visiting a known host the server should return a 444 response.

Begin by removing the following two files:

```
sudo rm /etc/nginx/sites-available/default
sudo rm /etc/nginx/sites-enabled/default
```

Now you need to add a catch all block to the Nginx configuration. Using _nano_, open the _nginx.conf_ file:

```
sudo nano /etc/nginx/nginx.conf
```

Towards the bottom of the file you’ll find a line that reads:

```
include /etc/nginx/sites-enabled/*;
```

Underneath add the following:

```
server {
    listen 80 default_server;
    server_name _;
    return 444;
}
```

Hit _CTRL X_ followed by _Y_ to save the changes and then test the Nginx configuration:

```
sudo nginx -t
```

If everything looks good, restart Nginx:

```
sudo service nginx restart
```

Now when you visit the FQDN you should receive an error.

![Browser error](https://s3.amazonaws.com/cdn.deliciousbrains.com/content/uploads/2015/03/Image-1-Catch-All.jpeg)

Here’s my final _nginx.conf_ file, after applying all of the above changes. I have removed the mail block, as this isn’t something I use:

<script src=“https://gist.github.com/A5hleyRich/d88e6de510dd8e303153.js?file=nginx.conf”></script>

That’s all for this post, if you have any questions please feel free to ask them below. In the next post I will guide you through the process of setting up your first WordPress site and how to manage multiple WordPress installs.
