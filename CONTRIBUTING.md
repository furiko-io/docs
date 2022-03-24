# Contribution Guide

Thanks for wanting to contribute to Furiko's documentation! Here are some tips and guidelines on how to start contributing.

## Navigating the Project

The documentation site is built using [MkDocs](https://www.mkdocs.org/), a static site generator. As such, the project is structured like how a typical MkDocs project is structured.

### Project Structure

- `docs`: The root of the doc site at <https://furiko.io>.
  - Any `.md` files in this folder will be built into `.html` files by `mkdocs`.
  - The names of `.md` files and any folders will be used in the resultant public URL path, so please choose names wisely.
  - Additional non-documentation files (e.g. image assets, CSS) can be added to this folder as well.
  - **IMPORTANT**: Try not to rename existing files or folders, as it may break existing links to the site.
- `src`: Contains any source code used for MkDocs code generation.
- `mkdocs.yml`: The [configuration file](https://www.mkdocs.org/user-guide/configuration/) used by MkDocs.

## External References

The following is a list of useful references when working with this repository.

- [MkDocs Documentation](https://www.mkdocs.org/user-guide/): Static site generator options.
- [Material for MkDocs Reference](https://squidfunk.github.io/mkdocs-material/reference/): Configuration of theme, components.
- [Python Markdown Reference](https://squidfunk.github.io/mkdocs-material/setup/extensions/python-markdown/): We also use extra Markdown syntax adopted from Material for MkDocs.

## Running Locally

1. MkDocs uses Python internally, so you will first need a working Python environment.
   - **Tip**: We recommend using `virtualenv` if possible. This [guide](https://pythonbasics.org/virtualenv/) might be a good starting point.
2. Inside the virtual environment, tnstall requirements from `requirements.txt`. This installs `mkdocs` as well as any additional extensions needed:

   ```sh
   pip install -r requirements.txt
   ```

3. To serve the site locally, run the following, which starts serving the site at <http://localhost:8000>:

   ```sh
   mkdocs serve
   ```

## Development Environment

- **IDE**: We recommend using VS Code for editing this repository, which bundles several useful tools used in the repository.
- **Formatting**: This repository uses [Prettier](https://prettier.io/) to format almost all code.
  - One exception is `.html` files which includes Jinja2 syntax, which gets messed up when running Prettier on the file.

## Testing Your Changes

Currently there is no testing infra for testing doc site changes, and all changes must be verified locally. We welcome ideas and contributions to this!

## Submitting Your Changes

1. Fork this repository.
2. Push your changes to a new branch.
3. Submit a PR to the `main` branch.
4. Once merged, the changes will be deployed via GitHub Actions to [the website](https://furiko.io).
