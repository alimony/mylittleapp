My Little App
=============

#### Easy setup of your own Heroku clone with Ansible and Dokku

---

This is an [Ansible](https://github.com/ansible/ansible) playbook that from scratch will automatically turn a pristine server into your own Heroku clone. All you need is the server, a domain name, and a DNS service supporting alias records. Everything is powered by [Dokku](https://github.com/progrium/dokku), which itself runs on [Docker](https://www.docker.io/).


Setup
-----

Just follow these instructions, and you will soon be up and running:

1. **Register a domain name** where your applications will live, for example `mylittleapp.org`. Each application will run on a subdomain of this host, e.g. `helloworld.mylittleapp.org`.

2. **Set up a new server** that you can access by SSH key only. This playbook has been tested on a new Ubuntu 13.04 droplet on [DigitalOcean](https://www.digitalocean.com/), but a variety of other setups should work as well.

3. **Set up two DNS entries** for the new domain name, both pointing at the new server's IP address:
 * One `A` record for `mylittleapp.org`
 * One `A` record for `*.mylittleapp.org`

    Your DNS provider must support alias A records of some kind. The above example works on [Amazon Route 53](http://aws.amazon.com/route53/), but something like [DNSimple](https://dnsimple.com/) should work as well.

4. Make sure your new domain name points at the name servers of your DNS service.

5. [Install Ansible](http://docs.ansible.com/intro_installation.html) and its dependencies on your local machine.

6. Copy the file `hosts.template` to `hosts` and change "mylittleapp.org" to your own domain name.

7. Copy your public SSH key to the `authorized_keys` directory (e.g. `cp ~/.ssh/id_dsa.pub authorized_keys/mykey.pub`)

8. **Everything should be ready now!** Sit back, relax, and run: `ansible-playbook site.yml -v`


Deploying applications
----------------------

To deploy an application to your new server, you just have to add a remote to an existing git repository and push to it. Starting with an empty directory, it can look something like this:

```bash
git init
echo '<?php echo "Hello my little app!" ?>' > index.php
git add .
git commit -m 'Initial commit.'
git remote add deploy dokku@mylittleapp.org:helloworld
git push deploy master
```

There will now be application deployment magic happening, much like you're used to from Heroku. The application type is auto-detected and everything needed to run it will be installed (inside a [Docker](https://www.docker.io/) container) and launched. To visit your new application, just go to `<appname>.mylittleapp.org`. (In the example above that would be `helloworld.mylittleapp.org`.)

All subdomains that are not pointing at an application will redirect to the main hostname. If you want to run an application at the main hostname, create one with just your domain name as its name:

```bash
git init
mkdir www
echo 'My little home' > www/index.html
touch .nginx
git add .
git commit -m 'Initial commit.'
git remote add deploy dokku@mylittleapp.org:mylittleapp.org
```

Touching the `.nginx` file is how the [nginx buildpack](https://github.com/rhy-jot/buildpack-nginx) detects it's a static application. All the actual files for the site should live in the `www` directory.


Additional notes
----------------

* To give more users access to the setup, just add their public SSH to key the `authorized_keys` directory and run the playbook again.
* By default, [Dokku](https://github.com/progrium/dokku) will be fetched from the `HEAD` of its `master` branch. If you want to use another branch, or a specific tag or commit, just change the `dokku_version=HEAD` part in `hosts`, for example `dokku_version=v0.2.2`.
* For maximum encapsulation, you might want to install and run [Ansible](https://github.com/ansible/ansible) itself from a [virtual environment](http://virtualenvwrapper.readthedocs.org/).


Improvements
------------
The purpose of this deliberately simple setup is to get you up and running as easy as possible. Only what should be useful to most people is included, which will continue to be the criteria for any changes. With that said, there is of course room for much improvement. If you have ideas, please [open an issue](https://github.com/alimony/mylittleapp/issues) about it, or open a pull request directly if you have code to contribute.
