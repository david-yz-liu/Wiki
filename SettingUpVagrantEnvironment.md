Setting up your Development Environment via Vagrant
===============================================================

Downloading and Installing
--------------------------
If you want to get started on working on MarkUs quickly and painlessly, this is the way to do it.

1. Install [VirtualBox](https://www.virtualbox.org/) and [Vagrant](http://www.vagrantup.com/)
2. Copy the [Vagrantfile](https://raw.githubusercontent.com/MarkUsProject/Markus/master/Vagrantfile) from the MarkUs github site to your machine.

3. `cd` to the repo (make sure you’re in the right directory - it should contain the Vagrantfile)
4. `vagrant up`

This will download a fairly large (3GB) Debian box from the internet, so go [take a walk](http://news.stanford.edu/news/2014/april/walking-vs-sitting-042414.html) or something. This box has GNOME, postgres, subversion, and all of MarkUs’s other dependencies installed. When the download is complete, VirtualBox will run the box in headless mode.

**NOTE:** If, for some reason, it fails and complains about SSH, you most likely have timed out. Check your internet connection attempt to limit network activity to `vagrant up`.



Connecting to your box
----------------------

Next, run `vagrant ssh` to connect to the virtual machine. (If it asks you for a password for vagrant, the password is "vagrant".)

Follow the instructions for [Setting up Git and MarkUs](GitHowTo) to clone the MarkUs repo on the virtual machine.

**NOTE:** It is possible to set up the vitual machine to share folders with the host machine, but in our experience, this is too slow to be a good work environment, and sometimes doesn't work at all.  If you do want to enable shared folders, you can check out that [vagrant documentation](http://docs.vagrantup.com/v2/synced-folders/).

Finally, move the `database.yml` file from the Home directory of the vagrant box to the project’s `config` directory.

Change to the `MarkUs` directory.  Before you can run the server you will need to install or update required gems with `bundle install`.  You may find that the installation of some gems will fail.  Following the instructions in the output and re-running bundle install usually resolves the problem.

Create the directory used for storing repos: `mkdir data/dev/repos`.

Ensure the database is correctly populated by running `bundle exec rake db:reset`

Finally, run `bundle exec rails server` from the project directory. 

You should now be able to access the site from your host machine's browser at `http://0.0.0.0:3000`.


Doing work on Markus's Git branch?
----------------------------------

WARNING: The gitolite instructions are still a work in progress.

The git branch requires some additional setup to your MarkUs instance. This is because the method student repositories are generated differs between SVN and Git.

We will use [Gitolite](http://gitolite.com/gitolite/index.html) to handle authentication.

To install Gitolite, we first need to create a new user. All of the operations below should be run on the virtual machine.

1. `vagrant up` - Start vagrant if it hasn't been yet.
2. `vagrant ssh` - Connect to vagrant if we haven't yet.
3. `ssh-keygen` - Generate a private & public key for the Gitolite admin account.
4. `sudo adduser git` - Gitolite requires a user named `git` to operate.
5. `sudo adduser git sudo` - Add `git` to `sudo` group so we can install Gitolite from there.
6. `sudo cp .ssh/id_rsa.pub /home/git/vagrant.pub` - Copy the public key we created earlier from `vagrant`'s account into `git`'s.
7. `su git` - Switch user. (password is vagrant)
8. `git clone git://github.com/sitaramc/gitolite` - Download the Gitolite binaries.
9. `mkdir bin`
10. `gitolite/install -ln`
11. `export PATH="~/bin:$PATH"` - Add Gitolite to your PATH.
12. `gitolite setup -pk vagrant.pub` - Setup Gitolite using our `vagrant` public key.

Let's test our Gitolite installation:

1. `exit` - Log back into vagrant user.
2. `eval $(ssh-agent -s)` - Start your key agent to hold the private key.
3. `ssh-add ~/.ssh/id_rsa` - Add key to key agent.
4. `ssh -T git@localhost` - Say `yes` if prompted.

You should see the following output:

		hello vagrant, this is git@vagrant-ubuntu-trusty-64 running gitolite3 v3.6.3-9-g2b918fc on git 1.9.1

		 R W    gitolite-admin
		 R W    testing


Now let's setup MarkUs with some additional libraries:

1. `git config --global user.name "Your Name"`
2. `git config --global user.email "your@email.com"`
3. `git clone git@github.com:<your github username>/Markus.git`
4. `sudo apt-get update`
5. `sudo apt-get install cmake libssh2* libgit2*`
6. `sudo gem install bundler` - This may be already installed.
7. `sudo apt-get install ruby1.9.1-dev` - This may be already installed.
8. `bundle install`
9. `cp ~/database.yml config`
10. `mkdir data/dev/repos`
11. `bundle exec rake db:setup`
12. `bundle exec rake db:reset`

At this point you are done. If you receive an `Early EOF` error, make sure `vagrant` has access to the gitolite-admin repo by doing `ssh git@localhost` from the `vagrant` user. If this does not work, then follow the steps outlined [here](http://gitolite.com/gitolite/emergencies.html) to clone the `gitolite-admin` repo manually somewhere and add the vagrant public key to the `keys` folder.


