## INSTALL
This document outlines how to install JSS Resource Tools on your platform of choice.

###Requirements
JSS Resource Tools is written in Python and has a few module dependencies. Specificially it requires:

* Python 2.7 or greater
* `click` Module
* `requests` Module 

###Easy Install with `pip`

JSS Resource Tools is listed in [Python Package Index](https://pypi.python.org/pypi/jss-resource-tools/) and can easily be installed and managed by `pip`.

####macOS

Due to some issues with the built in version of OpenSSL shipped with macOS, JSS Resource Tools won't work out of the box with the macOS python install. This is easily fixed with a proper Python install through Homebrew:

1. If you haven't already: install [Homebrew](https://brew.sh/).
2. Install python using Homebrew: `brew install python`
3. Install required modules using pip2: `pip2 install jss-resource-tools click requests`
4. You should now be able to run the tool by running `jss_resource_tools`

####Linux

Installing on most modern Linux variants is pretty simple as most of the time, Python > 2.7 is already installed. If `pip` is not already installed, you many need to install it by running `apt-get install python-pip` or something simular depending on your distribution. Once Python and Pip are sorted, it should be as simple as running:

`python -m pip install jss-resource-tools click requests`

after which, eou should now be able to run the tool by running `jss_resource_tools`
