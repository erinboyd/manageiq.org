## Install ManageIQ from source

To get started developing ManageIQ, or to deploy on an OS other than CentOS
or Red Hat Enterprise Linux (RHEL), there are a few steps to get started.

A lot of this has been automated in the [ManageIQ kickstart
file](https://github.com/ManageIQ/manageiq/blob/master/build/kickstarts/base.ks.erb).
Details on installing an image using a quickstart file are available from
[the Kickstart page on Fedora](http://fedoraproject.org/wiki/Anaconda/Kickstart).

{:.steps}
1.  For RHEL, make sure you have `rhel-6-server-rpms` channel enabled.

    ```
    rhn-channel --add --channel=rhel-6-server-rpms --user=(your RHN login) --password=(password)
    ```

2.  As root, add the miqbuilder user

    ```bash
    useradd miqbuilder
    passwd miqbuilder
    ```

3.  (Optional) For both RHEL and CentOS, download and install the epel (Extra Packages
    for Enterprise Linux) package

    ```bash
    yum -y install http://dl.fedoraproject.org/pub/epel/6Server/x86_64/epel-release-6-8.noarch.rpm
    ```

4.  `miqbuilder` should be set to use `sudo`. Add the following line to
    /etc/sudoers with the command `visudo`:

    ```bash
    miqbuilder ALL=(ALL) ALL
    ```

5.  ManageIQ prerequisites are git, libxml2 and libxslt (including headers),
    postgresql, and memcached. sudo should be upgraded to the latest version.
    On RHEL and CentOS, install these packages:

    ```bash
    yum -y install git libxml2-devel libxslt libxslt-devel sudo
    yum -y install postgresql-server postgresql-devel memcached ruby-devel gcc-c++
    ```

6.  Start memcached

    ````
    service memcached start
    chkconfig memcached on
    ````

7. Configure Postgres

   1. Initialize the database:

      ```bash
      service postgresql initdb
      ````

   2. Edit `/var/lib/pgsql/data/pg_hba.conf`, comment out everything and add
      the following line:

      ````bash
      local all all trust
      ````

   3. Edit `/var/lib/pgsql/data/postgresql.conf` and add
      `listen_addresses='*'`

   3. Start the database:

      ```bash
      service postgresql start
      ````

   4. Log in as postgres, and generate test, production and development
      databases and the root user.  Then assign the development database
      `vmdb_development` to `root`

      ```bash
      su - postgres
      for i in test production development;do createdb vmdb_$i;done
      psql -c "create role root login password 'smartvm'"
      psql -c "alter database vmdb_development owner to root"
      ```

8.  As user miqbuilder, set up the ManageIQ Ruby development environment.

    ```bash
    su - miqbuilder
    \curl -sSL https://get.rvm.io | bash -s stable
    rvm install 2.0.0
    gem install bundler -v "~>1.3"
    ```

    Note: You can install the development version of rvm by using:
    `\curl -sSL https://get.rvm.io | bash`

    See the [rvm website](http://rvm.io/rvm/install) for more information on installing rvm.

    Note: The bundler version needs to match the [version constraint specified in the lib/Gemfile](https://github.com/ManageIQ/manageiq/blob/43eb6a63e220e0274068549708644edce78485f4/lib/Gemfile#L3)

9.  Clone the ManageIQ repo and install dependencies:

    ```bash
    git clone https://github.com/ManageIQ/manageiq
    cd manageiq/vmdb
    bundle install --without qpid
    cd ..
    vmdb/bin/rake build:shared_objects
    cd vmdb
    bundle install --without qpid
    ```

10. Ensure that ManageIQ is connecting to the right database. Edit the
    `config/database.yml` file.  Add the `root` user for the
    `vmdb_development` database.

    ```
    cp config/database.pg.yml config/database.yml
    bin/rake db:migrate
    bin/rake evm:start
    ```

You should now be able to access the ManageIQ console at **\<IP_ADDRESS\>:3000**. The default username and password is `username : admin` and `password : smartvm`
