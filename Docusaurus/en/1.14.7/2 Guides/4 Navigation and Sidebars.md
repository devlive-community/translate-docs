[TOC]

Referencing Site Documents
--------------------------

If you want to reference another document in your `docs` directory (or the location you set via the [optional `customDocsPath`]($Site-Config-Js) path site configuration option), then you just use the name of the document you want to reference.

For example, if you are in `doc2.md` and you want to reference `doc1.md`:

```
I am referencing a [document](doc1.md).
```

How Documents are Linked
------------------------

New markdown files within `docs` will show up as pages on the website. Links to those documents are created first by using the `id` in the header of each document. If there is no `id` field, then the name of the file will serve as the link name.

For example, creating an empty file such as `docs/getting-started.md` will enable the new page URL as `/docs/getting-started.html`.

Suppose you add this to your document:

```
id: intro
title: Getting Started
---
My new content here..
```

If you set the `id` field in the markdown header of the file, the doc will then be accessed from a URL of the form `/docs/intro.html`.

> You need an `id` field to be able to add the document to the sidebar.

Adding Documents to a Sidebar
-----------------------------

Many times, you will want to add a document to a sidebar that will be associated with one of the headers in the top navigation bar of the website. The most common sidebar, and the one that comes installed when Docusaurus is initialized, is the `docs` sidebar.

> "docs" is just a name. It has no inherent meaning. You can change it as you wish.

You configure the contents of the sidebar, and the order of its documents, in the `website/sidebars.json` file.

> Until you add your document to `website/sidebars.json`, they will only be accessible via a direct URL. The doc will not show up in any sidebar.

Within `sidebars.json`, add the `id` you used in the document header to the existing sidebar/category. In the below case, `docs` is the name of the sidebar and `Getting Started` is a category within the sidebar.

```json
{
  "docs": {
    "Getting Started": [
      "getting-started"
    ],
    ...
  },
  ...
}
```

Or you can create a new category within the sidebar:

```json
{
  "docs": {
    "My New Sidebar Category": [
      "getting-started"
    ],
    ...
  },
  ...
}
```

However, for a document located in a docs subdirectory like below:

```
docs
└── dir1
    └── getting-started.md
```

You should provide `directory/id` instead of `id` in `sidebars.json`.

```json
{
  "docs": {
    "My New Sidebar Category": [
      "dir1/getting-started"
    ],
    ...
  },
  ...
}
```

### Adding Subcategories

It is possible to add subcategories to a sidebar. Instead of using IDs as the contents of the category array like the previous examples, you can pass an object where the keys will be the subcategory name and the value an array of IDs for that subcategory.

```json
{
  "docs": {
    "My Example Category": [
      "examples",
      {
        "type": "subcategory",
        "label": "My Example Subcategory",
        "ids": [
          "my-examples",
          ...
        ]
      },
      {
        "type": "subcategory",
        "label": "My Next Subcategory",
        "ids": [
          "some-other-examples"
        ]
      },
      "even-more-examples",
      ...
    ],
    ...
  }
}

/*
The above will generate:

- My Example Category
  - examples
  - My Example Subcategory
    - my-examples
    ...
  - My Next Subcategory
    - some-other-examples
  - even-more-examples
  ...
*/
```


### Adding New Sidebars

You can also put a document in a new sidebar. In the following example, we are creating an `examples-sidebar` sidebar within `sidebars.json` that has a category called `My Example Category` containing a document with an `id` of `my-examples`.

```json
{
  "examples-sidebar": {
    "My Example Category": [
      "my-examples"
    ],
    ...
  },
  ...
}
```

It is important to note that until you [add a document from the `"examples-sidebar"` sidebar to the nav bar](#additions-to-the-site-navigation-bar), it will be hidden.

Additions to the Site Navigation Bar
------------------------------------

To expose sidebars, you will add click-able labels to the site navigation bar at the top of the website. You can add documents, pages and external links.

### Adding Documents

After creating a new sidebar for the site by [adding](#adding-new-sidebars) it to `sidebars.json`, you can expose the new sidebar from the top navigation bar by editing the `headerLinks` field of `siteConfig.js`.

```js
{
  headerLinks: [
    ...
    { doc: 'my-examples', label: 'Examples' },
    ...
  ],
  ...
}
```


A label called `Examples` will be added to the site navigation bar and when you click on it at the top of your site, the `examples-sidebar` will be shown and the default document will be `my-examples`.

### Adding Custom Pages

To add custom pages to the site navigation bar, entries can be added to the `headerLinks` of `siteConfig.js`. For example, if we have a page within `website/pages/help.js`, we can link to it by adding the following:

```json
{
  headerLinks: [
    ...
    { page: 'help', label: 'Help' },
    ...
  ],
  ...
}
```

A label called `Help` will be added to the site navigation bar and when you click on it at the top of your site, the content from the `help.js` page will be shown.

### Adding External Links

Custom links can be added to the site navigation bar with the following entry in `siteConfig.js`:

```json
{
  headerLinks: [
    ...
    { href: 'https://github.com/facebook/docusaurus', label: 'GitHub' },
    ...
  ],
  ...
}
```

A label called `GitHub` will be added to the site navigation bar and when you click on it at the top of your site, the content of the external link will be shown.

> To open external links in a new tab, provide an `external: true` flag within the header link config.

Site Navigation Bar Positioning
-------------------------------

You have limited control where the search and languages dropdown elements are shown in the site navigation bar at the top of your website.

### Search

If search is enabled on your site, your search bar will appear to the right of your links. If you want to put the search bar between links in the header, add a search entry in the `headerLinks` config array:

```json
{
  headerLinks: [
    { doc: 'foo', label: 'Foo' },
    { search: true },
    { doc: 'bar', label: 'Bar' },
  ],
  ...
}
```

### Languages Dropdown

If translations are enabled on your site, the language dropdown will appear to the right of your links (and to the left of the search bar, if search is enabled). If you want to put the language selection dropdown between links in the header, add a languages entry in the `headerLinks` config array:

```json
{
  headerLinks: [
    { doc: 'foo', label: 'Foo' },
    { languages: true },
    { doc: 'bar', label: 'Bar' },
  ],
  ...
}
```

Active Links In Site Navigation Bar
-----------------------------------

The links in the top navigation bar get `siteNavItemActive` and `siteNavGroupActive` class names to allow you to style the currently active link different from the others. `siteNavItemActive` is applied when there's an exact match between the navigation link and the currently displayed web page.

> This does not include links of type `href` which are meant for external links only. If you manually set an `href` in your `headerLinks` to an internal page, document, or blog post, it will not get the `siteNavItemActive` class even if that page is being displayed.

The `siteNavGroupActive` class will be added to these links:

*   `doc` links that belong to the same sidebar as the currently displayed document
*   The blog link when a blog post or the blog listing page is being displayed

These are two separate class names so you can have the active styles applied to either exact matches only or a bit more broadly for docs that belong together. If you don't want to make this distinction you can add both classes to the same CSS rule.

Secondary On-Page Navigation
----------------------------

We support secondary on-page navigation so you can quickly see the topics associated with a given document. To enable this feature, you need to add the `onPageNav` site configuration [option]($Site-Config-Js) to your `siteConfig.js`.

```json
{
  onPageNav: 'separate',
  ...
}
```

Currently, `'separate'` is the only option available for this field. This provides a separate navigation on the right side of the page.

Collapsible Categories
----------------------

For sites with a sizable amount of content, we support the option to expand/collapse the links and subcategories under categories. To enable this feature, set the `docsSideNavCollapsible` site configuration [option]($Site-Config-Js) in `siteConfig.js` to true.

```json
{
  docsSideNavCollapsible: true,
  ...
}
```
