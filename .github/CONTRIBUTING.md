# How to contribute

First of all, thank you for taking the time to contribute to this project. We've tried to make a stable project and try to fix bugs and add new features continuously. You can help us do more.

## :innocent: Code of Conduct

This project and everyone participating in it is governed by the [Code of Conduct](CODE_OF_CONDUCT.md). By participating, you are expected to uphold this code. Please report unacceptable behavior to any member of our **administration team**.

## :octocat: How to getting started

### Check out the roadmap

We have some [functionalities in mind](ROADMAP.md) and we have issued them and there is a *milestone* label available on the issue. If there is a bug or a feature that is not listed in the **issues** page or there is no one assigned to the issue, feel free to fix/add it! Although it's better to discuss it in the issue or create a new issue for it so there is no confilcting code.

### Writing code

Contributing to a project on Github is pretty straight forward. If this is you're first time, these are the steps you should take.

1. Fork the repository.
1. Modify the source; please focus on the specific change you are contributing. If you also reformat all the code, it will be hard for us to focus on your change.
1. Commit to your fork using clear [commit messages](#git-commit-messages).
1. Send us a pull request, answering any default questions in the pull request interface.
1. Pay attention to any automated CI failures reported in the pull request, and stay involved in the conversation.

That's it! Read the code available and change the part you don't like! You're change should not break the existing code and should pass the tests.

If you're adding a new functionality, start from the branch **main**. It would be a better practice to create a new branch and work in there.

When you're done, submit a pull request and for one of the maintainers to check it out. We would let you know if there is any problem or any changes that should be considered.

> GitHub provides additional document on [forking a repository](https://help.github.com/articles/fork-a-repo/) and [creating a pull request](https://help.github.com/articles/creating-a-pull-request/).

### Documentation

Every chunk of code that may be hard to understand has some comments above it. If you write some new code or change some part of the existing code in a way that it would not be functional without changing it's usages, it needs to be documented.

### Code conventions

- [Python](https://google.github.io/styleguide/pyguide.html)
- [Shell](https://google.github.io/styleguide/shellguide.html)
- [Markdown](https://daringfireball.net/projects/markdown)
- [Docker](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

### Git Commit Messages

- Use the present tense ("Add feature" not "Added feature")
- Use the imperative mood ("Move cursor to..." not "Moves cursor to...")
- Limit the first line to 72 characters or less
- Always reference issues and pull requests under commit message description
- Consider starting the commit message with an applicable emoji:
  - :art: `:art:` when improving the format/structure of the code
  - :racehorse: `:racehorse:` when improving performance
  - :memo: `:memo:` when writing docs
  - :bug: `:bug:` when fixing a bug
  - :fire: `:fire:` when removing code or files
  - :green_heart: `:green_heart:` when fixing the CI build
  - :white_check_mark: `:white_check_mark:` when adding tests
  - :lock: `:lock:` when dealing with security
  - :arrow_up: `:arrow_up:` when upgrading dependencies
  - :arrow_down: `:arrow_down:` when downgrading dependencies
  - :shirt: `:shirt:` when removing linter warnings

## Environments

Please check the [`README.md`](../README.md) where you have all the informations about how to configure your project environment and start to code :nerd_face:

## Licensing

See the [LICENSE](../LICENSE) file for our project's licensing.

We may ask you to sign a [Contributor License Agreement (CLA)](https://en.wikipedia.org/wiki/Contributor_License_Agreement) for larger changes.

## Attribution

This document is adapted from:

- <https://mozillascience.github.io/working-open-workshop/github_for_collaboration>
- <https://github.com/atom/atom/blob/master/CONTRIBUTING.md>
- <https://gist.github.com/mpourismaiel/6a9eb6c69b5357d8bcc0>
