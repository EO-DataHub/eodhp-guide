# Build and Publish a Documentation Site with Material for MkDocs

This document explains everything that I did to set up a Python environment, install and use the Material theme for MkDocs, and publish the wireframe documentation website using GitHub. The same setup can be used for future maintenance and development. It assumes basic familiarity with the command line and GitHub.

## Setting up the Python environment

MkDocs is a Python-based tool, so you will need a working Python environment. Two common approaches are described below. I am a fan of pixi and used that method, with the Positron IDE, but choose the method that best fits your workflow.

### Option A: Standard Python environment

1. Install Python (version 3.11 or later is recommended).
   - On Windows or macOS, download Python from the official Python website.
   - On Linux, use your system package manager where possible.
2. Verify the installation by opening a terminal and running:
   ```
   python --version
   ```
   or
   ```
   python3 --version
   ```
3. Create a virtual environment for your project:
   ```
   python -m venv venv
   ```
4. Activate the virtual environment:
   - **Windows:**
     ```
     venv\Scripts\activate
     ```
   - **macOS or Linux:**
     ```
     source venv/bin/activate
     ```
5. Upgrade pip to ensure compatibility:
   ```
   python -m pip install --upgrade pip
   ```

More information is available in the [Python docs](https://docs.python.org/3/tutorial/venv.html).

### Option B: Installing and using pixi

pixi is a modern environment and package manager that can simplify dependency management, especially for reproducible builds.

1. Install pixi by following the instructions on the [pixi website](https://pixi.sh/) for your operating system.
2. Create a new pixi project:
   ```
   pixi init mkdocs-site
   ```
3. Change into the project directory:
   ```
   cd mkdocs-site
   ```
4. Add Python to the environment if it is not already included:
   ```
   pixi add "python==3.11"
   ```
5. Activate the pixi environment:
   ```
   pixi shell
   ```

pixi will manage Python and all dependencies for the project, reducing the need for manual virtual environment handling.

## Installing Material for MkDocs

Once your Python environment is ready, you can install MkDocs and the Material theme.

1. Install MkDocs and the Material theme and support for rendering Jupyter notebooks:
   ```
   pip install mkdocs-material mkdocs-jupyter
   ```
   If you are using Pixi, either run the same command inside the Pixi shell or use the add command:
   ```
   pixi add mkdocs-material mkdocs-jupyter
   ```
2. Create a new MkDocs project:
   ```
   mkdocs new my-site
   ```
3. Move into the project directory:
   ```
   cd my-site
   ```
4. Open the `mkdocs.yml` configuration file and set the theme:
   ```yaml
   theme: 
     name: material
   ```
5. Start the local development server:
   ```
   mkdocs serve
   ```
6. Open a web browser and navigate to the local address shown in the terminal to preview your site.

## Where to find help for Material for MkDocs

The following resources are recommended:

- The [official Material for MkDocs documentation](https://squidfunk.github.io/mkdocs-material/), which covers configuration, features, and customisation in detail.
- I used [this tutorial](https://squidfunk.github.io/mkdocs-material/getting-started/) to get me up and running - it's well worth understanding.

## Putting the website onto GitHub

GitHub Pages is a simple way to host the static site you produce. The following is what I did with the eodh-userdocs repository so it would only need to be done again if a new repository was set up.

1. Create a new repository on GitHub.
2. Initialise Git in your local project directory if it is not already set up:
   ```
   git init
   ```
3. Add your files and make an initial commit:
   ```
   git add .
   git commit -m "Initial MkDocs site"
   ```
4. Link your local repository to the GitHub repository:
   ```
   git remote add origin https://github.com/your-username/your-repository.git
   ```
5. Build the site:
   ```
   mkdocs build
   ```
6. Deploy the site to GitHub Pages:
   ```
   mkdocs gh-deploy
   ```

This command builds the site in `/docs` and publishes it to the `gh-pages` branch, which GitHub Pages can serve automatically. I also set up a GitHub Action in line with what is mentioned in the tutorial.

## Updating the site using GitHub

To update your site after the initial deployment:

1. Edit or add Markdown files in the `docs` directory.
2. Preview changes locally:
   ```
   mkdocs serve
   ```
3. Commit your changes:
   ```
   git add .
   git commit -m "Update documentation"
   ```
4. Push the changes to GitHub:
   ```
   git push origin main
   ```
5. Redeploy the site:
   ```
   mkdocs gh-deploy
   ```

After deployment, GitHub Pages will update the live site automatically. If you get stuck at this stage there are loads of folks on the EODH project who can help out with best practice and hints/tips for working with GitHub.

## Notes

The following are recommended as good next steps (based on the fact that this documentation site is a wireframe):

- Choose the pages that are required and rework the file structure/hierarchy
- Check existing URLS
- Add and update URLs
- Check and update content
- Add new content where needed
- Copy across icon/emoticons to maintain styling with Wagtail site
- Add/update any science examples
