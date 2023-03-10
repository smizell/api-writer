---
author: Stephen Mizell
---

# Proposal: Use Markdoc to create an API description format

This document outlines a proposal for a way to use [Markdoc](https://markdoc.dev/) to author API descriptions. This document does not propose any kind of standard at this point. All the examples in this document are up for debate and are meant as possibilities and not a specification. Markdoc is a tool that allows for writing and extending Markdown documents. It supports extending a Markdown file using things like tags, attributes, variables, partials, and functions. It provides all the building blocks needs to make an API-focused format.

Markdoc is more than a templating tool. It provides ways to access the AST after it's parsed and extend the AST with several different features. This feature of making the Markdown extensible at the AST level is what provides the functionality needed to create a specialized API format. Markdoc provides the parser and renderer functionality that's needed. And it provides the ability to extend it all.

## Benefits

- **Existing parser**. Markdoc provides many of the tools needed to extend and parse Markdown. It already has features for annotating Markdown, so there would be no need to build special tooling for parsing. It would only require building a customer renderer for OpenAPI and potentially one for generating HTML.
- **Supports the guiding principles**. Markdoc would support the guiding principles out of the box.
- **Great features out of the box**. Markdoc supports adding variables and functions to a document, along with including other Markdoc files. This provides features that would normally be left until later in development for a parser.

## Tradeoffs

- **It's another meta-language to learn**. You have to know what to write and where to put things in the document.
- **The syntax might be off-putting**. Having to use `{% ... %}` everywhere might not sit well with people.
- **It doesn't all look like Markdown**. One nice thing of API Blueprint was that it all felt like Markdown. Markdoc tags feel like you're breaking out of the Markdown.

## Details

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
