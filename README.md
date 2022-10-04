# Gareth Brown Web Log 

https://garethbrown.net (garethrbrown.github.io)

Static site (Jekyll) for GitHub pages. 

## Development Environment

### Using GitHub Codespaces

This blog is maintained using GitHub Codespaces. To replicate this block workflow, create a new GitHub codespace for the repository.

Configure the codespace with the command:

`> Codespaces: Configure Development Container Features`

`devcontainer.json` is set up running Ubuntu with the Ruby feature:

```json
"features": {
    "ruby": "3.1"
}
```

Following codespace container build / rebuild. Jekyll needs to be installed:

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

In case of error like `You have already activated public_suffix 5.0.0, but your Gemfile requires public_suffix 4.0.7. Prepending ``bundle exec`` to your command may solve this.`, run:

```
$ bundle update
```

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

## Notes on Local Install

Due to issues running `gem install jekyll bundler`, ruby and Gem were installed with the following command. However later on, running `bundle install` fails, even with `bundle update` workaround.

```bash
$ sudo apt-get install ruby-full
$ sudo apt install jekyll
```