# Debugging BioPortal

The first edition of this guide assumes that one is running Linux and that the IDE of choice is RubyMine.  As more than one person edits this in this directory the documentation will become more generic. Where it is known it will be indicated that a given solution is RubyMine or Linux specific.I also tend to favour running the infrastructure in docker to avoid lengthy installs to setup a developement environment.

## Running stuff

In order to expect anything to work one needs to have the infrastructure running.  This can be quickly provided by docker but some of out developers have it running on their development machines.  The docker solution will be described later in this note ([infrastructure through docker](#docker)) and the install option may be later described by other developers.

### Tests

Tests for a project are often run as rake tasks:
```
		bundler exex rake test
```
The ncbo project has a bunch of other useful rake tasks that we will find documentatioin for later.

### Rest API

The ontologies rest services are run via rackup from the ontologies_api project:
```
		bundler exec rackup
```
The difference betweeen rackup and rake is that rackup is optimized for web services.  Each service is implemented as a      `call` interface call where call takes one argument. I am not yet sure where rackup is configured.

#### Case Study: Missing config file

I saw the error:
```
bundler: failed to load command: rackup (/home/tredmond/.rbenv/versions/2.7.6/bin/rackup)                                                                              
Traceback (most recent call last):
            ...
         4: from /home/tredmond/.rbenv/versions/2.7.6/lib/ruby/gems/2.7.0/gems/rack-1.6.13/lib/rack/builder.rb:55:in `instance_eval'                                   
         3: from /home/tredmond/BioPortal/projects/ontologies_api/config.ru:1:in `block in <main>'                                                                     
         2: from /home/tredmond/BioPortal/projects/ontologies_api/config.ru:1:in `require'                                                                             
         1: from /home/tredmond/BioPortal/projects/ontologies_api/app.rb:77:in `<top (required)>'
/home/tredmond/BioPortal/projects/ontologies_api/app.rb:77:in `require_relative': cannot load such file -- /home/tredmond/BioPortal/projects/ontologies_api/config/environments/development.rb (LoadError) 
```
This error is what it looks like: a missing file which happens to be a config file.  In that directory there is a file called `config.rb.sample` which can be used as a configuration file, development.rb.  One of the important things that this file does is to specify where the servers are including port numbers.


### Main User Interface

I don't quite have it working yet but I also know the command to start the rails user web user interface.  To do this start the ontologies api (the rackup command) and then run
```
rails s
```
There is a bit more to this as rails needs to be configured (including setting up a mariadb or mysql database) and there is an exception that occues when adding myself as a user.  

#### Case Study: MySql problems

Unlike the rest API, the ruby on rails application uses mysql which brings its own problems.  When I first do a `bundle install` I get the error when making the mysql2 gem:
```
*** extconf.rb failed ***                                                                                                                                                      
Could not create Makefile due to some reason, probably lack of necessary libraries and/or headers.  Check the mkmf.log file for more details.  You may need configuration options. 
```
The Makefile is generally created by autoconf which checks if certain libraries are present. So the solution to this problem is to add the C/C++ libraries on which this compile depends which in Linux using mariadb would be done with the command:
```
sudo dnf install mariadb-connector-c mariadb-connector-c-devel
```
Then 
```
gem install mysql2
```
and 
```
bundle install
```
should work.

#### Setting the Mariadb root password


After installing mariadb with the command,
```
 dnf install mariadb-server
```
it is not clear what the root password is or how to login as root (in order to set such password). Fortunately the command,
```
sudo mysql -u root
```
works. Then one can set the root password with the command
```
set password for 'root'@'localhost' = password('mypass');
```
can be used to set the root password. One must then set database.yml (there is a sample file) in the config directory to correspond.

You need to install nodejs to defeat the error `Could not find a JavaScript runtime`:
```
sudo dnf install nodejs
```
After a bit of work (there are two sample config files) you can run the command
```
bundler exec rake db:create
```
to create the databases `bioportal_ui_test` and `bioportal_ui_dev`.  (I did not report this work here because I had to modify some files so they would compile which I found suspicious.)

## Rbenv and versions of Ruby

Unfortunately, one cannot simply install the latest version of ruby and expect that things would work.  One needs to choose the appropriate version of ruby.  Ruby is too essential to the development to see how to run it only in docker.  Thus it is useful to have a tool that allows one to select and use a specific version of ruby, possibly on a project specific basis.  We often use rbenv for this though there are other tools that are useful (and possibly more secure) for this purpose. A discussion of the installation and use of rbenv can be found [here](https://github.com/rbenv/rbenv/blob/master/README.md).

## <a name="docker"></a>Running from the command line with docker

The motivation behind this approach for running things under docker is that it means that the developer has an "instant" infrastructure rather than having to install and configure it himself, a costly process. Docker will also hide the configuration of the host machine. Installing and configuring the infrastructure on a Mac will proceed slightly differently than it would on a linux machine. With docker, the configuration and installation only needs to be done once, by a devops person, on a single machine.

There are other ways of using docker# Debugging BioPortal

The first edition of this guide assumes that one is running Linux and that the IDE of choice is RubyMine.  As more than one person edits this document will become more generic and where it is known it will be indicated that a given solution is RubyMine or Linux specific.I also tend to favour running the Infrastructure in docker to avoid lengthy installs to seeup a developement environment.

## Running stuff

In order to expect anything to work one needs to have the infrastructure running.  This can be quickly provided by docker but some of out developers have it running on their development machines.  The docker solution will be described later in this note ([infrastructure through docker](#docker)) and the install option may be later described by other developers.

### Tests

Tests for a project are often run as rake tasks:
```
		bundler exec rake test
```
Rake is a tool for writing ruby scripts that do something useful.  The ncbo project, in particular, has a bunch of useful rake tasks that we will find documentation for later. The rails project has a rake task, create:db, which installs a database. 

### Rest API

The ontologies rest services are run via rackup from the ontologies_api project:
```
		bundler exec rackup
```
The difference betweeen rackup and rake is that rackup is optimized for web services.  Each service is implemented as a      `call` interface call where call takes one argument. I am not yet sure where rackup is configured.

#### Case Study: Missing config file

I saw the error:
```
bundler: failed to load command: rackup (/home/tredmond/.rbenv/versions/2.7.6/bin/rackup)                                                                              
Traceback (most recent call last):
            ...
         4: from /home/tredmond/.rbenv/versions/2.7.6/lib/ruby/gems/2.7.0/gems/rack-1.6.13/lib/rack/builder.rb:55:in `instance_eval'                                   
         3: from /home/tredmond/BioPortal/projects/ontologies_api/config.ru:1:in `block in <main>'                                                                     
         2: from /home/tredmond/BioPortal/projects/ontologies_api/config.ru:1:in `require'                                                                             
         1: from /home/tredmond/BioPortal/projects/ontologies_api/app.rb:77:in `<top (required)>'
/home/tredmond/BioPortal/projects/ontologies_api/app.rb:77:in `require_relative': cannot load such file -- /home/tredmond/BioPortal/projects/ontologies_api/config/environments/development.rb (LoadError) 
```
This error is what it looks like: a missing file which happens to be a config file.  In that directory there is a file called `config.rb.sample` which can be used as a configuration file, development.rb.  One of the important things that this file does is to specify where the servers are including port numbers.


### Main User Interface

I don't quite have it working yet but I also know the command to start the rails user web user interface.  To do this start the ontologies api (the rackup command) and then run
```
rails s
```
There is a bit more to this as rails needs to be configured (including setting up a mariadb or mysql database) and there is an exception that occues when adding myself as a user.  

#### Case Study: MySql problems

Unlike the rest API, the ruby on rails application uses mysql which brings its own problems.  When I first do a `bundle install` I get the error when making the mysql2 gem:
```
*** extconf.rb failed ***                                                                                                                                                      
Could not create Makefile due to some reason, probably lack of necessary libraries and/or headers.  Check the mkmf.log file for more details.  You may need configuration options. 
```
The Makefile is generally created by autoconf which checks if certain libraries are present. So the solution to this problem is to add the C/C++ libraries on which this compile depends which in Linux using mariadb would be done with the command:
```
sudo dnf install mariadb-connector-c mariadb-connector-c-devel
```
Then 
```
gem install mysql2
```
and 
```
bundle install
```
should work.

#### Setting the Mariadb root password


After installing mariadb with the command,
```
 dnf install mariadb-server
```
it is not clear what the root password is or how to login as root (in order to set such password). Fortunately the command,
```
sudo mysql -u root
```
works. Then one can set the root password with the command
```
set password for 'root'@'localhost' = password('mypass');
```
can be used to set the root password. One must then set database.yml (there is a sample file) in the config directory to correspond.

You need to install nodejs to defeat the error `Could not find a JavaScript runtime`:
```
sudo dnf install nodejs
```
After a bit of work (there are two sample config files) you can run the commands:
```
bundler exec rake db:create
bundler exec rake db:migrate
```
to create the databases `bioportal_ui_test` and `bioportal_ui_dev`.  (I did not report this work here because I had to modify some files so they would compile which I found suspicious.)

## Rbenv and versions of Ruby

Unfortunately, one cannot simply install the latest version of ruby and expect that things would work.  One needs to choose the appropriate version of ruby.  Ruby is too essential to the development to see how to run it only in docker.  Thus it is useful to have a tool that allows one to select and use a specific version of ruby, possibly on a project specific basis.  We often use rbenv for this though there are other tools that are useful (and possibly more secure) for this purpose. A discussion of the installation and use of rbenv can be found [here](https://github.com/rbenv/rbenv/blob/master/README.md).

## <a name="docker"></a>Running from the command line with docker

The motivation behind this approach for running things under docker is that it means that the developer has an "instant" infrastructure rather than having to install and configure it himself, a costly process. Docker will also hide the configuration of the host machine. Installing and configuring the infrastructure on a Mac will proceed slightly differently than it would on a linux machine. With docker, the configuration and installation only needs to be done once, by a devops person, on a single machine.

There are other ways of using docker to support development, including operating the development tools inside a docker container or performing a remote debug to a docker container from the host machine (probably requires that the sources inside the docker container be synchronized with those used by the host development environment).  These approaches may be described in detail and motivated elsewhere.

There are many docker-compose files that create a running infrastructure.  I use the one in the ontoportal_utilities project (which is planned on being moved).  This docker image set provides
+	solr
+	redis
+	mgrep and
+	4store

which is sufficient to run to run all the ReST services of BioPortal.

To run this version of docker, go to the docker directory in the ontoportal_utilities project:
```
cd ontoportal_utilities/docker
```
and execute the command:
```
docker-compose -f docker-compose.dev.yml up -d
```
After this command `docker ps` shows
```
[tredmond@Titan docker]$ docker ps 
CONTAINER ID   IMAGE                       COMMAND                  CREATED             STATUS             PORTS                                                 NAMES
f16e39768cdf   ontoportal/mgrep-ncbo:0.1   "/sbin/tini -- /bin/…"   About an hour ago   Up About an hour   0.0.0.0:55555->55555/tcp, :::55555->55555/tcp         docker_mgrep_1
d1b49dfad9f3   bde2020/4store              "bash -c '4s-backend…"   About an hour ago   Up About an hour   6733/tcp, 0.0.0.0:9000->9000/tcp, :::9000->9000/tcp   docker_4store_1
c249920ad2a3   ontoportal/solr-ut:0.1      "docker-entrypoint.s…"   About an hour ago   Up About an hour   0.0.0.0:8983->8983/tcp, :::8983->8983/tcp             docker_solr_1
1e0a2e162ed0   redis                       "docker-entrypoint.s…"   About an hour ago   Up About an hour   0.0.0.0:6379->6379/tcp, :::6379->6379/tcp             docker_redis_1
[tredmond@Titan docker]$ 
```

Now on the command line you should be able to run commands such as
```
		bundle install
		bundler exec rake
```
to run the tests.

## Running from RubyMine

Running from RubyMine is, at least in theory, simple once you have the code  One catch which may explain why this is sometimes tricky for me is that you need to be sure that RubyMine is running the same versions of ruby and bundler as the command line environment. The ruby version can be set here:

	Settings->Languages & Frameworks->Ruby SDK and Gems

but I think (maybe) the bundler version needs to be set on the RubyMine command line.

To convince RubyMine to run the program, you need to tell it what to do with a Run/Debug configuration as follows:

![Runnable-Screenshot](./images/Runnable-screenshot.png "Tests Runnable")

This tells RubyMine to run the configuration as a rake task (default) in the 'goo' projects directory, with the argument
``` 
TEST=test/test_chunks_write.rb
```
and the environment
```
TESTOPTS="--name=test_reentrant_queries"
```
This is a real runnable from my work on the goo test and it says to run the test_reentrant_queries test from the file test/test_chunks_write.rb.  It took a little bit of figuring out but it was very useful to pick a particular test.

### Troubleshooting: Bundler version problems

A key responsibity of the bundler is to define the dependant libraries and to execute things in that context.  Therefore, when you get exceptions that do not seem related to a BioPortal bug that  indicate a problem with depe dent ruby code then it suggeeets a problem with the bundler.  These exceptions often aee geneeated by "require" statements.  An example of such an exception may be generated by running "rake" without the bundler:
```
[tredmond@Titan goo]$ rake
Warning: you should require 'minitest/autorun' instead.
Warning: or add 'gem "minitest"' before 'require "minitest/autorun"'
From:
  /home/tredmond/BioPortal/projects/goo/test/test_case.rb:12:in `<top (required)>'
  /home/tredmond/BioPortal/projects/goo/test/test_basic_persistence.rb:1:in `require_relative'
  /home/tredmond/BioPortal/projects/goo/test/test_basic_persistence.rb:1:in `<top (required)>'
  /home/tredmond/.rbenv/versions/2.7.6/lib/ruby/gems/2.7.0/gems/rake-13.0.6/lib/rake/rake_test_loader.rb:21:in `block in <main>'
  /home/tredmond/.rbenv/versions/2.7.6/lib/ruby/gems/2.7.0/gems/rake-13.0.6/lib/rake/rake_test_loader.rb:6:in `select'
  /home/tredmond/.rbenv/versions/2.7.6/lib/ruby/gems/2.7.0/gems/rake-13.0.6/lib/rake/rake_test_loader.rb:6:in `<main>'
MiniTest::Unit.autorun is now Minitest.autorun. From /home/tredmond/BioPortal/projects/goo/test/test_case.rb:13:in `<top (required)>'
Traceback (most recent call last):
        11: from /home/tredmond/.rbenv/versions/2.7.6/lib/ruby/gems/2.7.0/gems/rake-13.0.6/lib/rake/rake_test_loader.rb:6:in `<main>'
        10: from /home/tredmond/.rbenv/versions/2.7.6/lib/ruby/gems/2.7.0/gems/rake-13.0.6/lib/rake/rake_test_loader.rb:6:in `select'
         9: from /home/tredmond/.rbenv/versions/2.7.6/lib/ruby/gems/2.7.0/gems/rake-13.0.6/lib/rake/rake_test_loader.rb:21:in `block in <main>'
         8: from /home/tredmond/.rbenv/versions/2.7.6/lib/ruby/2.7.0/rubygems/core_ext/kernel_require.rb:83:in `require'
         7: from /home/tredmond/.rbenv/versions/2.7.6/lib/ruby/2.7.0/rubygems/core_ext/kernel_require.rb:83:in `require'
         6: from /home/tredmond/BioPortal/projects/goo/test/test_basic_persistence.rb:1:in `<top (required)>'
         5: from /home/tredmond/BioPortal/projects/goo/test/test_basic_persistence.rb:1:in `require_relative'
         4: from /home/tredmond/BioPortal/projects/goo/test/test_case.rb:15:in `<top (required)>'
         3: from /home/tredmond/BioPortal/projects/goo/test/test_case.rb:15:in `require_relative'
         2: from /home/tredmond/BioPortal/projects/goo/lib/goo.rb:4:in `<top (required)>'
         1: from /home/tredmond/.rbenv/versions/2.7.6/lib/ruby/2.7.0/rubygems/core_ext/kernel_require.rb:146:in `require'
/home/tredmond/.rbenv/versions/2.7.6/lib/ruby/2.7.0/rubygems/core_ext/kernel_require.rb:146:in `require': cannot load such file -- sparql/client (LoadError)
rake aborted!
Command failed with status (1)

Tasks: TOP => default => test
(See full trace by running task with --trace)
[tredmond@Titan goo]$ 
```
The end of this exception is very similar to something that I saw when running the rake command inside ruby.  To fix this I ran the following commands inside a RubyMine terminal:
```
gem install bundler
bundler init
bundle install
```
The init command may be important - in spite of the fact that it generated an error suggesting that it did nothing - becuase things did not seem to work without it,


#### Case Study: Bundler version problems

I recently saw the same exception that had bothered me so much before:
```
/bin/bash -c "env RBENV_VERSION=2.7.0 /home/tredmond/.rbenv/libexec/rbenv exec ruby /home/tredmond/.rbenv/versions/2.7.0/bin/rake default TEST=test/test_chunks_write.rb"
Warning: you should require 'minitest/autorun' instead.
Warning: or add 'gem "minitest"' before 'require "minitest/autorun"'
From:
  /home/tredmond/BioPortal/projects/goo/test/test_case.rb:19:in `<top (required)>'
  /home/tredmond/BioPortal/projects/goo/test/test_chunks_write.rb:1:in `require_relative'
  /home/tredmond/BioPortal/projects/goo/test/test_chunks_write.rb:1:in `<top (required)>'
  /home/tredmond/.rbenv/versions/2.7.0/lib/ruby/gems/2.7.0/gems/rake-13.0.6/lib/rake/rake_test_loader.rb:21:in `block in <main>'
  /home/tredmond/.rbenv/versions/2.7.0/lib/ruby/gems/2.7.0/gems/rake-13.0.6/lib/rake/rake_test_loader.rb:6:in `select'
  /home/tredmond/.rbenv/versions/2.7.0/lib/ruby/gems/2.7.0/gems/rake-13.0.6/lib/rake/rake_test_loader.rb:6:in `<main>'
MiniTest::Unit.autorun is now Minitest.autorun. From /home/tredmond/BioPortal/projects/goo/test/test_case.rb:20:in `<top (required)>'
Traceback (most recent call last):
	11: from /home/tredmond/.rbenv/versions/2.7.0/lib/ruby/gems/2.7.0/gems/rake-13.0.6/lib/rake/rake_test_loader.rb:6:in `<main>'
	10: from /home/tredmond/.rbenv/versions/2.7.0/lib/ruby/gems/2.7.0/gems/rake-13.0.6/lib/rake/rake_test_loader.rb:6:in `select'
	 9: from /home/tredmond/.rbenv/versions/2.7.0/lib/ruby/gems/2.7.0/gems/rake-13.0.6/lib/rake/rake_test_loader.rb:21:in `block in <main>'
	 8: from /home/tredmond/.rbenv/versions/2.7.0/lib/ruby/2.7.0/rubygems/core_ext/kernel_require.rb:92:in `require'
	 7: from /home/tredmond/.rbenv/versions/2.7.0/lib/ruby/2.7.0/rubygems/core_ext/kernel_require.rb:92:in `require'
	 6: from /home/tredmond/BioPortal/projects/goo/test/test_chunks_write.rb:1:in `<top (required)>'
	 5: from /home/tredmond/BioPortal/projects/goo/test/test_chunks_write.rb:1:in `require_relative'
	 4: from /home/tredmond/BioPortal/projects/goo/test/test_case.rb:22:in `<top (required)>'
	 3: from /home/tredmond/BioPortal/projects/goo/test/test_case.rb:22:in `require_relative'
	 2: from /home/tredmond/BioPortal/projects/goo/lib/goo.rb:4:in `<top (required)>'
	 1: from /home/tredmond/.rbenv/versions/2.7.0/lib/ruby/2.7.0/rubygems/core_ext/kernel_require.rb:92:in `require'
/home/tredmond/.rbenv/versions/2.7.0/lib/ruby/2.7.0/rubygems/core_ext/kernel_require.rb:92:in `require': cannot load such file -- sparql/client (LoadError)
rake aborted!
Command failed with status (1)

Tasks: TOP => default => test
(See full trace by running task with --trace)

Process finished with exit code 1
```
I did several iterations of looking at the bundler version in settings (I eyentually managed to set it there) and executing bundle initialization commands on the ruby and machine command  line to no avail. A new version of the bundler # Debugging BioPortal


