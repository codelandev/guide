# HTML (Slim) Style guide

## Table of content
1. [Protocol](#protocol)
2. [General formating rules](#general-formating-rules)
3. [Slim Formating rules](#slim-formating-rules)

### Protocol
Omit the protocol from embedded resources.

Omit the protocol portion (http:, https:) from URLs pointing to images and other media files, style sheets, and scripts unless the respective files are not available over both protocols.

Omitting the protocol—which makes the URL relative—prevents mixed content issues and results in minor file size savings.

```slim
/ Not recomended
= image_tag "http://www.placehold.it/100x100"
```

```slim
/ Recommended
= image_tag "//www.placehold.it/100x100"
```

```sass
// Not recommended
.example {
  background: url(http://www.google.com/images/example);
}
```
```sass
// Recommended
.example {
  background: url(//www.google.com/images/example);
}
```
## General formating rules

### Indentation rules
- Indent by 2 spaces at a time.
- Use only lowercase.
- Remove trailing white spaces.

### Comments
Explain code as needed, where possible. Follow our comments pattern.

### Action item
Mark todos and action items with TODO.
Highlight todos by using the keyword TODO only, not other common formats like @@. eg:
```slim
/ TODO: Something to do
ul
  li Apples
  li Oranges
```

## Slim Formating rules
### Document type
Use HTML5 `doctype html` or `doctype 5`.  
Although fine with HTML, do not close void elements.

### Semantic
Use elements (sometimes incorrectly called “tags”) for what they have been created for. For example, use heading elements for headings, p elements for paragraphs, a elements for anchors, etc.

### Entity reference
Do not use entity references.
```slim
/ Not recommended
p The currency symbol for the Euro is &ldquo;&eur;&rdquo;.
```
```slim
/ Recommended
p The currency symbol for the Euro is “€”.
```

### Text content
Alwas break the line and use a pipe.

```slim
/ Not recommended
body
  h1 id="headline" Welcome to my site.
```
```slim
/ Recommended
body
  h1 id="headline"
  | Welcome to my site.
```

### General formating
Use a new line for every block, list, or table element, and indent every such child element.

### Quotation Marks
When quoting attributes values, use double quotation marks.

### Attributes
Don't use attributes wrappers
```slim
/ Not recomended
body
  h1(id="logo") = page_logo
  h2[id="tagline" class="small tagline"] = page_tagline
```
```slim
/ Recommended
a href="http://slim-lang.com" title='Slim Homepage' Goto the Slim homepage
```

#### Boolean attributes
Only set the tribute without the boolean, if a elements has a attribute obviously it's true.

```slim
/ Not recommended
input type="text" disabled="true"
```
```slim
/ Recommended
input type="text" disabled
```