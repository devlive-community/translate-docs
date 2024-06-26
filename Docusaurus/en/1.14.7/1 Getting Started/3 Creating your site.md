[TOC]

Docusaurus was created to hopefully make it super simple for you to create a site and documentation for your open source project.

After [installation]($Installation) and [preparation]($Site-Preparation), much of the work to create a basic site for your docs is already complete.

Site Structure
--------------

Your site structure looks like the following:

    root-directory
    ├── docs
    └── website
        ├── blog
        ├── core
        │   └── Footer.js
        ├── package.json
        ├── pages
        ├── sidebars.json
        ├── siteConfig.js
        └── static


> This assumes that you removed the example `.md` files that were installed with the [initialization]($Installation) script.

All of your documentation files should be placed inside the `docs` directory as markdown `.md` files. Any blog posts should be inside the `blog` directory.

> The blog posts must be formatted as `YYYY-MM-DD-your-file-name.md`

Create Your Basic Site
----------------------

To create a fully functional site, you only need to do a few steps:

1.  Add your documentation to the `/docs` directory as `.md` files, ensuring you have the proper [header]($Markdown-Features#documents) in each file. One example header would be the following, where `id` is the link name (e.g., `docs/intro.html`) and the `title` is the webpage's title.

        ---
        id: intro
        title: Getting Started
        ---
        My new content here..


2.  Add zero or more docs to the [`sidebars.json`]($Navigation-And-Sidebars#adding-docs-to-a-sidebar) file so that your documentation is rendered in a sidebar if you choose them to be.


> If you do not add your documentation to the `sidebars.json` file, the docs will be rendered, but they can only be linked to from other documentation and visited with the known URL.

3.  Modify the `website/siteConfig.js` file to [configure your site]($Site-Config-Js), following the comments included in the [docs]($Site-Config-Js) and the `website/siteConfig.js` to guide you.
4.  Create any [custom pages]($Custom-Pages) and/or [customize]($Custom-Pages) the `website/core/Footer.js` file that provides the footer for your site.
5.  Place assets, such as images, in the `website/static/` directory.
6.  Run the site to see the results of your changes.

    cd website
    yarn run start # or `npm run start`
    # Navigate to http://localhost:3000


Special Customization
---------------------

### Docs Landing Page

If you prefer to have your landing page be straight to your documentation, you can do this through a redirect.

1.  Remove the `index.js` file from the `website/pages` directory, if it exists.
2.  Add a [custom static `index.html` page]($Custom-Pages) in the `website/static` directory with the following contents:

    ```html
    <!DOCTYPE html>
    <html lang="en-US">
      <head>
        <meta charset="UTF-8" />
        <meta
          http-equiv="refresh"
          content="0; url=docs/id-of-doc-to-land-on.html"
        />
        <script type="text/javascript">
          window.location.href = 'docs/id-of-doc-to-land-on.html';
        </script>
        <title>Your Site Title Here</title>
      </head>
      <body>
        If you are not redirected automatically, follow this
        <a href="docs/id-of-doc-to-land-on.html">link</a>.
      </body>
    </html>
    ```

> You will get the `id` of the document to land on the `.md` metadata of that doc page.

### Blog Only

You can also use Docusaurus to host your [blog only]($Adding-A-Blog).
