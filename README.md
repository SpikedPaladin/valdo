# Valdo

_(like "**Val**a **Do**", pronounced like "Waldo")_

Create a new Vala project from a repository of templates.

## Example use

`valdo new` - initializes a new project (from a template `new`) in the current directory

`valdo gtk` - initializes a new GTK app

`valdo lib` - initializes a new library

`valdo -l` - lists all available templates

## Creating a new a template

Templates should be added here so that they can be used by everyone.

Fork this repository and add a new directory under `templates/`. The name of
the directory is the template name. Then you need to add a `template.json`
describing the template, the variables it uses, and the files that need to be
substituted.

Here is what a template might look like:

```
.
├── meson.build.in
├── README.md.in
├── src
│   ├── main.vala
│   └── meson.build.in
└── template.json

1 directory, 5 files
```

Each file ending with `.in` is a template file.

Your `template.json` may start off looking like:
```json
{
    "description": "a bare app, with minimal dependencies",
    "variables": {
        "PROGRAM_NAME": {
            "summary": "the name of the program",
            "default": "main"
        }
    },
    "inputs": [
        "src/meson.build.in",
        "meson.build.in",
        "README.md.in"
    ]
}
```

`"variables"` is a dictionary mapping each variable name to a short
description. Having a default value for a variable is optional. There are at
least two variables every template uses, `PROJECT_NAME` and `PROJECT_VERSION`,
which you don't have to specify.

For every variable listed, the template engine will substitute
`${VARIABLE_NAME}` in each templated file listed in `"inputs"` after prompting
the user. All other files in the template directory will be copied over
unmodified. Templated files should end with `.in`.

**Once you're done, submit a PR to https://github.com/Prince781/valdo**

### Advanced template features

#### Variable substitution

You can define a variable's default value in terms of another variable like so:

```json
{
    "variables": {
        "API_NAMESPACE": {
            "summary": "the API namespace",
            "default": "MyLib",
            "pattern": "^[A-Za-z][[:word:]]*$"
        },
        "LIBRARY_NAME": {
            "summary": "the name of the library",
            "default": "/${API_NAMESPACE}/\\w+/\\L\\g<0>\\E/",
            "pattern": "^[[:word:]][[:word:]-]*$"
        },
    }
}
```

Here, this means that `LIBRARY_NAME` will be auto-generated from the namespace
name, unless the user enters something different in a prompt [^1]. In this case, if
the user enters nothing for `API_NAMESPACE` and `LIBRARY_NAME`, the result will
be `mylib`.

The syntax is to write the variable name, then a regular expression that
matches something you want to replace, then a regular expression for the
replacement. Here, `\L` says "make lowercase", `\g<0>` matches group 0 or the
entire string matched, and `\E` ends the conversion. Consult the [docs for
GLib.Regex.replace()](https://valadoc.org/glib-2.0/GLib.Regex.replace.html) for
more information about how regex syntax is used in GLib.

If you wanted to substitute another pattern in the text, you just have to add
another pair of regexes separated by a `/`. For example:

```
"default": "/${API_NAMESPACE}/\\w+/\\L\\g<0>\\E/bob/greta/"
```

After converting everything to lowercase, then substitute "bob" for "greta".
The substitutions will be applied in the order they appear, one after another.

[^1]: You can specify `"auto": true` for the variable to make it completely
  automatic, so there's no prompting of the user. This is useful when
  generating variable names in build scripts, for example.
