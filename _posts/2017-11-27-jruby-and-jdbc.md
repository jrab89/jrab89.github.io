---
layout: post
title:  JRuby and JDBC
date:   2017-11-27 14:00:00 -0600
categories: jruby ruby java
---

At work, I was asked to provide a JDBC database URL for an [Amazon Aurora](https://aws.amazon.com/rds/aurora/) database that I setup. Despite having written a fair amount of Java in college, I'm definitely not an expert when it comes to the huge ecosystem of Java libraries and tooling. I've also never used JDBC before! So in order to avoid having to read a bunch of [Oracle documentation](http://www.oracle.com/technetwork/java/javase/tech/index-jsp-136101.html) and setup [Java](https://www.jetbrains.com/idea/) [stuff](https://gradle.org/) on my laptop, I decided to try using [JRuby](http://jruby.org/) to verify that the JDBC database URL I was handing off actually worked.

JRuby is an implementation of Ruby that's written in Java instead of C. The current version of JRuby (9.1.x) is compatible with [MRI](https://en.wikipedia.org/wiki/Ruby_MRI) 2.3.x. But why bother with JRuby when you could just use MRI? JRuby allows you to embed a Ruby interpreter into Java applications and use Java libraries from your Ruby applications; you can run Ruby from your Java and Java from your Ruby! JRuby also has the advantage of being able to leverage JVM threads without being limited by a global interpreter lock, so you can have Ruby code running in parallel in the same process!

## Installation

On a Mac, you can install JRuby with [Homebrew](https://brew.sh/). [RVM](https://rvm.io/) also works.

```
$ brew install jruby
Sleeping for 1s...
==> Downloading https://search.maven.org/remotecontent?filepath=org/jruby/jruby-dist/9.1.14.0/jruby-dist-9.1.14.0-bin.tar.gz
######################################################################################################################         100.0%
🍺  /usr/local/Cellar/jruby/9.1.14.0: 1,305 files, 27.0MB, built in 1 minute 25 seconds
```

Homebrew prefixes JRuby's executables with "j". So if you wanted to run `rake` with JRuby you'd run `jrake`. For MRI you'd still run `rake`.

```
$ ls -althrG /usr/local/bin | grep rake
-rwxr-xr-x    1 jrabovsky  admin   476B Jan  3  2017 rake
lrwxr-xr-x    1 jrabovsky  admin    34B Nov 24 20:28 jrake -> ../Cellar/jruby/9.1.14.0/bin/jrake
```

If you `jgem install` gems with executables, those executables won't be in your `PATH`. To run JRuby gems' executables, use JRuby's `jruby -S`. The `-S ` option tells JRuby to use __its__ version of the executable.

```
$ jgem install rspec
Fetching: rspec-support-3.7.0.gem (100%)
Successfully installed rspec-support-3.7.0
Fetching: rspec-core-3.7.0.gem (100%)
Successfully installed rspec-core-3.7.0
Fetching: diff-lcs-1.3.gem (100%)
Successfully installed diff-lcs-1.3
Fetching: rspec-expectations-3.7.0.gem (100%)
Successfully installed rspec-expectations-3.7.0
Fetching: rspec-mocks-3.7.0.gem (100%)
Successfully installed rspec-mocks-3.7.0
Fetching: rspec-3.7.0.gem (100%)
Successfully installed rspec-3.7.0
6 gems installed
$ rspec
bash: rspec: command not found
$ jgem list

*** LOCAL GEMS ***

did_you_mean (default: 1.0.1)
diff-lcs (1.3)
jar-dependencies (default: 0.3.10)
jruby-openssl (default: 0.9.21 java)
jruby-readline (default: 1.1.1 java)
json (default: 1.8.3 java)
minitest (default: 5.4.1)
net-telnet (default: 0.1.1)
power_assert (default: 0.2.3)
psych (default: 2.2.4 java)
rake (default: 10.4.2)
rdoc (default: 4.2.0)
rspec (3.7.0)
rspec-core (3.7.0)
rspec-expectations (3.7.0)
rspec-mocks (3.7.0)
rspec-support (3.7.0)
test-unit (default: 3.1.1)
$ jruby -S rspec
No examples found.


Finished in 0.02254 seconds (files took 0.5004 seconds to load)
0 examples, 0 failures

```

## Java Interoperability

To run Java code from JRuby, you first need to `require 'java'`. This allows you to access Java classes in the classpath by specifying their full name. For example, the following calls the static [getProperties](https://docs.oracle.com/javase/7/docs/api/java/lang/System.html#getProperties()) method on the [System](https://docs.oracle.com/javase/7/docs/api/java/lang/System.html) class within the [java.lang](https://docs.oracle.com/javase/7/docs/api/java/lang/package-summary.html) package.

```ruby
require 'java'

puts java.lang.System.properties['java.runtime.version']
```

The `getProperties` method returns an instance of the [Properties](https://docs.oracle.com/javase/7/docs/api/java/util/Properties.html) class, which implements the [Map](https://docs.oracle.com/javase/7/docs/api/java/util/Map.html) interface. JRuby conveniently allows you to treat this object like a Ruby [Hash](https://ruby-doc.org/core-2.4.2/Hash.html). Another nice thing JRuby does here is that it allows calling the Java method `getProperties` with the more Ruby-like method name of `properties`.

## JDBC

JDBC's classes are in the [java.sql](https://docs.oracle.com/javase/7/docs/api/java/sql/package-summary.html) and [javax.sql](https://docs.oracle.com/javase/7/docs/api/javax/sql/package-summary.html) Java packages.

In addition to JDBC itself, you also need a JDBC driver for the particular DBMS you're working with. JDBC drivers are client-side adapters that implement the network protocols for different databases. The official JDBC driver for MySQL is packaged as the [jdbc-mysql](https://github.com/jruby/activerecord-jdbc-adapter/tree/master/jdbc-mysql) Ruby gem.

Here's how you'd run a MySQL Docker container and then use JRuby and JDBC to execute a query against it:

```
$ docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=database -d mysql:5.7
b115964fab20bfe6cddae01ab78b1dfeb8b07ff6cf9ac722b7c2f49fff781898
$ jgem install jdbc-mysql
Fetching: jdbc-mysql-5.1.44.gem (100%)
Successfully installed jdbc-mysql-5.1.44
1 gem installed
$ jruby hello_jdbc.rb
Hello JDBC!
```

Where `hello_jdbc.rb` contains:

```ruby
require 'java'
require 'jdbc/mysql'

Jdbc::MySQL.load_driver

database_url = 'jdbc:mysql://localhost:3306/database'
user = 'root'
password = 'password'
query = 'SELECT "Hello JDBC!"'

connection = java.sql.DriverManager.get_connection(database_url, user, password)
result_set = connection.create_statement.execute_query(query)
result_set.next?
puts result_set.get_string(1)
```
