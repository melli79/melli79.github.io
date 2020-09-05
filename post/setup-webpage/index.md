There is a considerable easy way to set up a web page similar to this one.

# Step 1: install Hugo
A. If you are using a Mac, just get accustomed to [Homebrew](https://brew.sh/) and install hugo via
```sh
  brew install hugo
```

B. If you are using Linux it probably comes with a package manager that should have some compatible version of hugo available, e.g. on Ubuntu like systems:
```sh
  sudo apt-get install hugo
```

If you feel that your system is too old consider updating it first.  If that is unavailable, you can try to install a second package manager, e.g. [LinuxBrew](https://docs.brew.sh/Homebrew-on-Linux) from which you can install the latest version of Hugo as under MacOS.

C. If you are stuck with Windows, consider converting your computer to Linux.  Most decent versions of Linux run much smoother than Windows, plus the user 2 user support base is much bigger.  If there is no way to get hold of Linux, then you can pimp up your Windows either with [WSL](https://www.windowscentral.com/install-windows-subsystem-linux-windows-10) or with a package manager such as [Chocolatey](https://chocolatey.org/) or [Scoop](https://scoop.sh/) from which you can then install
```ps1
  choco install hugo -confirm
```
or
```ps1
  scoop install hugo
```

Once you have set up Hugo, test it via
```sh
  hugo version
```
which should report the installed version together with the system information on which it is running.  As of writing this summary, the current Hugo version is 0.74.3, but yours could already be much higher.


# Step 2: Set up a new site structure
In your shell change to the directory that should host the Page source directory, e.g. "~/Documents" and enter
```sh
  hugo new site <my-awesome-homepage>
```
where `<my-awesome-homepage>` is the name of your home page collection.

Before you start configuring the central "config.toml", download the theme you like, because most of these come with quite some defaults for the "config.toml" to have a faster ramp up.  You can find a large collection of recent themes at [Hugo-Themes](https://themes.gohugo.io/), once you have decided for a theme go to its git repo and download the current content from there, either as "*.zip" archive or via `git clone "http://..." themes/<name>` where `<name>` is the name of the theme.
If you have cloned the theme, please remove its ".git*", because you are likely not uploading your personalization to the public git repository.

If you are comfortable with git, you can now put the root directory of your home page sources under git control, e.g. `git init`.  You may also want to start with some ".gitignore" such as
```gitignore
  public/
  resources/_gen/
```

You can now start copying the proposed "config.toml" from the "theme/<name>" directory to the root directory of your home page source.  Once you have done that, you should adjust it, e.g. enter the name of your home page and maybe your eMail, github name, ... .

# Step 3: Show the current state of you Home page
You can always show the current state of your home page by running
```sh
  hugo server -D
```
in the root directory of your home page.  This will start a local http server together with a compiled version of your current sources.  The server is reachable locally under some address such as "http://localhost:1313/" (watch the startup message) and will update automatically if you modify the page sources.  Internal links, content and external links should be working, but integrated resources such as a visit counter may not work out of the temporary version.

# Step 4: Adding new Content
To add a page, you just call
```sh
  hugo new <type>/<title>.md
```
where type is some type such as "posts" and title the title of the (file containing the) page, e.g. "todays results".  It does not have to coincide with the title of the page and can be changed afterwards.

Hugo will then generate a minimal empty page (in Hugo-markdown) that contains some meta-information.  You can edit this file with any UTF-8 editor of your liking such as vim, atom or others.

# Step 5: Building a publishable version of your Home Page
```sh
  hugo -D
```
in the root directory.  This will compile the whole page into the subdirectory "public" which you should not check into the source git repository.  Instead you can upload it to the wwwroot of your web host, e.g. a repository on github.com to have your homepage hosted on "https://<mynic>.github.io/".  Remember that people will see content you put up there as soon as webcrawlers like Google find it.  Please respect the copyrights of other people's work.
