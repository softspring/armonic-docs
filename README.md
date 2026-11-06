---
title: "Armonic Documentation Repository"
description: "Source Markdown files for the Armonic documentation site, including guides, bundle references, and contribution information."
---

<p align="center"><a href="https://armonic.softspring.es/6.0/" target="_blank">
  <img src="https://storage.googleapis.com/softspring-images/d3ca848a7019bd4965f1b7da8ccb03c473b81dd0.png">
</a></p>

<h3 align="center">
  The Armonic Documentation
</h3>

<p align="center">
  <a href="https://armonic.softspring.es/6.0/">
    Read this documentation online
  </a>
    ·
  <a href="https://github.com/softspring/armonic">
    Fork Armonic on GitHub
  </a>
    ·
  Supported by
  <a href="https://softspring.es/">
    Softspring
  </a>
</p>

## Documentation navigation {#documentation-navigation}

`index.json` defines the public navigation. Each main section has an overview page, a `slug`, and a `file`, so its sidebar title opens a useful entry page:

- [Introduction](introduction/overview.md)
- [Getting started](getting-started/overview.md)
- [Armonic configuration](configuration/overview.md)
- [User manual](user-manual/overview.md)
- [Bundles](bundles/overview.md)
- [Components](components/overview.md)

Keep these pages focused on where to start and which guide to choose. Keep detailed instructions in the linked guides. The separate `components/components.md` page documents the UI components package.

The server adds a “Continue reading” footer to each indexed page. Previous/next links follow index order by default. Use `next: "configuration"` in frontmatter when a workflow should continue elsewhere, and a `related` list of indexed slugs for useful alternatives or supporting guides. The server also suggests the section overview and child pages. Links keep the selected documentation version, and missing destinations are ignored.

## Configuration examples {#configuration-examples}

Symfony configuration examples offer YAML and PHP tabs. Choose one format for each configuration file; merge the example into existing configuration instead of loading both alternatives. PHP examples use Configurator closures supported by Symfony 6.4, 7.4, and 8.0. Imports from installed bundles keep their original file format.

For authors, wrap exactly two fenced blocks, `yaml` followed by `php`, between `<!-- config-tabs:start -->` and `<!-- config-tabs:end -->`. Leave blank lines around the blocks and markers. The server remembers the reader's selected format. Without JavaScript, both examples remain readable. Native CMS definitions, translations, and Docker Compose files keep their own format. Database setup examples cover both MySQL and PostgreSQL; keep the driver, connection URL, server version, and Docker volume layout consistent. See [Configure MySQL or PostgreSQL](getting-started/configure-database.md).

## Contributing

Follow [the authoring instructions](AGENTS.md). Current release work targets `6.0`. Guides follow Symfony's practical technical style: explain the goal, establish prerequisites, show usable examples and describe verified limits. Compare page coverage with real CMS elements, bundle features and plugin configuration, adding missing guides with their navigation and package README links.

Review examples against the selected package branch. State whether a configuration block is complete or an override of an existing definition. Use disposable projects and synthetic data to check installation, migrations, forms, and editorial workflows. A successful Composer install or migration is only one part of validation: open the relevant screen and exercise its main action before describing that integration as working.

Use relative links to indexed Markdown files, including explicit heading anchors when linking to a subsection. Keep screenshots local to the guide's `.files/` directory. The documentation server's tests check version-aware links, navigation, configuration alternatives, and rendering; see its repository README for the commands.

Screenshots show real interfaces from the guide's package branch. Refresh them when the UI changes, using fictitious content and accounts. Keep older branches' images independent. Reference applications are read-only sources for generic patterns; never copy customer data, private configuration or assets into examples or captures. Keep page coverage and runtime findings in review reports and the associated issue, with a clear distinction between executed workflows, mocks and checks that remain unverified.

We will be happy to receive your contributions to this documentation. Feel free to fork the repository, make your
changes and send a pull request.
