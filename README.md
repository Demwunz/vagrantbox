# **This is old and out of date**. Consider a modern alternative.

![Science](https://gimmebar-assets.s3.amazonaws.com/4fafbdce48c48.jpg)

## Setup Vagrant

We're going to build a custom Vagrant box, so you'll need to download these:

* [Vagrant](http://www.vagrantup.com/downloads.html)
* [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

At some point in the future it will be better to have a [Puppet](http://puppetlabs.com/puppet/what-is-puppet) or [Chef](http://docs.vagrantup.com/v2/provisioning/chef_solo.html) created by a SysAdmin to take care of the below installs, but for now, we'll keep it manual and stick to the script.

#### Initialise Vagrant

We now want to create a Vagrant directory, move into this directory then finally initialise a new vagrant box based upon [Ubuntu Lucid 32](http://www.vagrantbox.es/) and name it whatever you like, e.g 'mycustombox'

    $ cd ~
    $ mkdir Vagrant && cd $_
    $ vagrant init mycustombox http://files.vagrantup.com/lucid32.box

You will now have a **Vagrantfile** file in this new directory. We're going to update it, so drag this file into your [favourite text editor](http://www.sublimetext.com/)

----
###### VMware Fusion

If you're using VMware fusion you will have to run these commands to get it running properly. This assumes you've purchased vmware fusion and also the adapter for vagrant. Download your licence and place it in the same directory as your Vagrantfile.

    $ vagrant plugin install vagrant-vmware-fusion
    $ vagrant plugin license vagrant-vmware-fusion license.lic
    $ vagrant up --provider=vmware_fusion
----

Before we do anything else, lets fire up the box, this may take up to 20 minutes as the image is downloaded to your computer:

    $ vagrant up

once its downloaded, lets SSH in and create a working directory:

    $ vagrant ssh
    vagrant@lucid32:~$ mkdir dev

Now logout and continue with the setup from our own machine.

    vagrant@lucid32:~$ logout

#### Update working directory
Lets create a different folder for our development work to take place. In your home directory create a folder called *Development*, then edit this line in your vagrantfile:

    config.vm.synced_folder "~/Development", "/home/dev"

From now on you will want to do all your development within the Development folder, this will sync with your vagrantbox, so you'll have access to your files at all times.

#### SSH
By default when you create a Vagrant box if uses it own SSH key to allow you to login. However this is a problem if you want to deploy or issue git commands from in the box. We can change the configuration of Vagrant to allow us to use our primary SSH key to login and also forward it so the box can use it as well.

*If you don't have an SSH key, these instructions are easy to follow [https://help.github.com/articles/generating-ssh-keys](https://help.github.com/articles/generating-ssh-keys)*

First copy your ssh public key to the vagrant folder

    $ cp ~/.ssh/id_rsa.pub /path/to/vagrant/box

Then login to your vagrant box and add this key.

    $ vagrant ssh
    vagrant@lucid32:~$ cat /vagrant/id_rsa.pub >> ~/.ssh/authorized_keys
    vagrant@lucid32:~$ exit

Finally update your Vagrantfile with these additional lines in the config block.

    config.ssh.private_key_path = "/Users/YOURUSERNAME/.ssh/id_rsa"
    config.ssh.forward_agent = true

You'll need to edit the private_key_path (same as your public but without the .pub). This path needs to be the full one so don't use ~.

----
###### Windows users
Because Git-Bash does not support SSH Forwarding natively we need to create a script to load the key into an agent.

Create a .bashrc.txt file in your home directory (Normally *C:/Users/username/*) and paste this

    # Note: ~/.ssh/environment should not be used, as it
    #       already has a different purpose in SSH.

    env=~/.ssh/agent.env

    # Note: Don't bother checking SSH_AGENT_PID. It's not used
    #       by SSH itself, and it might even be incorrect
    #       (for example, when using agent-forwarding over SSH).

    agent_is_running() {
        if [ "$SSH_AUTH_SOCK" ]; then
            # ssh-add returns:
            #   0 = agent running, has keys
            #   1 = agent running, no keys
            #   2 = agent not running
            ssh-add -l >/dev/null 2>&1 || [ $? -eq 1 ]
        else
            false
        fi
    }

    agent_has_keys() {
        ssh-add -l >/dev/null 2>&1
    }

    agent_load_env() {
        . "$env" >/dev/null
    }

    agent_start() {
        (umask 077; ssh-agent ]]
    >
    "$env")
        . "$env" >/dev/null
    }

    if ! agent_is_running; then
        agent_load_env
    fi

    if ! agent_is_running; then
        agent_start
        ssh-add
    elif ! agent_has_keys; then
        ssh-add
    fi

    unset env

    Then rename the file in Git Bash

    mv ~/.bashrc.txt ~/.bashrc

Finally reload your Git Bash windows!

----
#### Open custom ports
We can open any port we need i.e accessing a DB or LiveReload.

    #access dev through the browser
    config.vm.network :forwarded_port, guest: 8000, host: 8000
    #LiveReload
    config.vm.network :forwarded_port, guest: 35729, host: 35729

## Install development dependendencies

So far we've added an SSH key, opened a port and synced the working directories. Now we need to update the VM and install our dependencies.

reload vagrant to make sure all the above has worked:

    $ vagrant reload
    $ vagrant ssh

#### RVM
Update Ubuntu and install [RVM](https://rvm.io/):

    vagrant@lucid32:~$ sudo apt-get update
    vagrant@lucid32:~$ sudo apt-get install curl    
    vagrant@lucid32:~$ \curl -L https://get.rvm.io | bash -s stable --ruby

Reload your profile:

    vagrant@lucid32:~$ source ~/.profile

Check to see it was successful and see which version we have:

    vagrant@precise32:/vagrant$ rvm -v
    rvm 1.25.19 (stable) by Wayne E. Seguin <wayneeseguin@gmail.com>, Michal Papis <mpapis@gmail.com> [https://rvm.io/]

And also the Ruby should be at its latest version:

    vagrant@precise32:/vagrant$ rvm list

    rvm rubies

    =* ruby-2.1.1 [ i686 ]

    # => - current
    # =* - current && default
    # * - default

now we're free to install Ruby gems. I would advise installing [Compass](http://compass-style.org/) and [Bundler](http://bundler.io/) globally as you're likely to use these most often.

    vagrant@lucid32:~$ gem update --system
    vagrant@lucid32:~$ gem install bundler
    vagrant@lucid32:~$ gem install compass


#### Git

    vagrant@lucid32:~$ sudo apt-get install git-core

#### NVM

Whilst we can install Node without the need for a version manager, there is a case for using a [version manager](https://github.com/creationix/nvm), particularly because development is rapid and ongoing.

Uninstall any previous Node.js versions you may already have, install NVM by running the following in Terminal:

    vagrant@lucid32:~$ git clone git://github.com/creationix/nvm.git ~/.nvm
    vagrant@lucid32:~$ printf "\n\n# NVM\nif [ -s ~/.nvm/nvm.sh ]; then\n\tNVM_DIR=~/.nvm\n\tsource ~/.nvm/nvm.sh\nfi" >> ~/.bashrc
    vagrant@lucid32:~$ NVM_DIR=~/.nvm
    vagrant@lucid32:~$ source ~/.nvm/nvm.sh

Install Node.js by running the following in Terminal:

    vagrant@lucid32:~$ nvm install v0.10.26
    vagrant@lucid32:~$ nvm alias default 0.10
    vagrant@lucid32:~$ nvm use 0.10


#### Node.js packages
**note: ignore any sudo commands, they're not needed**

Install some useful Node Packages:

     vagrant@lucid32:~$ npm install -g grunt-cli
     vagrant@lucid32:~$ npm install -g express

You should now have a fully functioning Virtual Machine through Vagrant with Ruby and Node installed and ready!
