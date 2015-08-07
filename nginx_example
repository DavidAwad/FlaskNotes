# Flask + Nginx + Gunicorn(on a Vagrant box)

I had some difficulty in grasping how exactly to set up a server when I tried to do so recently, so I decided to write a tutorial that will guide you through the process. Hopefully, this post will help you avoid at least some of the confusion that I encountered.

We'll be using Nginx + Gunicorn to host a simple Flask app. Many of you may not have access to a server but don't worry, we'll use [Vagrant][vagrant], which makes use of a VirtualBox VM to emulate a server.

###The Flask App

Because this is a post about deployment more than development, we'll make the web app super-simple. If you're not familiar with Flask, please [check it out][flask], its awesome and really easy to learn. You'll also probably want to develop the app inside [virtualenv][virtualenv]- it makes things a lot neater. Make a folder in your local machine(we're not working with the virtual-machine yet) for your app, I'll call it `codebase`. Create two folders called `static` and `templates`, and a Python file called `app.py`. `codebase` should now look like this:

    .
    ├── app.py
    ├── static
    └── templates

Now, open `app.py` with a text editor and add the following:

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return "Hello world!"

if __name__ == '__main__':
    app.run()
```

At this point, if you run `app.py` with `python app.py`, you should be able to open http://localhost:5000/ and see a "Hello World!" printed. Now, freeze your requirements with

```shell
pip freeze > requirements.txt
```
Great, now we'll start working on the actual server.

### Vagrant

As I said before, Vagrant allows you to work with server-like environments on your local machine. It's absolutely great. To get [Vagrant][vagrant] up and running:

```shell
# first make sure Virtualbox is installed, then,
gem install vagrant
vagrant box add base http://files.vagrantup.com/precise32.box
vagrant init
vagrant up
```

If nothing went wrong, you should now see a file called `Vagrantfile` inside `codebase`- that's Vagrant's configuration file. Open the file, we'll need to make a few changes to the file.

First, uncomment the line:

```shell
config.vm.network :hostonly, "192.168.33.10"
If nothing went wrong, you should now see a file called `Vagrantfile` inside `codebase`- that's Vagrant's configuration file. Open the file, we'll need to make a few changes to the file.
```

and change "192.168.33.10" to "33.33.33.33". This will enable the host-machine(that is your computer) to access the webserver running on the VM.

That way we should be able to access a web app running in the VM's `localhost`, on our machine.

Because, we did a `vagrant up` the Vagrant box should already be running. Now, run

```shell
vagrant reload
```

so that the changes we made to the Vagrantfile take place.

After the VM restarts, run

```shell
vagrant ssh
```

This allows you to run commands into the VM. Once inside the VM, we'll need to get some things installed. Run

```shell
apt-get install nginx
pip install virtualenv
```

Now let's create a folder inside the VM where we'll keep the application

```shell
cd /home/vagrant
mkdir www
cd www
virtualenv --no-site-packages .
mkdir codebase
```

And let's grab the application from our local machine

```shell
cp /vagrant/* /home/vagrant/www/codebase/
```
*Note that while I used `cp`, its always a better idea to use `git` or some other version-control system. For more on that, I recommend that you read [this post][pelletier].*

Then, activate the virtualenv we created.

```shell
cd /home/vagrant/www
source bin/activate
```

Install gunicorn with `pip`

```shell
pip install gunicorn
```

Also install the other Python dependencies your app has with

```shell
pip install -r requirements.txt
```

That will grab and install your app's required dependencies like Flask.

Now, if you run

```shell
gunicorn /home/vagrant/www/codebase/app.py:app -b 127.0.0.1:8000
```
you'll have your app running but if you try opening it from your browser you'll find that you can't actually see the "Hello World" message that we were expecting. That's where nginx comes in.

### Nginx

First of all, you need to start nginx with

```shell
/etc/init.d/nginx start
```

Then

```shell
rm /etc/nginx/sites-enabled/default
touch /etc/nginx/sites-available/codebase
ln -s /etc/nginx/sites-available/codebase  /etc/nginx/sites-enabled/codebase
```

To `/etc/nginx/sites-enabled/codebase` add

```shell
server {
        location / {
                proxy_pass http://127.0.0.1:8000;
        }
}
```

And restart nginx with

```shell
/etc/init.d/nginx restart
```
Now, from inside `codebase` run

```shell
gunicorn app:app -b localhost:8000
```

If everything went right, if you visit [http://33.33.33.33/][33] you should now see the "Hello World!" message. Congratulations! You've successfully set up your own server.

**Update-** *I've written a follow-up to this post which covers Puppet, a really handy tool that's comes packaged with Vagrant- you can [find the post here][follow_up].*

[vagrant]: http://vagrantup.com
[flask]: http://flask.pocoo.org/
[pelletier]: http://thomas.pelletier.im/2011/04/git-django-deployment/
[virtualenv]: http://pypi.python.org/pypi/virtualenv
[33]: http://33.33.33.33/

