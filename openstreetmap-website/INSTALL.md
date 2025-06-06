# Installation

These instructions are designed for setting up `openstreetmap-website` for development and testing.
If you want to deploy the software for your own project, then see the notes at the end.

You can install the software directly on your machine, which is the traditional and probably best-supported approach. However, there
are two alternatives which make it easier to get a consistent development environment and may avoid installation difficulties:

* **Vagrant** This installs the software into a virtual machine. For Vagrant instructions see [VAGRANT.md](VAGRANT.md).
* **Docker** This installs the software using containerization. For Docker instructions see [DOCKER.md](DOCKER.md).

These instructions are based on Ubuntu 22.04 LTS, which is the platform used by the OSMF servers.
The instructions also work, with only minor amendments, for all other current Ubuntu releases, Fedora and MacOSX

We don't recommend attempting to develop or deploy this software on Windows. Some Ruby gems may not be supported. If you need to use Windows the easiest solutions in order are [Docker](DOCKER.md), [Vagrant](VAGRANT.md), and Ubuntu in a virtual machine.

## Dependencies

Many of the dependencies are managed through the standard Ruby on Rails mechanisms -
i.e. ruby gems specified in the Gemfile and installed using bundler. However, there are a large number
of packages required before you can get the various gems installed.

## Minimum requirements

* Ruby 3.0+
* PostgreSQL 9.1+
* Bundler (see note below about [developer Ruby setup](#rbenv))
* Javascript Runtime

These can be installed on Ubuntu 22.04 or later with:

```
sudo apt-get update
sudo apt-get install ruby3.0 libruby3.0 ruby3.0-dev \
                     libvips-dev libxml2-dev libxslt1-dev nodejs \
                     build-essential git-core firefox-geckodriver \
                     postgresql postgresql-contrib libpq-dev libsasl2-dev \
                     libffi-dev libgd-dev libarchive-dev libbz2-dev yarnpkg
sudo gem3.0 install bundler
```

### Alternative platforms

#### Fedora

For Fedora, you can install the minimum requirements with:

```
sudo dnf install ruby ruby-devel rubygem-rdoc rubygem-bundler rubygems \
                 libxml2-devel nodejs \
                 gcc gcc-c++ git \
                 postgresql postgresql-server postgresql-contrib libpq-devel \
                 perl-podlators libffi-devel gd-devel libarchive-devel \
                 bzip2-devel nodejs-yarn vips-devel
```

If you didn't already have PostgreSQL installed then create a PostgreSQL instance and start the server:

```
sudo postgresql-setup initdb
sudo systemctl start postgresql.service
```

Optionally set PostgreSQL to start on boot:

```
sudo systemctl enable postgresql.service
```

#### MacOSX

For MacOSX, you will need XCode installed from the Mac App Store; OS X 10.7 (Lion) or later; and some familiarity with Unix development via the Terminal.

Installing PostgreSQL:

* Install Postgres.app from https://postgresapp.com/
* Make sure that you've initialized and started Postgresql from the app (there should be a little elephant icon in your systray).
* Add PostgreSQL to your path, by editing your profile:

`nano ~/.profile`

and adding:

`export PATH=/Applications/Postgres.app/Contents/MacOS/bin:$PATH`

After this, you may need to start a new shell window, or source the profile again by running `. ~/.profile`.

Installing other dependencies:

* Install Homebrew from https://brew.sh/
* Install the latest version of Ruby: `brew install ruby`
* Install other dependencies: `brew install libxml2 gd yarn pngcrush optipng pngquant jhead jpegoptim gifsicle svgo advancecomp vips`
* Install Bundler: `gem install bundler` (you might need to `sudo gem install bundler` if you get an error about permissions - or see note below about [developer Ruby setup](#rbenv))

You will need to tell `bundler` that `libxml2` is installed in a Homebrew location. If it uses the system-installed one then you will get errors installing the `libxml-ruby` gem later on<a name="macosx-bundle-config"></a>.

```
bundle config build.libxml-ruby --with-xml2-config=/usr/local/opt/libxml2/bin/xml2-config
```

If you want to run the tests, you need `geckodriver` as well:

```
brew install geckodriver
```

Note that OS X does not have a /home directory by default, so if you are using the GPX functions, you will need to change the directories specified in config/application.yml.

## Cloning the repository

The repository is reasonably large (~150MB) and it's unlikely that you need the full history. If you are happy to wait for it all to download, run:

```
git clone https://github.com/openstreetmap/openstreetmap-website.git
```

To clone only the most recent version (~23MB), instead use a 'shallow clone':

```
git clone --depth=1 https://github.com/openstreetmap/openstreetmap-website.git
```

If you want to add in the full history later on, perhaps to run `git blame` or `git log`, run `git fetch --depth=1000000`


## Ruby gems

We use [Bundler](http://gembundler.com/) to manage the rubygems required for the project.

```
cd openstreetmap-website
bundle install
```

## Node.js modules

We use [Yarn](https://yarnpkg.com/) to manage the Node.js modules required for the project.

```
bundle exec bin/yarn install
```

## Prepare local settings file

This is a workaround. [See issues/2185 for details](https://github.com/openstreetmap/openstreetmap-website/issues/2185#issuecomment-508676026).

```
touch config/settings.local.yml
```

## Storage setup

`openstreetmap-website` needs to be configured with an object storage facility - for
development and testing purposes you can use the example configuration:

```
cp config/example.storage.yml config/storage.yml
```

## Database setup

`openstreetmap-website` uses three databases -  one for development, one for testing, and one for production. The database-specific configuration
options are stored in `config/database.yml`, which we need to create from the example template.

```
cp config/example.database.yml config/database.yml
```

PostgreSQL is configured to, by default, accept local connections without requiring a username or password. This is fine for development.
If you wish to set up your database differently, then you should change the values found in the `config/database.yml` file, and amend the
instructions below as appropriate.

### PostgreSQL account setup

We need to create a PostgreSQL role (i.e. user account) for your current user, and it needs to be a superuser so that we can create more databases.

```
sudo -u postgres -i
createuser -s <username>
exit
```

### Create the databases

To create the three databases - for development, testing and production - run:

```
bundle exec rails db:create
```

### Database structure

To create all the tables, indexes and constraints, run:

```
bundle exec rails db:migrate
```

## Running the tests

To ensure that everything is set up properly, you should now run:

```
bundle exec rails test:all
```

This test will take a few minutes, reporting tests run, assertions, and any errors. If you receive no errors, then your installation is successful.

The unit tests may output parser errors related to "Attribute lat redefined." These can be ignored.

### Starting the server

Rails comes with a built-in webserver, so that you can test on your own machine without needing a server. Run

```
bundle exec rails server
```

You can now view the site in your favourite web-browser at [http://localhost:3000/](http://localhost:3000/)

Note that the OSM map tiles you see aren't created from your local database - they are just the standard map tiles.

# Configuration

After installing this software, you may need to carry out some [configuration steps](CONFIGURE.md), depending on your tasks.

# Ruby development install and versions<a name="rbenv"></a> (optional)

For simplicity, this document explains how to install all the website dependencies as "system" dependencies. While this is simpler, and usually faster, you might want more control over the process or the ability to install multiple different versions of software alongside eachother. For many developers, [`rbenv`](https://github.com/rbenv/rbenv) is the easiest way to manage multiple different Ruby versions on the same computer - with the added advantage that the installs are all in your home directory, so you don't need administrator permissions.

If you choose to install Ruby and Bundler via `rbenv`, then you do not need to install the system libraries for Ruby:

* For Ubuntu, you do not need to install the following packages: `ruby3.0 libruby3.0 ruby3.0-dev bundler`,
* For Fedora, you do not need to install the following packages: `ruby ruby-devel rubygem-rdoc rubygem-bundler rubygems`
* For MacOSX, you do not need to `brew install ruby` - but make sure you've installed a version of Ruby using `rbenv` before running `gem install bundler`!

After installing a version of Ruby with `rbenv` (the latest stable version is a good place to start), you will need to make that the default. From inside the `openstreetmap-website` directory, run:

```
rbenv local $VERSION
```

Where `$VERSION` is the version you installed. Then install bundler:

```
gem install bundler
```

You should now be able to proceed with the rest of the installation. If you're on MacOSX, make sure you set up the [config override for the libxml2 location](#macosx-bundle-config) _after_ installing bundler.
