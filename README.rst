github -> travis -> bumpversion -> pypi
=======================================

This guide is outdated. Github Actions are a better solution.

See: https://docs.github.com/en/actions/guides/building-and-testing-python#publishing-to-package-registries

---

This documentation explains how I do CI for small open source python projects.

Four steps:

#. github commit
#. travis CI
#. bumpversion
#. Upload to pypi

The first step gets done by you, the next steps are automated :-)

Terms:

* Keepass: This is my tool to store secrets. Of course you can use any other tool
* reprec: This is the name of my example git repo. Replace this with the name of your git repo.

Steps:

* Create github repository. For example: https://github.com/guettli/reprec
* You need a "setup.py". For example: https://github.com/guettli/reprec/blob/master/setup.py
* You need .travis.yml https://github.com/guettli/reprec/blob/master/.travis.yml
* You need a requirements.txt https://github.com/guettli/reprec/blob/master/requirements.txt
* Remember: requirements.txt is **not** for depedency management. For me it exists to set up a development system in travis.
  Use install_requires in setup.py for depedencies.
* You need a .bumpversion.cfg https://github.com/guettli/reprec/blob/master/.bumpversion.cfg
* Ensure that the version in .bumpversion.cfg and setup.py are equal.
* Create an account in travis for your project. Via web gui: https://travis-ci.org/ "add repo". If you created the github repository just some minutes ago, then the new repo is missing in the list. It can take several minutes until the repo is visible in travis. If you type in the URL to your repo, then it works nevertheless. In this example the URL is https://travis-ci.org/guettli/reprec/
* Now the difficult part: Auth data in an open source project ... where to store the passwords for github commits and pypi uploads?
* During the next step, please pay attention: Do not commit any files, except you know what you do. Plain private keys must not get into the git repo!
* Travis does provide a way to decrypt data ... great. See https://docs.travis-ci.com/user/encrypting-files/#Encrypting-multiple-files (but if you read the following instruction, you don't need to read the travis docs).
* cd ~/src/reprec/; ssh-keygen -f travis_deploy_key # keep passphrase empty
* save travis_deploy_key and travis_deploy_key.pub in your Keepass. If you want to use this recipie for several repos: The bad news: You need to create and store new deploy keys for every project::

    cat travis_deploy_key
    cat travis_deploy_key.pub
* Create a bot-account for pypi via web GUI: https://pypi.python.org/pypi Create it like a normal user account. Use "Register" at top/right. I use the pattern mylogin-project-bot. In this example guettli-reprec-bot. Verify the mail address of the bot (Pypi will send you a registration mail, after you created the new pypi account)
* Create a file "~/src/reprec/.pypirc-bot" and store username and passwort of pypi bot-account in Keepass::

    [pypi]
    username = mylogin-project-bot
    password = yourpassword
* tar -cf secret-files.tar travis_deploy_key .pypirc-bot
* vi .travis.yml # The file from reprec does contain a line which you need to remove. The line is after "before_install" and looks like this " openssl ... -out secret-files.tar -d" (can be on two lines)). But leave "- tar xvf secret-files.tar" there.
* travis login --org
* travis  encrypt-file -r yourgithublogin/reprec secret-files.tar --add
* The above command changed your .travis.yml file. Changes should be ok. If you removed the old openssl calls everything is fine.
* enter travis_deploy_key.pub to github via github Web-GUI to Settings/Deploy-Keys. Name is "travis_deploy_key.pub". Content is the file content. AND Allow write access. 
* move files which must not get into the git repo: mv secret-files.tar travis_deploy_key travis_deploy_key.pub ~/tmp
* .pypirc-bot is still around. We move this file (containing unecrypted secrets) away later.
* git add secret-files.tar.enc .travis.yml; git commit; git push
* The next step is to upload the package to pypi. The first upload is manual, all others are via travis (if all tests are ok). Now the project gets registered at pypi.
* Before uploading you need to add mylogin-project-bot to the project as maintainer via pypi web gui.
* Install `twine` (via pip)
* python setup.py sdist
* twine upload --config-file .pypirc-bot dist/...tar.gz
* Verify that above line is in your .travis.yml file.
* Move away the file containing because it contains unecrypted secrets: mv .pypirc-bot ~/tmp/
* Update your README.rst and add the links to the latest travis-results. Copy+Paste from here: https://raw.githubusercontent.com/guettli/reprec/master/README.rst
* commit and push. Check the travis web site of your repo. 
* Now have fun. All you do is to code, write tests and commit. The next steps (execute tests, bumpversion, upload new release) are automated :-)
* I hope above instructions helped you. Please tell me if you find errors or things to improvemnt.




