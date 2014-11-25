EngageNY.org
============

Vagrant for EngageNY.org

Setup
-----

1. Install Vagrant, Virtualbox, Git, and Drush

  Requires Vagrant 1.5+.

  See http://vagrant-up.com/downloads and https://www.virtualbox.org/wiki/Downloads for binary installers.
  See https://github.com/drush-ops/drush for info on installing drush on your system.

2. Highly suggested: 

  Install vagrant-faster plugin.  This module will automatically scale up your vagrant box based on your host machine.
  
  ```
  vagrant plugin install vagrant-faster
  ```

2. Clone this repo if you haven't already.
  ````
  git clone git@github.com:pcgroup/engage-vagrant.git engageny
  cd engageny
  ````

3. Raise the virtual machine.
  ````
  vagrant up
  ````

  Vagrant will automatically clone the project repo to ./src on first up.

4.  Add this line to your hosts file to allow local.engageny.org to point to 127.0.0.1

  This Vagrantfile uses port forwarding so the VM is accessible from localhost/127.0.0.1

  ````
  # MacOS/Linux: /etc/hosts
  # Windows: C:\Windows\System32\drivers\etc\hosts
  1.2.3.4       local.engageny.org
  ````

5. Copy the database from another server. It is faster and more reliable to do this from within the vagrant box.

  Also, I find it more reliable to sql-dump then sqlc.

  ````
  vagrant ssh
  drush @engageny2.prod sql-dump > prod.sql
  drush @engage sqlc < prod.sql
  ````
  After syncing from production, you usually need to rebuild the registry:
  
  ```
  drush @engage rr
  ```
  
  Then, use `drush @engage upgradepath` to run all the things needed to rebuild the site.
  The command includes reverting all features, clearing all caches, and running all updates.
  
  It can take a long time the first time you sync from production if there are a lot of updates to do.

6. Visit http://local.engageny.org.

NOTES
-----

Drush aliases are created both on your host machine and inside the vagrant box called @engage.
