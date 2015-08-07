# Trac

Installs and configures Trac instances. Supports external PostgreSQL database/schema.

# Module Description

Trac is a wiki and issue tracking system for software development projects. This module
will aid in quickly provisioning a new Trac environment.

Currently, the module supports creation of either Subversion or Git repositories, and
either Sqlite or PostgreSQL for the backend database. An apache virtualhost and logo image
for you project will also be automatically created for you.

This module is an expansion of the eschiller trac module. It adds the option for a PostgreSQL
database on a different server and the use of an alternative schema.

# Requirements

 * puppetlabs/apache ( version 1.0.1 or greater )
 * puppetlabs/concat
 * puppetlabs/stdlib
 * puppetlabs/vcsrepo
 * puppetlabs/firewall (if trac::tracenv::open_firewall is true)
 * puppetlabs/postgresql (if using postgresql)
 
This is release 0.2.1, 2015-08-04.

# Caveats

This module attempts to construct a Trac site from scratch. If the database schema already
exists OR if Trac cannot create a schema, problems will ensue, including failure to create
the necessary apache wsgi file that makes the Trac instance work.

No support at present exists for MySQL trac databases.
No support at present for handling Trac version upgrades that might alter the database schema.

# Usage

Most of the heavy lifting in the module is done by the trac::tracenv define, including
creating the config files for the trac environment and calling other defines which create
the database, apache virtualhost, code repository and logo image for the Trac install. 

Once the Trac project instance is installed by Puppet, you will need to use the "htdigest"
utility on the target machine to set up user IDs and passwords for the project. Although
a future release may allow other security options. LDAP is being considered.

*Note*

When using external PostgreSQL database, make sure that the database user or host you're
using with the Trac system is already empowered for access in the remote database pg_hba.conf
file. Failure to attend to this will keep the trac instance from creating the prototype schema
and may even result in the wrong schema being used! If you forget, fix pg_hba, restart PostgreSQL,
rip out the Trac project (rm -rf /trac/screwedupproject or whatever the "envpath" was) and re-run
the Puppet agent.

Tracenv defines the basic characteristics of a Trac instance. You can have multiple instances
on a single machine, each with its own database (and schema). Just define multiple tracenv's.

In order to call a tracenv though, you'll first need to declare the trac class.

<pre>
  class{'trac':}
</pre>

After the trac class has been declared, you can call a trac::tracenv to install a new
Trac environment.

<pre>
  trac::tracenv{'test':
    redir_http => true,
    vhost_name => '*',
  }
</pre>

Note that passing a value to the vhost_name parameter is required (this is to avoid
a conflict if creating multiple trac environments.)

If you'd like to create an environment with a postgres database and a git repository,
it can be done as follows:

<pre>
  trac::tracenv{'test':
    redir_http => true,
    vhost_name => '*',
    repo_type  => 'git',
    db_server  => 'pg_server.mydomain.com',
    db_schema  => 'test',
    db_type    => 'postgres',
    # Also db_user and db_password if authenticating via userid.
  }
</pre>

If not specified, repo type will default to 'svn' and db type will default to 'sqlite'.
For a db_type of 'postgres', a local PostgreSQL server will be automatically installed,
but ONLY if the db_server parameter is set to "localhost" (default). Note that
equivalents are not supported. The local server will not be installed, for example, if
you specify db_server to be "127.0.0.1", it has to literally be "localhost".

Using name based virtual hosts you can provision multiple Trac environments on the same
system. For example, if both project1.example.com and project2.example.com resolve to
the same system, you could set up two environments as follows:

<pre>
  trac::tracenv{'project1':
    vhost_name => 'project1.example.com',
  }
  
  trac::tracenv{'project2':
    vhost_name => 'project2.example.com',
  }
</pre>

Each virtual host gets its own database. By default the database name will be projectx_db for
PostgreSQL where "projectx" is the project name, but the db_file parameter allows you to select
any valid PostgreSQL database name. The schema name defaults to "public". For sqlite, the database
file name is the project name unless overridden by db_file.

### Trac Authentication

Authentication to the Trac web application is handled through htdigest. The file
the application uses for auth will be located in '/pathToTrac/.htpasswd. In order to
allow users to access your Trac instance, you will first need to add them to the htdigest
file, with a command like the following.

<pre>
htdigest /pathToTrac/.htpasswd projectName user
</pre>

So if your project name is test, and your user name is jon, the command would probably be
the following (assuming you used the default path for your trac instance).

<pre>
htdigest /trac/test/.htpasswd test jon
</pre>

### Git

The Trac module will set up Git repositories to authenticate via SSH.

If using Trac with a Git repository, the default checkout will be:

<pre>
git clone ssh://pathToTrac/repos/projectName
</pre>

For example, if you simply declare a trac instance as follows:

<pre>
  trac::tracenv{'test':
    vhost_name => '*'
    repo_type  => 'git'
  }
</pre>

The command for the svn checkout will be:

<pre>
git clone ssh://pathToTrac/test/repos/test
</pre>

To give system users commit access to the repository, they will need to be added to the
group of the same name as the repository.

### Subversion

The Trac module will set up Subversion repositories to authenticate via SSH.

If using Trac with a Subversion repository, the default checkout will be:

<pre>
svn co svn+ssh://pathToTrac/repos/projectName
</pre>

For example, if you simply declare a trac instance as follows:

<pre>
  trac::tracenv{'test':
    vhost_name => '*',
    repo_type  => 'svn',
  }
</pre>

The command for the svn checkout will be:

<pre>
svn co svn+ssh://trac/test/repos/test
</pre>

To give system users commit access to the repository, they will need to be added to the
group of the same name as the repository.

# Limitations

Currently the corresponding eschiller module is only tested with Ubuntu 12.04, Ubuntu 14.04 and CentOS 6, although 
it's likely that it will function with other variants of Redhat and Debian OS families. 
This module (with the PostgreSQL extensions) has only been tested on CentOS 6.5 so far, but probably
didn't break anything for the other tested systems.

Although telling the module not to create a virtualhost, database, or code repository
is possible via parameters in the trac::tracenv define, doing so is untested and is
likely to fail.

Installing of Trac is generally done via easy_install in this module. Installing Trac via 
package management (apt/yum) is also possible with parameters in trac::tracenv, but is 
untested. 
