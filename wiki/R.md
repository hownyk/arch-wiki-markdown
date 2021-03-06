R
=

  

R is a free software environment for statistical computing and graphics
(http://www.r-project.org/).

Contents
--------

-   1 Installation
    -   1.1 Initial configuration
    -   1.2 Variables
        -   1.2.1 R_ENVIRON
        -   1.2.2 R_PROFILE
-   2 Installing R packages
    -   2.1 Upgrading R packages
-   3 Configuration files
    -   3.1 .Renviron
    -   3.2 Rprofile.r
-   4 Adding a graphical frontend to R
    -   4.1 R Commander frontend
    -   4.2 RKWard frontend
    -   4.3 Rstudio IDE

Installation
------------

Install the r package available in the official repositories

Some external packages may require to be compile in Fortran as well, so
installing the gcc-fortran can be a good idea

> Initial configuration

Please refer to Initialization at Start of an R Session to get a
detailed understanding of startup process. The home directory of the R
installation is usr/bin/R. Base packages are found in
usr/bin/R/library/base and site configuration files in /etc/R/. Aspects
of the Locale are accessed by the functions Sys.getlocale and
Sys.localeconv within the R session. Locales will be the one defined in
your system.

To start a R session, open your terminal and type this command:

    $ R

> Note:

-   Use Shift+u for the command (some terminals use the r letter to
    repeat the last entered command). Once in your R session, the prompt
    will change to >
-   site refers to system-wide in R Documentation

Run help(startup) to read the documentation about system file
configuration, help() for the on-line help,help.start() for the HTML
browser interface to help, demo() for some demos and q() to close the
session and quit.

When closing the session, you will be prompted :
Save workspace Image ?[y/n/c]. The workspace is your current working
environment and include any user-defined objects, functions. The saved
image is stored in .RData format and will be automatically reloaded the
next time R is started. You can manually save the workspace at any time
in the session with the save.image(image.RData) command, save as many
images as you want (eg : image1.RData, image2.RData). You can load image
with the load.iamge(image.RData) command at any time of your session.

> Tip:

-   Tired of R's verbose startup message ? Then start R with the --quiet
    command-line option.

$ R --quiet You can alias R ="R --quiet" in one of your Startup_files

-   Unless explicitly defined somewhere in your configuration files, R
    will start in your $HOME directory. If you want to start in a
    specific directory. first time you create the directory do this:

    $ R
    > setwd("path/to/your/directory")
    > q()
    Save workspace image? [y/n/c]: y

R will create a .RData image file of your current environment. Then,
when double-clicking this file, R will automatically change its working
directory to the file's directory.

> Variables

R can be confusing when it comes to Environment variables, as they are
large and duplicated following the site or user sides. There are two
sorts of files used in startup: environment files, defined by $R_ENVIRON
and profile files, defined by $R_PROFILE.

Most important variables can be found on Environment Variables R
Documentation].

R_ENVIRON

At startup, R search at early stage for site and user .Renviron files to
process for setting Environment variables. The site file is located in
/etc/R, and generated by configure.

The name of the user file can be specified by the R_ENVIRON_USER
Environment variables. If you don't specify any file, R will
automatically read .Renviron in your home directory if there is one. In
case you want to use another emplacement for this file, append this line
 export R_ENVIRON_USER ="path/to/.Renviron" in one of your
Startup_files. This is the place to set all kind of environment
variables using the R syntax.

R_PROFILE

Then R searches for the site-wilde Rprofile.site defined by the
R_PROFILE Environment variables. This file does not exist after a fresh
installation. Finally, R seraches for user R_PROFILE_USER. if unset, a
file called .Rprofile is searched for in the current directory, returned
by the R command > getwd() or in the user's home directory. This is the
place to put all your custom R code.

Installing R packages
---------------------

There are many add-on R packages, which can be browsed on The R
Website.. They can be installed from within R using the
install.packages(pkgname) command. R can install its packages locally as
per user local settings or system wide.

Within your R  session, run this command to check that your user library
exists and is set correctly:

    > Sys.getenv("R_LIBS_USER")

    [1] "/path/to/directory/R/packages"

Installation within your R session is the safest way and won't conflict
with the pacman package management, but there is another method to
install packages. Run the following command in your terminal:

    $ R CMD INSTALL -l $R_LIBS_USER pkg1 pkg2 ...

> Upgrading R packages

    > update.packages(ask=FALSE)

Or when you also need to rebuild packages which were built for an older
version:

    > update.packages(ask=FALSE,checkBuilt=TRUE)

Configuration files
-------------------

The two user configuration files you want in your home folder are
.Renviron and Rprofile.r. If you want to keep your $HOME directory as
clean as possible, a good practice will be to make the ~/.config/r
directory, put the Rprofile.r file at the root of the directory and
append all your R code in this file.

> .Renviron

Lines in Renviron file should be either comment lines starting with # or
lines of the form name=value.Here is a very basic .Renviron you can
start with:

    .Renviron

                                           
    R_HOME_USER = /path/to/your/r/directory
    R_PROFILE_USER = ${HOME}/.config/r/Rprofile.r
    R_LIBS_USER = /path/to/your/r/library
    R_HISTFILE = /path/to/your/filename.Rhistory   # Do not forget to append the .Rhistory

> Rprofile.r

For convenient reasons, you can put a specific Rprofile.r in each of
your usual working directories. One facility would be to dedicate one
directory per project, with its specific profile. When R will change to
the working directory, it will then read the Rprofile.r file inside it.

Here is a very short list of useful options and code:

    Rprofile.r

    options(prompt = paste(paste (Sys.info () [c ("user", "nodename")], collapse = "@"),"[R] "))  # customize your R prompt with username and hostname in this format: user@hostname [R]
    options(digits = 4)                                                                           # number of digits to print
    options(stringsAsFactors = FALSE)
    options(show.signif.stars = FALSE)
    error = quote(dump.frames("${R_HOME_USER}/testdump", TRUE))                                   # post-mortem debugiging facilities

You can add more global options to customize your R environment. See
this post for more user configurations.

Adding a graphical frontend to R
--------------------------------

The linux version of R does not include a graphical user interface.
However, third-party user interfaces for R are available, such as R
commander and RKWard.

> R Commander frontend

R Commander is a popular user interface to R. There is no Arch linux
package available to install R commander, but it is an R package so it
can be installed easily from within R. R Commander requires Tk:

    # pacman -S tk

To install R Commander, run 'R' from the command line. Then type:

    > install.packages("Rcmdr", dependencies=TRUE)

This can take some time.

You can then start R Commander from within R using the library command:

    > library("Rcmdr")

> RKWard frontend

RKWard is an open-source frontend which allows for data import and
browsing as well as running common statistical tests and plots. You can
install rkward from AUR.

> Rstudio IDE

RStudio an open-source R IDE. It includes many modern conveniences such
as parentheses matching, tab-completion, tool-tip help popups, and a
spreadsheet-like data viewer.

Install rstudio-desktop-bin (binary version from the Rstudio project
website) or rstudio-desktop-git (development version) from AUR.

The R library path is often configured with the R_LIBS environment
variable. RStudio ignores this, so the user must set R_LIBS_USER in
~/.Renviron, as documented above.

Retrieved from
"https://wiki.archlinux.org/index.php?title=R&oldid=300122"

Categories:

-   Mathematics and science
-   Programming language

-   This page was last modified on 23 February 2014, at 09:10.
-   Content is available under GNU Free Documentation License 1.3 or
    later unless otherwise noted.
-   Privacy policy
-   About ArchWiki
-   Disclaimers
