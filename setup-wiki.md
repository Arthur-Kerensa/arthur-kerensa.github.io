## Full Install.
This guide is using digitalocean cloud hosting but the same steps should work equally well with another cloud provider AWS, GoogleCloud, etc.

### Inital Digital Ocean Setup
- Navigate to https://www.digitalocean.com/
- Create an account, verify email, connect twitter, google and github accounts, fill out profile and add payment method.
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
- Create a firewall on the console for your two frontend servers, allow all HTTP(S) and SSH traffic.

### Next setup the database
- SSH into your database server
- Run the [Initial Server Setup](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04)
- Install [Mongo DB](https://docs.mongodb.com/manual/administration/install-community/) from [here](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/).
- Start MondoDB and pin the version.
- Note the use of port 27017, add a ufw rule on the database server to allow it from both frontend servers. eg. `sudo ufw allow 27017` or the (safer) `sudo ufw allow from your_other_server_ip/32 to any port 27017` (remember to allow ingress from both frontend servers)
- If you need help, check out [this guide](https://www.digitalocean.com/community/tutorials/how-to-install-mongodb-on-ubuntu-16-04) which also gives firewall instuctions.

### contine to install the software.
- Enable HTTP(S) on both frontend servers on the ufw firewall by running `sudo ufw allow https` and `sudo ufw allow http`
- Make sure the console set firewall allows egress on at least port 27017 for mongodb.
- Set the firewall to allow ingress on port 27017 for TCP/UDP also.

#### Cost
- At this point you may note the excessive cost of running a load balancer and three instances, so if you don't expect lots of visitors to yur site you may want to scale down.
- You can reduce costs by destroying the load balancer and then saveing the second of the frontend instances as a `snapshot` or `image` for later cloning. Befre then destroying it also.
- To do this, stop the instance with `sudo shutdown -h now` then use the console to make a snapshot or image, verify this is done, then destroy the instance (make sure it is the correct instance).
- You will already have a SSL/HTTPS Cert from earlier, however you may want to generate another to use certbot's automatic nginx configuration abilty, you can use [this guide](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04) to set up your remaining frontend nginx installation with HTTPS.
- Remember to repoint the DNS records to your remaining frontend IP or your site will be unreachable.

###Further installation

- Node JS
- Wiki JS

### Setup Git repo
- git

### Authentication
- 0Auth

### Services and further considerations
- https://docs.requarks.io/wiki/prerequisites
- https://gitter.im/Requarks/wiki
- https://www.keycdn.com/blog/make-a-favicon/
- https://cloudinary.com/
- https://www.keycdn.com/blog/image-cdn/
- https://www.keycdn.com/blog/responsive-images/
- https://developers.google.com/speed/
- https://www.digitalocean.com/community/tutorials/how-to-optimize-nginx-configuration




