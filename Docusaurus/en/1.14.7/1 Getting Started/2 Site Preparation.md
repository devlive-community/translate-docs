[TOC]

After [installing Docusaurus]($Installation), you now have a skeleton to work from for your specific website. The following discusses the rest of the Docusaurus structure in order for you to prepare your site.

Directory Structure
-------------------

As shown after you [installed Docusaurus]($Installation), the initialization script created a directory structure similar to:

    root-directory
    ├── .gitignore
    ├── docs
    │   ├── doc1.md
    │   ├── doc2.md
    │   ├── doc3.md
    │   ├── exampledoc4.md
    │   └── exampledoc5.md
    └── website
        ├── blog
        │   ├── 2016-03-11-blog-post.md
        │   ├── 2017-04-10-blog-post-two.md
        │   ├── 2017-09-25-testing-rss.md
        │   ├── 2017-09-26-adding-rss.md
        │   └── 2017-10-24-new-version-1.0.0.md
        ├── core
        │   └── Footer.js
        ├── package.json
        ├── pages
        ├── sidebars.json
        ├── siteConfig.js
        └── static


### Directory Descriptions

*   **Documentation Source Files**: The `docs` directory contains example documentation files written in Markdown.
*   **Blog**: The `website/blog` directory contains examples of blog posts written in markdown.
*   **Pages**: The `website/pages` directory contains example top-level pages for the site.
*   **Static files and images**: The `website/static` directory contains static assets used by the example site.

### Key Files

*   **Footer**: The `website/core/Footer.js` file is a React component that acts as the footer for the site generated by Docusaurus and should be customized by the user.
*   **Configuration file**: The `website/siteConfig.js` file is the main configuration file used by Docusaurus.
*   **Sidebars**: The `sidebars.json` file contains the structure and order of the documentation files.
*   **.gitignore**: The `.gitignore` file lists the necessary ignore files for the generated site so that they do not get added to the git repo.

Preparation Notes
-----------------

You will need to keep the `website/siteConfig.js` and `website/core/Footer.js` files but may edit them as you wish. The value of the `customDocsPath` key in `website/siteConfig.js` can be modified if you wish to use a different directory name or path. The `website` directory can also be renamed to anything you want it to be.

However, you should keep the `website/pages` and `website/static` directories. You may change the content inside them as you wish. At the bare minimum, you should have an `en/index.js` or `en/index.html` file inside `website/pages` and an image to use as your header icon inside `website/static`.

If your directory does not yet have a `.gitignore`, we generate it with the necessary ignored files listed. As a general rule, you should ignore all `node_modules`, build files, system files (`.DS_Store`), logs, etc. [Here](https://github.com/github/gitignore/blob/master/Node.gitignore) is a more comprehensive list of what is normally ignored for Node.js projects.