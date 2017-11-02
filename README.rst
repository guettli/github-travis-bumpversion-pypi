github-travis-bumpversion-pypi
==============================

This documentation explains how I do CI for small open source python projects.

Four steps:

#. github commit
#. travis CI
#. bumpversion
#. Upload to pypi

The first step gets done by you, the next steps are automated :-)

Terms:

* **Keepass**: This is my tool to store secrets. Of course you can use any other tool

Steps:

* Create github repository. For example: https://github.com/guettli/reprec
* You need a "setup.py". For example: https://github.com/guettli/reprec/blob/master/setup.py
* You need .travis.yml https://github.com/guettli/reprec/blob/master/.travis.yml
* You need a requirements.txt https://github.com/guettli/reprec/blob/master/requirements.txt
* Remember: requirements.txt ist **not** for depedency management. For me it exists to set up a development system in travis.
  Use install_requires in setup.py for depedencies.
* You need a .bumpversion.cfg https://github.com/guettli/reprec/blob/master/.bumpversion.cfg
* Ensure that the version in .bumpversion.cfg and setup.py are equal.
* Create an account in travis for your project. Via web gui: https://travis-ci.org/ "add repo". If you created the github repository just some minutes ago, then the new repo is missing in the list. If you type in the URL to your repo, then it works nevertheless. In this example the URL is https://travis-ci.org/guettli/reprec/
* Now the difficult part: Auth data in an open source project ... where to store the passwords for github commits and pypi uploads?
* During the next step, please pay attention: Do not commit any files, except you know what you do. Plain private keys most not get into the git repo!
* Travis does provide a way to decrypt data ... great. See https://docs.travis-ci.com/user/encrypting-files/#Encrypting-multiple-files
* cd ~/src/reprec/; ssh-keygen -f travis_deploy_key # keep passphrase empty
* save travis_deploy_key and travis_deploy_key.pub in your Keepass. If you want to use this recipie for several repos: The bad news: You need to create and store new deploy keys for every project.
* Create a bot-account for pypi via web GUI: https://pypi.python.org/pypi Create it like a normal user account. Use "Register" at top/right. I use the pattern mylogin-project-bot. In this example guettli-reprec-bot
* Create a file "~/src/reprec/.pypirc-bot" and store username and passwort of pypi bot-account in Keepass::

    [pypi]
    username = mylogin-project-bot
    password = yourpassword
* tar -cf secret-files.tar travis_deploy_key .pypirc-bot
* vi .travis.yml # remove the old "before_install" line " openssl ... -out secret-files.tar -d" (can be on two lines). But leave "- tar xvf secret-files.tar" there.
* travis login --org
* travis  encrypt-file -r guettli/reprec secret-files.tar --add
* The above command changed your .travis.yml file. Changes should be ok. If you removed the old openssl calls everything is fine.
* enter travis_deploy_key.pub to github via github Web-GUI to Settings/Deploy-Keys. Allow write access
* move files which must not get into the git repo: mv .pypirc-bot secret-files.tar travis_deploy_key travis_deploy_key.pub  ~/tmp
* git add secret-files.tar.enc .travis.yml; git commit; git push
* mv ~/.pypirc ~/.pypirc-orig
* cp ~/tmp/.pypirc-bot ~/.pypirc-orig
* The next step is to upload the package to pypi. The first upload is manual, all others are via travis (if all tests are ok). Now the project gets registered at pypi.
* cd src/reprec; python setup.py sdist; twine upload dist/reprec-...tar.gz
* mv ~/.pypirc-orig ~/.pypirc
* Update your README.rst and add the links to the latest travis-results. Copy+Paste from here: https://raw.githubusercontent.com/guettli/reprec/master/README.rst
* Now have fun. All you do is to code, write tests and commit.




