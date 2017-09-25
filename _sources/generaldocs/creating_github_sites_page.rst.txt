############################################
Creating your own github site using sphinx
############################################


Set up a python virtualenv and install software
***********************************************

.. code-block:: sh

  cd ~/
  apt-get install python-virtualenv python-dev build-essential
  virtualenv merlintechs-source-virtualenv
  . merlintechs-source-virtulenv
  pip install python-sphinx


Pull down your personal github pages repo
*****************************************

Go into github and create a new repository called <username>.github.io

.. code-block:: sh

  cd ~/
  git clone git@github.com:shannonmitchell/shannonmitchell.github.io
  cd shannonmitchell.github.io
  git config --global user.name shannonmitchell
  git config --global user.email shannon.mitchell@merlintechs.com


Create your source repo
***********************

I first went into github and created a new repo called merlintechs-source.

.. code-block:: sh

  cd ~/
  mkdir merlintechs-source
  cd merlintechs-source  
  git init
  vi README.rst
     (place your own docs in here...)
  git add README.rst
  mkdir docs
  cd docs
  
  sphinx-quickstart
    (kept defaults except for the following)
    (Project Name: Merlintechs)
    (Author Name: Shannon Mitchell)
    (create .nojeckll file to publish the document on github pages(y))

  git config --global user.name shannonmitchell
  git config --global user.email shannon.mitchell@merlintechs.com
  git commit -m 'Creating initial sphinx managed source repo'
  git remote add origin git@github.com:shannonmitchell/merlintechs-source.git
  git push origin master 

  vi Makefile

.. code-block:: sh

  ...
  BUILDDIR = /path/to/local/repo/shannonmitchell.github.io/
  PUBLISH_REPO = ../../shannonmitchell.github.io/
  ...
  allthethings: html
          mv $(BUILDDIR)/html/* $(PUBLISH_REPO)/; cd $(PUBLISH_REPO); git add .; git commit -m "rebuild docs"; git push origin master


.. code-block:: sh

  make allthethings

  git add Makefile
  git commit -m "save the things"
  git push origin master


View it in your browser
***********************

It make take a few seconds to update. When it did, I could access the site at the following locaiton.

http://shannonmitchell.github.io




