# AGENTS.md

Working rules for documentation changes in this repository.

## Format

- Use Markdown for all documentation pages.
- Add frontmatter to every page.
- Frontmatter must include `title` and `description`.
- Add explicit ids to headings using the `{#...}` syntax.

## Structure

- Prefer a single documentation file for each bundle or component.
- For bundles and components, use a single file such as `bundles/media-bundle.md` or `components/foo.md`.
- Do not use `index.md` inside bundle or component directories.
- Classify pages in `bundles/` or `components/` by the kind of functionality they expose, not only by the package technical type.
- Reserve `bundles/` mainly for user-facing packages with controllers, screens, or application-facing flows.
- Place infrastructure or technical integration packages in `components/` even if they are implemented as Symfony bundles.
- Organize the page in titled sections, similar to Symfony documentation.
- Keep English simple and direct.
- Follow the communication guidelines from `VOZ_DE_MARCA.md` while keeping the result as technical documentation.
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
- Use `FEATURES.md` as the starting point for documentation structure, but always verify the current code and real integrations before writing or changing guides.
- Prefer current configuration, commands, and examples over historical content.
- If a bundle or component already has documentation in its local `docs/` directory, reuse the useful parts when moving that content to `armonic-docs`.
- After moving useful documentation from a package `docs/` directory to `armonic-docs`, remove the old package documentation files.
- Update indexes and internal links when pages are merged, renamed, or removed.
- Do not add a `Validation and Maintenance` section to pages in `armonic-docs`.
- Package maintenance commands and validation information should go to the `Contributing` section of each package `README.md`.
