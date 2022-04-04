This is a documentation setup for the ASCEDS project. It uses sphinx (a dcoumentationi library based on python) to build html docs from .rst (restructured text) files. We use a free service https://www.readthedocs.io to make the docs publicly accessible.



Local build instructions
========================

Setup
-----


1. Install python3, pip
2. Clone repo and run 

        pip install -r requirements.txt


Build
-----
This should output the built html pages. Open build/html in your file explorer and open index.html with your browser. (You may see some errors/warning, they can be ignored.)

    cd docs
    make html



Remote setup instructions
=========================
Follow these instructions to setup the docs website.

1. Sign in to readthedocs.io using your github oauth.
2. Click "Import a project", select the github repo containgin the documentation code.
3. Follow on screen setup instructions

Remote build instructions
=========================
Follow these instructions to update the docs website.

1. Push doc changes to docs repo
2. Login to readthedocs.io
3. Open the project, click "Build version"

