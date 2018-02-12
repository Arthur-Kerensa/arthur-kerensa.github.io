## Full Install.
This guide is using digitalocean cloud hosting but the same steps should work equally well with another cloud provider AWS, GoogleCloud, etc.

### Inital Digital Ocean Setup
- Navigate to https://www.digitalocean.com/
- Create an account, verify email, connect twitter, google and github accounts, fill out profile and add payment method.
- If you don't have github, google, and twitter accounts, make them now.
- Create an SSH key and add it to the Digital Ocean console.
- Start by creating an instance running Ubuntu 16.04 x64.
- Create another identical instance. (mirror all actions to keep it indentical)
- Add another instance for database backend, ignore this for now as you will configure it later.
- Consider assigning a `static` or `floating` ip address to your instances via the console.
- SSH into both instances from your local machine.
- Follow this [intial setup guide](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04) on both machines.
- Login to your domain registar (namecheap, godaddy etc) and point the nameservers to digitalocean. `ns1.digitalocean.com` `ns2.digitalocean.com` `ns3.digitalocean.com`
- Taking note of [this guide](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-host-name-with-digitalocean) now follow this [guide](https://www.digitalocean.com/community/tutorials/how-to-configure-ssl-termination-on-digitalocean-load-balancers) to create a load balancer.
- During this procedure you will have installed NGinx and created a HTTPS certificate. 
- Any NGinx issues can be checked with the help of [this guide](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-16-04).
- Create a firewall on the console for your two frontend servers, allow all HTTP(S) and SSH traffic.

### Next setup the database
- SSH into your database server
- Run the [Initial Server Setup](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04)
- Install [Mongo DB](https://docs.mongodb.com/manual/administration/install-community/) from [here](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/).
- Start MondoDB and pin the version.
- Note the use of port 27017, add a ufw rule on the database server to allow it from both frontend servers. eg. `sudo ufw allow 27017` or the (safer) `sudo ufw allow from your_other_server_ip/32 to any port 27017` (remember to allow ingress from both frontend servers)
- If you need help, check out [this guide](https://www.digitalocean.com/community/tutorials/how-to-install-mongodb-on-ubuntu-16-04) which also gives firewall instuctions.
- Follow this [guide](https://ianlondon.github.io/blog/mongodb-auth/) with the notes below:
```
use wiki_db

db.createUser({
    user: 'jeremy',
    pwd: 'secretPassword',
    roles: [{ role: 'readWrite', db:'wiki_db'}]
})
```
- Use `nano /etc/mongod.conf`
- indent is important.
- restart mongod `sudo service mongod start` or `restart` then `status` to check it.
- Note wiki.js will need this: `mongodb://jeremy:secretPassword@159.65.30.111:27017/wiki_db` string to connect.
- Read [this on URI](https://docs.mongodb.com/manual/reference/connection-string/), [this on secuirty](https://www.digitalocean.com/community/tutorials/how-to-securely-configure-a-production-mongodb-server) and [this install guide](https://www.digitalocean.com/community/tutorials/how-to-install-mongodb-on-ubuntu-16-04) for more useful info.
- after trying a million things such a below, i gave up:
```
https://docs.mongodb.com/manual/tutorial/enable-authentication/
mongod --auth --port 27017 --dbpath /
mongo -u jeremy -p secretPassword 159.65.30.111/wiki_db

https://docs.mongodb.com/manual/tutorial/configure-linux-iptables-firewall/
iptables -A INPUT -s <ip-address> -p tcp --destination-port 27017 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -d <ip-address> -p tcp --source-port 27017 -m state --state ESTABLISHED -j ACCEPT

use admin
db.createUser(
  {
    user: "admin",
    pwd: "secretPassword",
    roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
  }
)

mongo --port 27017 -u "admin" -p "secretPassword" \
  --authenticationDatabase "admin"

mongo -u admin -p secretPassword --authenticationDatabase admin

mongo -u admin -p secretPassword --authenticationDatabase admin

mongo -u admin -p "secretPassword" 159.65.30.111/wiki_db
````
- i then installed it on the frontend instead.

### contine to install the software.
- Enable HTTP(S) on both frontend servers on the ufw firewall by running `sudo ufw allow https` and `sudo ufw allow http`
- Make sure the console set firewall allows egress on at least port 27017 for mongodb.
- Set the firewall to allow ingress on port 27017 for TCP/UDP also.

#### Cost
- At this point you may note the excessive cost of running a load balancer and three instances, so if you don't expect lots of visitors to your site you may want to scale down.
- You can reduce costs by destroying the load balancer and then saveing the second of the frontend instances as a `snapshot` or `image` for later cloning. Before then destroying it also.
- To do this, stop the instance with `sudo shutdown -h now` then use the console to make a snapshot or image, verify this is done, then destroy the instance (make sure it is the correct instance).
- You will already have a SSL/HTTPS Cert from earlier, however you may want to generate another to use certbot's automatic nginx configuration abilty, you can use [this guide](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04) to set up your remaining frontend nginx installation with HTTPS.
- Remember to repoint the DNS records to your remaining frontend IP or your site will be unreachable.

### Further installation (node.js)
- Node JS is able to be a webserver in it's own right, but in this case NGinx will be used in front.
- Use this [guide](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-16-04) to install node.js. follow the hello world example. (NB. I deleted the `location ~ /.well-known { ...` section by mistake). make sure you site displays hello world before continuing.
- Remove the hello world init with `pm2 unstartup systemd` command.

### Setup Git repo
- Make sure git is installed (use `git --version`) on your frontend server.
- Create a git repo.
- Follow this [guide](https://docs.requarks.io/wiki/install/git) to setup your repo, you will need to setup a new [SSH key](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/) on your frontend server.

### wiki node
- Use this [wiki js](https://docs.requarks.io/wiki/install) installation guide.
- I used `npm install wiki.js@latest` to install in a `Wiki` directory in root home directory.
- Custom port is 3000.
- start wiki.js with `node wiki configure`
- direct root nginx to port 3000 and hello.js to `/hello` (for debugging). Do this by:
- Edit  `nano /etc/nginx/sites-available/default` to look like:
```
        location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;

                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
        }

        location /hello {
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;

        }
```

- Check syntax with `nginx -t` and restart Nginx `sudo systemctl restart nginx`, then test root and `/hello` on the domain name. eg. `wiki.org.uk` and `wiki.org.uk/hello` they should both work.
- Install lynx via `sudo apt-get install lynx`.
- Using your second SSH connection, use lynx to complete the wiki.js config on `http://localhost:3000` (this won't be used unless you really need it).
- add 
```
        location ~ ^/(static|media|js|images|css|fonts)/ {
        proxy_pass http://localhost:3000;
        }
```
 - to the nginx config or it won't work.
  - At this point you should just copy [this nginx config](https://github.com/Arthur-Kerensa/arthur-kerensa.github.io/blob/master/nginx-settings.md) because all the previous nginx is screwed to hell...
- This [info](https://docs.requarks.io/wiki/admin-guide/setup-nginx) is a good guide too, but note a few rules will need to be wiped out due to duplication caused presumebly by certbot.
- [This](https://www.nginx.com/resources/wiki/start/) and [this](https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/) and [this](https://www.nginx.com/resources/wiki/start/) are worth reading.
- 

### Further installation 
- Wiki JS

Is Wiki.js going to be behind a web server (e.g. nginx / apache / IIS) or proxy?
- Make sure the upload limit is sufficient. Most web servers have a low limit (e.g. 2 MB) by default.
- Make sure your web server is configured to allow web sockets. Wiki.js will fallback to standard XHR queries if not available.
- Do not rewrite URLs after the domain. This can cause unexpected issues in Wiki.js navigation.
- Do not remove or alter the client IP when proxying the requests. This can cause the authentication brute force protection to engage unexpectedly.

### Authentication
- 0Auth

### Final steps
- Once complete: use `sudo shutdown`, create a snapshot, reboot and clone as needed.
- Creating a init script would be good too.

### Services and further considerations
- https://www.nginx.com/blog/5-performance-tips-for-node-js-applications/
- https://docs.requarks.io/wiki/prerequisites
- https://gitter.im/Requarks/wiki
- https://www.keycdn.com/blog/make-a-favicon/
- https://cloudinary.com/
- https://www.keycdn.com/blog/image-cdn/
- https://www.keycdn.com/blog/responsive-images/
- https://developers.google.com/speed/
- https://www.digitalocean.com/community/tutorials/how-to-optimize-nginx-configuration
- https://serverfault.com/questions/564127/nginx-location-regex-for-multiple-paths-with-backend

### To host a second site
To also host mediawiki:
- create a [LEMP stack](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-in-ubuntu-16-04).
- Use [this](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-virtual-hosts-on-ubuntu-16-04) to setup more than one domain.

### add robots.txt and sitemap
- Edit `nano /etc/nginx/sites-available/default`
- Add:
```
    location = /robots.txt {
        alias /var/www/html/robots.txt;

        }

    location = /sitemap.xml {
        alias /var/www/html/sitemap.xml;

        }
```
- Restart NGinx with `sudo systemctl restart nginx`
- Find out about sitemaps [here](https://www.sitemaps.org/protocol.html#sitemapXMLExample) and robots.txt files at the [google search console](https://support.google.com/webmasters/answer/6062596?hl=en&ref_topic=6061961).

```
Feb 12 14:30:43 ubuntu-s-1vcpu-1gb-lon1-01-wiki-js-01 nginx[13776]: nginx: [warn] duplicate value "TLSv1" in /etc/letsencrypt/options-ssl-nginx.conf:10
Feb 12 14:30:43 ubuntu-s-1vcpu-1gb-lon1-01-wiki-js-01 nginx[13776]: nginx: [warn] duplicate value "TLSv1.1" in /etc/letsencrypt/options-ssl-nginx.conf:10
Feb 12 14:30:43 ubuntu-s-1vcpu-1gb-lon1-01-wiki-js-01 nginx[13776]: nginx: [warn] duplicate value "TLSv1.2" in /etc/letsencrypt/options-ssl-nginx.conf:10
Feb 12 14:30:43 ubuntu-s-1vcpu-1gb-lon1-01-wiki-js-01 nginx[13781]: nginx: [warn] duplicate value "TLSv1" in /etc/letsencrypt/options-ssl-nginx.conf:10
Feb 12 14:30:43 ubuntu-s-1vcpu-1gb-lon1-01-wiki-js-01 nginx[13781]: nginx: [warn] duplicate value "TLSv1.1" in /etc/letsencrypt/options-ssl-nginx.conf:10
Feb 12 14:30:43 ubuntu-s-1vcpu-1gb-lon1-01-wiki-js-01 nginx[13781]: nginx: [warn] duplicate value "TLSv1.2" in /etc/letsencrypt/options-ssl-nginx.conf:10
Feb 12 14:30:43 ubuntu-s-1vcpu-1gb-lon1-01-wiki-js-01 systemd[1]: nginx.service: Failed to read PID from file /run/nginx.pid: Invalid argument
```
- robots.txt
```
# Rule 1
User-agent: Googlebot
Disallow: /private/

# Rule 2
User-agent: *
Allow: /

Sitemap: https://wiki.org.uk/sitemap.xml
```
- sitemap.xml
```
<?xml version="1.0" encoding="UTF-8"?>

<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">

   <url>

      <loc>https://wiki.org.uk/</loc>

      <lastmod>2018-02-02</lastmod>

      <changefreq>daily</changefreq>

      <priority>1</priority>

   </url>

</urlset>
```
### setup CDN
- https://app.keycdn.com/users/view

