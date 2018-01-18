
# [![**Brooklyn**](https://brooklyn.apache.org/style/img/apache-brooklyn-logo-244px-wide.png)](http://brooklyn.apache.org/)

### Apache Brooklyn helps to model, deploy, and manage systems.

It supports blueprints in YAML or Java, and deploys them to many clouds and other target environments.
It monitors those deployments, maintains a live model, and runs autonomic policies to maintain their health.

For more information visit **[brooklyn.apache.org]( https://brooklyn.apache.org/ )**,
where you'll find:
* **[ Pre-built binaries ]( https://brooklyn.apache.org/download/ )**
* **[ Getting Started instructions ]( https://brooklyn.apache.org/v/latest/start/running.html )**
* **[ A Product Tour ](https://brooklyn.apache.org/learnmore/)**


### Quick Start

This is the uber-repo. To build the entire codebase, 
get this project and its sub-modules:

The Brooklyn documentation uses [Jekyll](https://jekyllrb.com/) to process the site content into HTML. 
This in turn requires Ruby and gems as described in the `Gemfile`:
install [RVM](http://rvm.io/) to manage Ruby installations and sets of Ruby gems.

    \curl -sSL https://get.rvm.io | bash -s stable --auto-dotfiles

Close your shell session and start a new one, to get the new
environment that RVM has configured. Change directory to the location where
this project is (where this file is located).

RVM should detect its configuration inside `Gemfile` and try to configure itself. 
Most likely it will report that the required version of Ruby is not installed,
and it will show the command that you need to run to install the correct version. 
Follow the instructions it shows, typically something like `rvm install ruby-2.1.2`.

Once the correct version of Ruby is installed, change to your home directory
and then change back (`cd ~ ; cd -`).
This will cause RVM to re-load configuration from `Gemfile` with the correct version of Ruby.

Finally, run this command to install all the required Gems 
at the correct versions:

    bundle install

Any time you need to reset your Ruby environment for `jekyll` to run correctly,
go to this directory (or the `_build` subdir) and re-run the above command.

On some platforms there may be some fiddling required before `jekyll` runs without errors,
but the ecosystem is fairly mature and most problems can be resolved with a bit of googling.
Some issues we've encountered are:

 * on Mac, install xcode and its command-line tools
 * if ruby gets confused about versions,
   [clean out your gems](http://judykat.com/ken-judy/force-bundler-rebuild-ruby-rails-gemset/)
 * if `libxml2` fails, set `bundle config build.nokogiri --use-system-libraries` before the install
   (more details [here](http://www.nokogiri.org/tutorials/installing_nokogiri.html))
 * on Ubuntu, `sudo apt-get install libxslt-dev libxml2-dev libcurl4-openssl-dev python-minimal`

If you are building the PDF documentation, this requires [wkhtmltopdf](http://wkhtmltopdf.org/). 
You can download it from [here](http://wkhtmltopdf.org/downloads.html) or use the usual apt-get / yum / brew.

Seeing the Website and Docs
---------------------------

To build and most of see the documentation, run this command in this folder:

    jekyll serve
    
This will start up a local web server. The URL is printed by Jekyll when the server starts,
e.g. http://localhost:4000/ . The server will continue to run until you press Ctrl+C.
Modified files will be detected and regenerated (but that might take up to 1m).
Add `--no-watch` argument to turn off regeneration, or use `jekyll build` instead
to generate a site in `_site` without a server.

This does <i>not</i> generate API docs and certain other material;
see the notes on `_build/build.sh` below for that.


Project Structure
-----------------

Note that there are two interlinked micro-sites in this project:

* `/website`: this contains the main Brooklyn website, including committer instructions,
  download instructions, and "learn more" pages;
  this content has **only one instance** on the live website,
  and as changes are published they replace old content
  
* `/guide`: this contains the user guide and information pertaining to a 
  specific Brooklyn version, including code structure and API documentation;
  the live website contains a **copy of the guide for each Brooklyn version**,
  with the code coming from the corresponding branch in `git`

In addition note the following folders:

* `/style`: contains JS, CSS, and image resources;
  on the live website, this folder is installed at the root *and* 
  into archived versions of the guide. 
  
* `/_build`: contains build scripts and configuration files,
  and tests for some of the plugins

* `/_plugins`: contains Jekyll plugins which supply tags and generation
  logic for the sites, including links and tables of contents

* `/_layouts`: contains HTML templates used by pages

* `/_includes`: contains miscellaneous content used by templates and pages

Jekyll automatically excludes any file or folder beginning with `_`
from direct processing, so these do *not* show up in the `_site` folder
(except where they are embedded in other files).  

**A word on branches:**  The `/website` folder can be built against any branch;
typically changes are made and published from `master`, to ensure that all versions
are listed correctly.
In contrast the `/guide` folder should be updated and built against the branch for which 
instructions are being made, e.g. `master` for latest snapshot updates, 
or `0.7.0-M2` for that milestone release.
It *is* permitted to make changes to docs (and docs only!) after a release has
been made. In most cases, these changes should also be made to master.


Website Structure
-----------------

The two micro-sites above are installed on the live website as follows:

* `/`: contains the website
* `/v/<version>`: contains specific versions of the guide,
  with the special folder `/v/latest` containing the recent preferred stable/milestone version 

The site itself is hosted at `brooklyn.apache.org` with a `CNAME`
record from `brooklyn.io`.

Content is published to the site by updating an 
Apache subversion repository, `brooklyn-site-public` at
`https://svn.apache.org/repos/asf/brooklyn/site`.
See below for more information.


Building the Website and Guide
------------------------------

For most users, the `jekyll serve` command described above is sufficient to test changes locally.
The main reason to use the build scripts (and to read this section) is to push changes to the server
(requires Apache Brooklyn commit rights), or to test generated content such as API docs.

The build is controlled by config files in `_build/` and accessed through `_build/build.sh`.
There are a number of different builds possible; to list these, run:

    _build/build.sh help

The normal build outputs to `_site/`.  The three builds which are most relevant to updating the live site are:

* **website-root**: to build the website only, in the root
* **guide-latest**: to build the guide only, in `/v/latest/`
* **guide-version**: to build the guide only, in the versioned namespace e.g. `/v/<version>/`

There are some others, including `test-both`, which apply slightly different configurations
useful for testing.
Supported options beyond that include `--serve`, to start a web browser serving the content of `_site/`,
and `--skip-javadoc`, to speed up the build significantly by skipping javadoc generation.
A handy command for testing the live files, analogous to `jekyll serve` 
but with the correct file structure, and then checking links, is:

    _build/build.sh test-both --skip-javadoc --serve

And to run link-checks quickly (without validating external links), use:

    htmlproof --href_ignore "https?://127.*" --alt_ignore ".*" --disable_external _site



Preparing for a Release
-----------------------

When doing a release and changing versions:

* Before branching:
  * Change the `brooklyn-stable-version` variable in `_config.yml`
  * Update `website/meta/versions.md` with a bit of info on this release
*  In the branch, with `change-version.sh` run (e.g. from `N.SNAPSHOT` to `N`)
  * Ensure the `guide/start/release-notes.md` file is current
  * Build and publish `website-root`, `guide-latest`, and `guide-version`
* In master, with `change-version.sh` run (e.g. to `N+1-SNAPSHOT`)
  * Clear old stuff in the `guide/start/release-notes.md` file
  * Optionally build and public `guide-version`
 

Publishing the Website and Guide
--------------------------------

The Apache website publication process is based around the Subversion repository; 
the generated HTML files must be checked in to Subversion, whereupon an automated process 
will publish the files to the live website.
So, to push changes the live site, you will need to have the website directory checked out 
from the Apache subversion repository. We recommend setting this up as a sibling to your
`brooklyn` git project directory:

    # verify we're in the right location and the site does not already exist
    ls _build/build.sh || { echo "ERROR: you should be in the docs/ directory to run this command" ; exit 1 ; }
    ls ../../brooklyn-site-public > /dev/null && { echo "ERROR: brooklyn-site-public dir already exists" ; exit 1 ; }
    pushd `pwd -P`/../..
    
    svn --non-interactive --trust-server-cert co https://svn.apache.org/repos/asf/brooklyn/site brooklyn-site-public
    
    # verify it
    cd brooklyn-site-public
    ls style/img/apache-brooklyn-logo-244px-wide.png || { echo "ERROR: checkout is wrong" ; exit 1 ; }
    export BROOKLYN_SITE_DIR=`pwd`
    popd
    echo "SUCCESS: checked out site in $BROOKLYN_SITE_DIR"

With this checked out, the `build.sh` script can automatically copy generated files into the right subversion sub-directories
with the `--install` option.  (This assumes the relative structure described above; if you have a different
structure, set BROOKLYN_SITE_DIR to point to the directory as above.  Alternatively you can copy files manually,
using the instructions in `build.sh` as a guide.)

A typical update consists of the following commands (or a subset),
copied to `${BROOKLYN_SITE_DIR-../../brooklyn-site-public}`:

    # ensure svn repo is up-to-date (very painful otherwise)
    cd ${BROOKLYN_SITE_DIR-../../brooklyn-site-public}
    svn up
    cd -

    # versioned guide, safe for snapshots, relative to /v/<version>/
    _build/build.sh guide-version --install

    # main website, if desired, relative to / 
    _build/build.sh website-root --install
    
And then, with jdk 1.8+ and maven 3.1+ installed:

    mvn clean install -Dno-go-client -Dno-rpm

The results are in `brooklyn-dist/dist/target/`, including a tar and a zip.
Or to run straight after the build, do:

    pushd brooklyn-dist/dist/target/brooklyn-dist/brooklyn/
    bin/brooklyn launch


### Resources

<!--- BROOKLYN_VERSION_BELOW -->
The **[Developers](https://brooklyn.apache.org/developers/)** section of the main website contains more detail on working with the codebase. There is also a more **Developer Guide** specific to each version, including [this branch (0.12.0)](https://brooklyn.apache.org/v/0.12.0/dev/), [latest stable](https://brooklyn.apache.org/v/latest/dev/), and [older releases](https://brooklyn.apache.org/meta/versions.html).

Useful topics include:

* getting the **[source code](https://brooklyn.apache.org/developers/code/)**

* **[setting up Git](https://brooklyn.apache.org/developers/code/git-more.html)** with forks, submodules (or alternatively [avoiding submodules](https://brooklyn.apache.org/developers/code/git-more.html#not-using-submodules)) and other productivity hints

* the **[maven build](http://brooklyn.apache.org/v/latest/dev/env/maven-build.html)** and what to do on build errors

<!--- BROOKLYN_VERSION_BELOW -->
* **[project structure](https://brooklyn.apache.org/v/0.12.0/dev/code/structure.html)** of the codebase and submodules

* the **[people](https://brooklyn.apache.org/community/)** behind Apache Brooklyn
