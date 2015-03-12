---
title: Heroku Deployment
layout: en
permalink: /user/deployment/heroku/
---

Travis CI can automatically deploy your [Heroku](https://www.heroku.com/) application after a successful build.

For a minimal configuration, all you need to do is add the following to your `.travis.yml`:

    deploy:
      provider: heroku
      api_key: "YOUR API KEY"

You can retrieve your api key by running `heroku auth:token`. It is recommended to encrypt that key.
Assuming you have the Heroku and Travis CI command line clients installed, you can do it like this:

    travis encrypt $(heroku auth:token) --add deploy.api_key

You can also have the `travis` tool set up everything for you:

    $ travis setup heroku

Keep in mind that the above command has to run in your project directory, so it can modify the `.travis.yml` for you.

### Application to deploy

By default, we will try to deploy to an application by the same name as the repository. For example, if you deploy an application from the GitHub repository [travis-ci/travis-chat](https://github.com/travis-ci/travis-chat) without explicitly specify the name of the application, Travis CI will try to deploy to a Heroku app named *travis-chat*.

You can explicitly set the name via the **app** option:

    deploy:
      provider: heroku
      api_key: ...
      app: my-app-123

It is also possible to deploy different branches to different applications:

    deploy:
      provider: heroku
      api_key: ...
      app:
        master: my-app-staging
        production: my-app-production

If these apps belong to different Heroku accounts, you will have to do the same for the API key:

    deploy:
      provider: heroku
      api_key:
        master: ...
        production: ...
      app:
        master: my-app-staging
        production: my-app-production

### Branch to deploy from

If you have branch specific options, as [shown above](#application-to-deploy), Travis CI will automatically figure out which branches to deploy from. Otherwise, it will only deploy from your **master** branch.

You can also explicitly specify the branch to deploy from with the **on** option:

    deploy:
      provider: heroku
      api_key: ...
      on: production

Alternatively, you can also configure it to deploy from all branches:

    deploy:
      provider: heroku
      api_key: ...
      on:
        all_branches: true

Builds triggered from Pull Requests will never trigger a deploy.

### Running commands

In some setups, you might want to run a command on Heroku after a successful deploy. You can do this with the **run** option:

    deploy:
      provider: heroku
      api_key: ...
      run: "rake db:migrate"

It also accepts a list of commands:

    deploy:
      provider: heroku
      api_key: ...
      run:
        - "rake db:migrate"
        - "rake cleanup"

#### Restarting

Sometimes you want to restart your Heroku application between or after commands. You can easily do so by adding a "restart" command:

    deploy:
      provider: heroku
      api_key: ...
      run:
        - "rake db:migrate"
        - restart
        - "rake cleanup"

### Deploying build artifacts

After your tests ran and before the deploy, Travis CI will clean up any additional files and changes you made.

Maybe that is not what you want, as you might generate some artifacts (think asset compilation) that are supposed to be deployed, too. There is now an option to skip the clean up:

    deploy:
      provider: heroku
      api_key: ...
      skip_cleanup: true

{% include conditional_deploy.html provider="heroku" %}

### Buildpack

When deploying via the Anvil strategy (as described [below](#deploy-strategy)), you can now set the [buildpack](https://devcenter.heroku.com/articles/buildpacks) to use:

    deploy:
      provider: heroku
      buildpack: ruby

You can either use a shorthand for the [default buildpacks](https://devcenter.heroku.com/articles/buildpacks#default-buildpacks), like `ruby` or `nodejs` or give it the full URL for a [custom buildpack](https://devcenter.heroku.com/articles/buildpacks#using-a-custom-buildpack).

### Deploy Strategy

Travis CI supports different mechanisms for deploying to Heroku:

* **api:** Uses Heroku's [Build API](https://devcenter.heroku.com/articles/build-and-release-using-the-api). This is the default strategy.
* **anvil:** Uses an [unofficial build server](https://github.com/ddollar/anvil), which accepts archives of the application you want to deploy.
* **git:** Does a `git push` over HTTPS.
* **git-ssh:** Does a `git push` over SSH. This will generate a new key on every deployment.
* **git-deploy-key:** Does a `git push` over SSH. It will reuse the same key on every deployment. This is only available for private projects.

It defaults to **api**, but you can change that via the **strategy** option:

    deploy:
      provider: heroku
      api_key: ...
      strategy: git

Note that the **anvil**, **git-ssh** and **git-deploy-key** strategies are considered **deprecated**. Please contact us if you have issues switching away from these.

### Running commands before and after deploy

Sometimes you want to run commands before or after deploying. You can use the `before_deploy` and `after_deploy` stages for this. These will only be triggered if Travis CI is actually deploying.

    before_deploy: "echo 'ready?'"
    deploy:
      ..
    after_deploy:
      - ./after_deploy_1.sh
      - ./after_deploy_2.sh
