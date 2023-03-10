# API Writer

This repository includes proposals for creating a Markdown format that transpiles to OpenAPI.

## Guiding principles

- **It should feel like writing documentation**. It shouldn't feel like writing YAML or code. It should feel like you're writing a Markdown file and sprinkling in details throughout it.
- **It should allow for progressive authoring**. You should be able to write the documentation an enhance it afterwards. The format should not force the author to think about HTTP and JSON details.
- **It should transpile to OpenAPI**. The goal is for interoperability with the existing ecosystem, not to create something new and different.
- **It should begin as strict and opinionated**. OpenAPI has a lot of functionality. There's no need to support every feature there. Keep it basic at first and grow as the needs arise. A new format like this is an opportunity to have a narrow focus around authoring.
- **It should have good defaults**. When you define a property on an object, it should be a string. When you define a response, it should be `application/json` and a `200`. You should be able to overwrite these, but this is another area to be opinionated about. Some of this could also be changed through configuration.
- **It should be built with change in mind**. The format should consider how someone might make changes to the design over time. It should provide the means to mark annotate parts of the API design with metadata that allows for generate context-specific OpenAPI documents, such as one for production or one for a specific vendor.

## Proposals

- [Using Markdoc](./proposals/markdoc.md)

## Creating a proposal

- Add a `.md` file to the `proposals` directory
- Include a summary along with benefits, tradeoffs, and details
- Add an `author` to the proposal frontmatter
- Open a Pull Request

