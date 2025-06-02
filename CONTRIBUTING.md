# Contributing Documentation

## Pre-requisites

To run this repository locally you will need the following tools.

* Docker

## Development lifecycle

* Clone the repository locally
* Checkout a new branch
* Make changes within the source directory
* Test changes locally to running the relevant commands in the Development section
* Commit changes
* Push to GitHub
* Create Pull Request
* Review generated site that is linked in the Pull Request
* Obtain approval from the IDAM Team
* Merge Pull Request into `main` branch

### Development

#### Editing

The documentation is created by editing `*.html.md.erb` files, found in the [source](source) folder.

The syntax is Markdown, more details can be found [here](https://daringfireball.net/projects/markdown/).

For guidance see the Tech Docs Template [Write your content](https://tdt-documentation.london.cloudapps.digital/write_docs/content/).

The Markdown syntax may use [kramdown](https://kramdown.gettalong.org/syntax.html) TBC.

While editing the files locally, you can start a Docker container that will use Middleman to act as a server hosting the webpages. See [preview docs](#preview-docs).

Every change should be reviewed in a pull request, no matter how minor. PR request reviewer/s should be enabled within the main branch protection settings.

Merging the changes to the `main` branch automatically publishes the changes via GH Action. See [publishing](#publishing).

#### Preview docs

You can preview how your changes will look, if you've cloned this repo to your local machine, and run this command:

```
make preview
```

This will run a preview web server on http://localhost:4567 which you can open in your browser.

Use `make check` to compile the site to html and check the URLs are valid.

This is only accessible on your computer, and won't be accessible to anyone else.

For more details see the [tech-docs-github-pages-publisher](https://github.com/ministryofjustice/tech-docs-github-pages-publisher) repository.

#### Publishing

Any changes you push/merge into the `main` branch should be published to GitHub Pages site automatically.

#### Template configuration

The webpage layout is configured using the config/tech-docs.yml file.

The template can be configured in [config/tech-docs.yml](config/tech-docs.yml)

Key config settings:

- `host:` - This should be the URL of your published GitHub Pages site, e.g:

  ```
  https://ministryofjustice.github.io/modernisation-platform
  ```

  > Do not include a `/` at the end of this URL

- `service_link:` - This should be the docpath to your site. This is usually
  `/[repo name]`, so if your repository is `ministryofjustice/awesome-docs`
  `service_link` will be `/awesome-docs`

Further configuration options are described on the Tech Docs Template website: [Global Configuration](https://tdt-documentation.london.cloudapps.digital/configure_project/global_configuration/).

