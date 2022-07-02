# garethrbrown.github.io

Static site (Jekyll) for GitHub pages. 

Published at https://garethbrown.net

## Development Environment

### Using GitHub CodeSpaces

`devcontainer.json` is set up running Ubuntu with Ruby features:

```json
"features": {
    "ruby": "3.1"
}
```

Following workspace container build / rebuild, Jekyll needs to be installed:

```
$ gem install jekyll bundler
```

Check with

```
$ jekyll -v
```

You will likely need to install gems from Gemfile.lock

```
$ bundle install
```

Due to error `cannot load such file -- webrick (LoadError)`, you may need to add the following package. (Ref: https://stackoverflow.com/questions/65617143/cannot-load-such-file-webrick-httputils)

```
$ bundle add webrick
```

## Debug

To debug locally run:

```
$ jekyll serve
```

## Deploy

Simply commit to main branch and push to GitHub. GitHub has adapters for building and deploying.
