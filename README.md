# Gareth Brown Web Log 

https://garethbrown.net (garethrbrown.github.io)

Static site (Jekyll) for GitHub pages. 

## Development Environment

### Using GitHub CodeSpaces

This blog is maintained using GitHub Codespaces. To replicate this block workflow, create a new GitHub workspace for the repository.

Configure the workspace with the command:

`> Codespaces: Configure Development Container Features`

`devcontainer.json` is set up running Ubuntu with the Ruby feature:

```json
"features": {
    "ruby": "3.1"
}
```

Following workspace container build / rebuild. Jekyll needs to be installed:

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

To create a new blog template, run:

```
$ jekyll new <path>
```

## Debug

To debug locally run:

```
$ jekyll serve
```

## Deploy

Simply commit to main branch and push to GitHub. GitHub has adapters for building and deploying.

## Git Configuration

Merge strategy configured (defaults to merge):

```
 git config pull.rebase false 
```