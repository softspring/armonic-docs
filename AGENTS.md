# AGENTS.md

Working rules for documentation changes in this repository.

## Branch and editorial scope

- Current release preparation targets `6.0`. Keep new guides, edits and refreshed captures on that branch unless the user explicitly changes the target.
- Keep each older branch's examples and captures with that branch. The initial `5.5` copy of `5.4` is an acknowledged starting point, not permission to replace its screenshots with 6.0 captures or claim complete functional equivalence.
- Review guides page by page for usefulness, readability, current behavior and missing topics. Compare coverage with CMS elements and bundle/plugin code, configuration and real application usage, rather than checking package names alone.
- Follow Symfony's technical documentation style: professional, clear, practical and detailed when necessary. Lead with what users can do, introduce prerequisites before steps, and use examples to explain capabilities without marketing claims.

- Exclude experimental package guides from `6.0`, including CMS AI, Analytics, SEO, API, MCP, Mate, and Form Schema. Add them only to the future `6.1@dev` documentation after revalidation; do not create or enable that version as part of a scope edit.
- Exclude Notification Bundle and Subscription Bundle from Armonic documentation until their update and relevance have been reviewed. Keep navigation, related links, catalogs and package README destinations consistent with these exclusions.

## Format

- Use Markdown for all documentation pages.
- Add frontmatter to every page.
- Frontmatter must include `title` and `description`.
- Add explicit ids to headings using the `{#...}` syntax.
- Pair Symfony configuration examples as YAML then PHP between `<!-- config-tabs:start -->` and `<!-- config-tabs:end -->`, with blank lines around each fence. Use the PHP Configurator closure syntax supported by Symfony 6.4 and later. Do not add a standalone-use tab or invent PHP alternatives for native CMS definitions, translations, or Docker Compose files.
- Use `ContainerConfigurator` for package/service configuration and `RoutingConfigurator` for routing files. Keep imports, route prefixes, defaults and environment-specific behavior equivalent in both alternatives.
- Add heading ids only to actual Markdown headings; never alter comments inside fenced code examples while processing headings.

## Structure

- Give each top-level navigation group an overview page and `slug`/`file` entries in `index.json`. Keep its overview links aligned with its child guides.
- Give major nested sections, including the CMS entry, a useful landing page when their title opens a group of guides. A section title should help readers start, rather than only expanding an empty heading.
- Use optional `next` and `related` frontmatter slugs for meaningful reading paths. Keep every destination in the same branch's index; the server renders the shared page footer, so do not duplicate its previous/next links in Markdown.
- Choose continuations by the reader's workflow, as in Symfony documentation. Link to prerequisites, the next practical task and relevant feature guides; update them when adding or reorganizing pages.

- Prefer a single documentation file for each bundle or component.
- For bundles and components, use a single file such as `bundles/media-bundle.md` or `components/foo.md`.
- Do not use `index.md` inside bundle or component directories.
- Classify pages in `bundles/` or `components/` by the kind of functionality they expose, not only by the package technical type.
- Reserve `bundles/` mainly for user-facing packages with controllers, screens, or application-facing flows.
- Place infrastructure or technical integration packages in `components/` even if they are implemented as Symfony bundles.
- Organize the page in titled sections, similar to Symfony documentation.
- Keep English simple and direct.
- Follow the communication guidelines from `../armonic-docs-server/VOZ_DE_MARCA.md` while keeping the result as technical documentation.
- Write in a clear, honest, practical, and accessible way, avoiding unnecessary jargon and empty marketing language.

## Content

- Document real bundle and component behavior from the source code.
- Read the package `FEATURES.md` first when it exists and use it as the functional base for the guide.
- Treat `FEATURES.md` as a functional contract for behavior and scope, not as a documentation outline to copy mechanically.
- Read the relevant classes, configuration, forms, commands, Twig extensions, services, and examples before writing documentation.
- Do not write superficial documentation based only on package metadata or previous docs.
- Focus first on the functionality the bundle or component provides, the problem it solves, and how to use it well.
- Document the relevant functional behavior of each bundle or component with enough detail to be useful in practice.
- Prioritize usage guides for each relevant feature over explanations centered on internal implementation details.
- Explain how to use the bundle or component in real applications, not only what classes it contains.
- Treat bundles and components as extensible tools, and document the supported extension points and the recommended extension patterns.
- When possible, use other bundles, components, and `armonic-standalone` as references to show how the package is integrated in practice.
- When authorized, consult `site-new` and `softspring-web-project` as read-only configuration references. Rewrite examples with generic values; do not include customer data, internal URLs, credentials, branding or screenshots.
- Use `FEATURES.md` as the starting point for documentation structure, but always verify the current code and real integrations before writing or changing guides.
- Prefer current configuration, commands, and examples over historical content.
- Identify override fragments explicitly; complete examples must include required schema fields and the classes or templates they reference.
- Verify the relevant runtime flow in a disposable project before claiming compatibility. Record unsupported integrations and validation limits honestly, even when Composer and database migrations succeed.
- When documenting database setup, include MySQL and PostgreSQL alternatives with matching drivers, URLs, server versions, and persistent volumes. Keep platform-specific troubleshooting clearly scoped to its database.
- Carry both database alternatives through every affected setup example, not just the requirements page. Check plugin-specific queries and functions before extending a core database compatibility claim to a plugin.
- Compatibility summaries should show Symfony LTS lines and the latest stable line, omitting intermediate non-LTS columns such as 7.0–7.3. Derive support and PHP requirements from the selected branches' Composer constraints; presentation must not change those constraints.
- When adding a package guide, update its branch-specific README documentation link to that guide. Keep detailed usage here and package README content concise.
- If a bundle or component already has documentation in its local `docs/` directory, reuse the useful parts when moving that content to `armonic-docs`.
- After moving useful documentation from a package `docs/` directory to `armonic-docs`, remove the old package documentation files.
- Update indexes and internal links when pages are merged, renamed, or removed.
- Do not add a `Validation and Maintenance` section to pages in `armonic-docs`.
- Package maintenance commands and validation information should go to the `Contributing` section of each package `README.md`.

## Screenshots and runtime verification

- Capture real interfaces from applications running the documentation's target package branch. Refresh screenshots when UI styles or workflows change; do not invent interfaces or use mockups as proof of functionality.
- Store screenshots in the relevant guide's `.files/` directory and use relative Markdown image links. Keep images in the same Git branch as their guide; do not hardcode another branch's raw image URL.
- Use isolated application copies, fresh test databases, fictitious accounts and synthetic content. Never reset a developer or customer database to prepare documentation captures.
- Execute documented commands and configurations, then check the affected forms, generated output, public routes and main user actions. Record any extra setup or workaround instead of implying the original sequence passed unchanged.
- Validate complete examples against the real configuration schema and referenced classes/templates. Validate explicit override fragments with the definition they inherit. Load YAML/PHP alternatives with the correct Symfony loaders, and validate Twig with the relevant extensions available.
- Use the server checks documented in `../armonic-docs-server/README.md` for indexed navigation, relative links, configuration alternatives, image origins and rendering. Check image loading, reading paths and representative desktop/mobile pages in a browser after material content or capture changes.
- Distinguish syntax/schema checks from executed feature flows. State when mocks, optional-helper fixtures or unavailable provider credentials limit validation. Do not present an incomplete API or integration as working because its schema accepts the configuration.
- Keep page coverage, executed steps, compatibility findings and remaining validation limits in a review report and the associated issue at meaningful milestones. Do not turn public guides or this file into a task log.
