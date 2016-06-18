# dependency-env
Loads environment variables that dependencies declared as exported in their package.json

# NOT PRODUCTION READY

### Idea:

> Bring the sandboxing model to environment variables.

> If you add `dependency-env` to your `dependencies`, then you can source `dependencyEnv`
> in any of your npm `scripts`, and it will ensure that your immediate dependencies
> (and *only* your immediate dependencies) can set environment variables that
> are set only for the remainder of that one script. You must use the `. ` syntax to
> "source" `dependencyEnv`.

Inside your package.json:

```
"scripts": {
  "doStuff": ". dependencyEnv && restOfCommandHere"
}
```

Then on the command line you can do:

```
npm run-script doStuff
```

> Note that in these examples, `dependencyEnv` itself is not a globally installed
binary. When you depend on `dependency-env`, it makes sure that `dependencyEnv`
is in `./node_modules/.bin/`, which `npm` always ensures is added to your path
when for the duration of `run-script`s. This is standard `npm` behavior that
we're using here - we happen to be using it to be bootstrapping creating an
environment that uses an (arguably) better model for constructing `PATH`s and
any other env variable.

> You might want to source `. node_modules/.bin/dependencyEnv` directly in your 
own build scripts because `npm run-script` has a large overhead.


### Conventions:

- Any `package.json` may include an `"exportedEnvVars"` field which maps
  environment variables by their names to their respective variable configs.
- 
- Each variable config should have a `"val"` field.
- By default environment variables are scoped to the package they reside in. This means that
the environment variable name must be prefixed to indicate the package it was
exported from. We do this by placing `PACKAGE_NAME__` at the beginning of each
exported var from the `package.json` for the package named `package-name` (we
replace hyphens with single underscores and then include a double underscore.
For example, for a package named `my-package`, by default, environment
variables must begin with `MY_PACKAGE__`, such as `MY_PACKAGE__FOO`, or
`MY_PACKAGE__BAR_BAR`.
- Environment variables may declare that they are paths that should be absolute
  paths that are resolved relative to the location of the `package.json` they
  reside in. Set "resolveAsRelativePath": false
- Environment variables may declare themselves global by including `"global":
  true` in their configuration. For global variables, there is another optional
  field called `globalCollisionBehavior` which determines how to resolve values
  when multiple packages set the same variable name. Options are `"fail"`(the
  default), `"clobber"`, or `"joinPath"` (which will combine all the values
  from all package via `:`).

- See below for examples:


### Examples of package.json fields:

```json
  "exportedEnvVars": {
    "PATH": {
      "val": "./src/_stage2",
      "resolveAsRelativePath": true,
      "global": true,
      "globalCollisionBehavior": "joinPath"
    }
    },
    "PACKAGE_NAME__SOME_VAR": {
      "val": "foo",
      "resolveAsRelativePath": false
    },
    "ANOTHER_GLOBALVAR": {
      "val": "./lib/ocaml",
      "resolveAsRelativePath": true,
      "global": true,
      "globalCollisionBehavior": "fail"
    }
  },
```


### Motivation:

`npm`'s run-script has a great feature that will augment the `PATH` environment
variable to include `/node_modules/.bin`, only for the duration of the
run-script. This is great because it means changing of paths is *scoped* to a
particular duration, and in a predefined way that your *dependencies* get to
influence. But there are some shortcomings of `npm` `run-script`, and some ways
that we can take the idea further beyond simply augmenting a `PATH`, but
setting and augmenting arbitrary environment variables.

- `npm` `run-script` only modifies the `PATH` and only for the sake of setting
  up `bin`. Dependencies might want to set other environment variables.
- `npm` `run-script` is slow (often hundreds of `ms` overhead to startup).
  Let's not be slow. The examples here demonstrate using `dependency-env` with
  `run-script` in this doc are merely for convenience. `dependency-env`
  provides a way to not *need* `run-script` to correctly wire up paths to
  binaries that your deps publish if your dependencies configure `PATH` using
  `dependency-env`.
