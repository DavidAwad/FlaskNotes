A first look at provisioning with Puppet(on a Vagrant box)

I talked about deploying a Flask app on a Vagrant box using Gunicorn and Nginx. I've decided to write about another neat tool that's awesome for deploying web apps- Puppet. [Vagrant](http://vagrantup.com) actually encourages its users to use it, and you should! **There's also an alternative to Puppet called [Chef](https://www.chef.io/chef/); some people prefer that over Puppet, so you might want to check it out.**

Hopefully I'll be able to demonstrate what Puppet does and why its awesome in this post. But please note that this isn't meant as a comprehensive tutorial, you should check out Puppet's [docs](https://docs.puppetlabs.com/) for that. Also even though the Puppet docs asks you to get the Learning Puppet VM, I found it much more comfortable to use `vagrant ssh` for learning Puppet, so if you already have Vagraant installed, you might want to try that out too- just try running `puppet` inside the virtual machine. 

What Puppet does is something called *provisioning*- that means that it **configures your server for you**. To understand what that means, let's see it in action.

First create a Vagrant box, 

	```shell
	mkdir codebase_with_puppet
	cd codebase_with_puppet

	vagrant init
	```
	
You should now see Vagrantfile. Open it with a text editor, then uncomment the lines

	```puppet
	
	config.vm.provision :puppet do |puppet|
		puppet.manifests_path = "manifests"
		puppet.manifest_file  = "base.pp"
	end
	
	```
	
Now, create `base.pp` inside a folder called manifests, and add the following to it.

	```puppet
	
	package {"nginx":
		ensure => present,
	}
	
	```
Now, run `vagrant up`. You'll notice that Vagrant automatically installs `nginx` after it boots the VM. You should get a message like 

	```shell
	
	notice: /Stage[main]//Package[nginx]/ensure: ensure changed 'purged' to 'present'
	
	```
	
This can become a real treasure as this way, you won't have to memorize what you need to install, to get the app running. All the system needs is to have `puppet` installed, after that `puppet` with the right manifests will handle everything. 

Now, let's do something different with Puppet- instead of installing another package, we'll use it to configure `nginx`. 

First, create a file in your local machine, inside `codebase_with_puppet` called `codebase_nginx`. To that file add the following

	```nginx
	server {
		location / {
			proxy_pass http://127.0.0.1:8000;
		}
	}
	```
	
If you've gone through the previous post you'll notice that it's the same configuration that we had used. 

Now, we'll use Puppet to make sure that the configuration file is placed where its supposed to be. To your `base.pp` file, add 

	```puppet 
	
	file {"rm-nginx-default":
		path => '/etc/nginx/sites-enabled/default',
		ensure => absent,
		require => Package['nginx'],
	}

	file {"setup-nginx-codebase":
		path => '/etc/nginx/sites-enabled/codebase_nginx',
		ensure => present,
		require => Package['nginx'],
		source => "/vagrant/codebase_nginx",
	}
	```
	
Run `vagrant reload` and you're done with the `nginx` configuration. Besides removing the repetitiveness for you, Puppet is also wonderful when you're working on a team or on an open-source project. Now, all you need to do is write the manifests and once you share them you can rest assured that the entire team has the exact same environment.  

	

[vagrant](http://vagrantup.com)
[puppet](http://docs.puppetlabs.com/)
[chef](http://www.opscode.com/chef/)
[scobleizer_puppet](http://www.youtube.com/watch?v=bP8Mtiuc8XM)
