# API Writer

This document outlines a proposal for a way to use [Markdoc](https://markdoc.dev/) to author API descriptions. This document does not propose any kind of standard at this point. All the examples in this document are up for debate and are meant as possibilities and not a specification.

## Guiding principles

- **It should feel like writing documentation**. It shouldn't feel like writing YAML or code. It should feel like you're writing a Markdown file and sprinkling in details throughout it.
- **It should allow for progressive authoring**. You should be able to write the documentation an enhance it afterwards. The format should not force the author to think about HTTP and JSON details.
- **It should transpile to OpenAPI**. The goal is for interoperability with the existing ecosystem, not to create something new and different.
- **It should begin as strict and opinionated**. OpenAPI has a lot of functionality. There's no need to support every feature there. Keep it basic at first and grow as the needs arise. A new format like this is an opportunity to have a narrow focus around authoring.
- **It should have good defaults**. When you define a property on an object, it should be a string. When you define a response, it should be `application/json` and a `200`. You should be able to overwrite these, but this is another area to be opinionated about. Some of this could also be changed through configuration.
- **It should be built with change in mind**. The format should consider how someone might make changes to the design over time. It should provide the means to mark annotate parts of the API design with metadata that allows for generate context-specific OpenAPI documents, such as one for production or one for a specific vendor.

## Proposal: Using Markdoc to create an API description format

[Markdoc](https://markdoc.dev) is a tool that allows for writing and extending Markdown documents. It supports extending a Markdown file using things like tags, attributes, variables, partials, and functions. It provides all the building blocks needs to make an API-focused format.

Markdoc is more than a templating tool. It provides ways to access the AST after it's parsed and extend the AST with several different features. This feature of making the Markdown extensible at the AST level is what provides the functionality needed to create a specialized API format. Markdoc provides the parser and renderer functionality that's needed. And it provides the ability to extend it all.

### Use annotations to add technical details

I should be able to write something like this in Markdown.

```markdown
# Resources
## Users
### List
### Create

## User
### Retreive
### Update
### Delete
```

I should be able to add descriptions through writing plain Markdown. This should all be valid when writing the API description. I should be able annotate the Markdown to give meaning to those values.

```markdown
# Resources
## Users {% .resource url="/users" %}
### List {% .get }
### Create {% .post %}

## User {% .resource url="/users/{id}" %}
### Retreive {% .get %}
### Update {% .put %}
### Delete {% .delete %}
```

In Markdoc, the `{% ... %}` is a tag, and when an attribute is something like `{% .get %}`, it is shorthand for `{% class="get" %}`.

This can also apply to schemas.

```markdown
# Schemas
## Users {% .schema %}
- items {% .array .items="User" %}

## User {% .schema %}
- name {% .required } Here is the description for the name
- age {% .number .nullable }
  Here is a description below the property
- email {% format="email" %}
- phone {% format="phone" %}
```

### Use tags to connect things in the document

Markdoc provides tags

```markdown
# Resources
## Users {% .resource url="/users" %}
### List {% .get }
{% response schema="Users" %}
### Create {% .post %}
{% request schema="Create User" %}
  Some description here
{% /request %}

<!-- self closing response -->
{% response schema="User" status=201 /%}

{% error status=[400, 500] schema="Error" /%}

# Schemas
## Users {% .schema %}
- items {% .array .items="User" %}

## Create User
- name {% .require %}

## User {% .schema extends="Create User" %}
- id {% .required }
```

### Use configurations to define styles and standards

In the example above, I used normal titles for the headings. Markdoc provides a way to include configurations for the compiler. Details like "use snake case for object properties" could be a configuration that is used when rendering an OpenAPI document.

### Use partials to break up documents

Markdoc comes with a way to include other documents into another.

```markdown
# Resources
{% partial file="users-resource.md" /%}
{% partial file="user-resource.md" /%}

# Schemas
{% partial file="user-schema.md" /%}
```

This could work out of the box, and could also be extended to propose a directory structure. **I would not start with this**. I only mention as an example.

```markdown
<!-- Includes all resource and schema partials -->
{% resource name="users" /%}
```

### Use frontmatter for high-level details

This is an example of how the format might use frontmatter to add additional information outside the scope of the API Design.

```markdown
---
title: My API
hosts:
  production: "api.example.com",
  staging: "api-staging.example.com"
---

# Resources
...

# Schemas
...
```

### Use custom tags for better documentation

Markdoc allows for defining custom tag. These can be used to provide better documentation. This below could be used to generate HTML tags that include styling and icons if applicable.

```markdown
# Resources
## User
{% callout type="warn" %}
  This is a beta resource. The details of this may change.
{% /callout %}
```

### Use attributes for evolving the document

The below example shows a resource that is in the development phase. The renderer would ignore this resource when generating an OpenAPI for production.

```markdown
# Resources
## User {% .resource url="/users" phase="development" %}
```

### Use tags for extending the documentation

OpenAPI provides `x-*` for extending the API description. This could use a special tag for this as well.

```markdown
# Resources
## Users
### List
<!-- Markdoc allows for any JSON for attribute values -->
{% extend 
   name="amazon-apigateway-any-method" 
   value={foo: "bar"} /%}
```

## Tradeoffs

- **It's another meta-language to learn**. You have to know what to write and where to put things in the document.
- **The syntax might be off-putting**. Having to use `{% ... %}` everywhere might not sit well with people.
- **It doesn't all look like Markdown**. One nice thing of API Blueprint was that it all felt like Markdown. Markdoc tags feel like you're breaking out of the Markdown.