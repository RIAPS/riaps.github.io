# Contributing to RIAPS

The following is a set of guidelines for contributing to RIAPS whose code base is hosted in the
[RIAPS Organization](https://github.com/RIAPS) on GitHub. These are mostly guidelines, not rules.
Use your best judgment, and feel free to propose changes to this document in a pull request.

#### Table Of Contents

[Code of Conduct](#code-of-conduct)

[License and developer Certificate of Origin](#license-and-developer-certificate-of-origin)

[How Can I Contribute?](#how-can-i-contribute)
  * [Reporting Bugs and Suggesting Enhancements](#reporting-bugs-and-suggesting-enhancements)
  * [Contributing Code](#contributing-code)
  * [Tools for contributions](#tools-for-contributions)

[Styleguides](#styleguides)
  * [Git Commit Messages](#git-commit-messages)

[Project Governance](#project-governance)
  * [Project Owner](#project-owner)
  * [Committers](#committers)
  * [Technical Steering Committee](#technical-steering-committee)
  * [Contributors](#contributors)

[Roadmap](#roadmap)
  * [Documentation](#documentation)

Please note we have a code of conduct, please follow it in all your interactions with the project.

# <a name="code-of-conduct">Code of Conduct</a>

This project and everyone participating in it is governed by the [RIAPS Code of Conduct](CODE_OF_CONDUCT.md).
By participating, you are expected to uphold this code. Please report unacceptable behavior
to [riaps+owner@lists.lfenergy.org](mailto:riaps+owner@lists.lfenergy.org).


## <a name="license-and-developer-certificate-of-origin">License and Developer Certificate of Origin</a>

RIAPS is an open source framework licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0).
By contributing to RIAPS, you accept and agree to the terms and conditions for your present and future contributions
submitted to RIAPS.

The project also uses a mechanism known as a [Developer Certificate of Origin (DCO)](https://developercertificate.org/)
to manage the process of ensuring that we are legally allowed to distribute all the code and assets for the project.
A DCO is a legally binding statement that asserts that you are the creator of your contribution, and that you wish to
allow RIAPS to use your work.

Contributors sign-off that they adhere to these requirements by adding a ```Signed-off-by``` line to commit messages.
All commits of all repositories of the RIAPS community have to be signed-off like this :

```
This is my commit message.

Signed-off-by: John Doe <john.doe@github.com>
```
You can write it manually but Git even has a -s command line option to append this automatically to your commit message:
```
$ git commit -s -m 'This is my commit message'
```

Note that a check will be performed during the integration, indicating whether or not commits in a
Pull Request do not contain a valid ```Signed-off-by``` line.

The ```Signed-off-by``` line at the end of the commit message can be written automatically.
You just have to configure the ``` .gitconfig``` file with the following alias :
```
[alias]
    ci = commit -s
```
and instead of
```
$ git commit -m "This is my message"
```
you just have to write
```
$ git ci -m "This is my message"
```
Note that most IDEs can be configured to add a ```Signed-off-by``` line at the end of the
commit message (Intellij IDEA and Eclipse IDE for example, do not hesitate to ask us).

## <a name="how-can-i-contribute">How Can I Contribute?</a>

### <a name="reporting-bugs-and-suggesting-enhancements">Reporting Bugs and Suggesting Enhancements</a>

Bugs and enhancement suggestions are tracked as [GitHub issues](https://guides.github.com/features/issues/).
Create an issue and provide the following information by filling in [the template](ISSUE_TEMPLATE.md).

Before creating bug reports or suggesting enhancement, please
**perform a [cursory search](https://github.com/search?q=+is%3Aissue+user%3ARIAPS)** to see if the problem has already
been reported. If it has **and the issue is still open**, add a comment to the existing issue instead of opening a new one.

You can also contact the team directly to talk about your ideas at [riaps@lists.lfenergy.org](mailto:riaps@lists.lfenergy.org).

> **Note:** If you find a **Closed** issue that seems like it is the same thing that you're experiencing, open a new issue and include a link to the original issue in the body of your new one.

### <a name="contributing-code">Contributing Code</a>

Code Contribution is tracked as [GitHub Pull Requests](https://help.github.com/en/articles/about-pull-requests).
Crafting a good pull request takes time and energy and we will help as much as we can, but be prepared to follow
our iterative process. The iterative process has several goals:

- Maintain RIAPS's quality
- Fix problems that are important to users
- Engage the community in working toward the best possible RIAPS
- Enable a sustainable system for RIAPS's maintainers to review contributions

Please follow these steps to have your contribution considered by the maintainers:

1. Follow all instructions in [the template](PULL_REQUEST_TEMPLATE.md)
2. Follow the [styleguides](#styleguides)
3. After you submit your pull request, verify that all
[status checks](https://help.github.com/articles/about-status-checks/) are passing.
5. Request a GitHub review by one of the core developers (e.g. @adubey14 @imadari @MMetelko)
6. Follow their instructions or discuss about the requested changes. Please don't take criticism personally,
it is normal to iterate on this step several times.
7. Repeat step 6 until the pull request is merged !

Continuous integration is setup to run on all branches automatically and will often report problems,
so don't worry about getting everything perfect on the first try.
Until you add a reviewer, you can trigger as many builds as you want by amending your commits.

### <a name="tools-for-contributions">Tools for contributions</a>

Continuous integration is setup automatically on all contributions. However, it's faster to iterate locally to fix
problems than waiting for the status checks to finish. There are many tools that can be used to do the verifications
that are enforced by all status checks. The most simple and universal tool is maven, but IDE integrations can be
used to get more immediate feedback. Most of the team uses Eclipse, but others IDEs can be used, for example
Visual Studio Code.

## <a name="styleguides">Styleguides</a>

### <a name="git-commit-messages">Git Commit Messages</a>

As usual, please start the commit message with a short line describing the commit, then leave a blank line,
then give more context and explanations. You can use GitHub's integrations, for example to link to existing issues.
In general, pull requests with more than one commits will be squashed when merged in master.

## <a name="project-governance">Project Governance</a>

#### <a name="project-owner">Project Owner</a>

RIAPS is part of the LF Energy Foundation, a project of The Linux Foundation that supports open source innovation
projects within the energy and electricity sectors.

#### <a name="committers">Committers</a>

Committers are contributors who have made several valuable contributions to the project and are now relied upon to
both write code directly to the repository and screen the contributions of others. In many cases they are programmers,
but it is also possible that they contribute in a different role. Typically, a committer will focus on a specific aspect
of the project, and will bring a level of expertise and understanding that earns them the respect of the community and
the project owner.

#### <a name="technical-steering-committee">Technical Steering Committee</a>

The Technical Steering Committee (TSC) is composed of voting members elected by the active Committers as described in
the project’s Technical Charter.

RIAPS TSC voting members are:
- Abhishek Dubey (https://github.com/adubey14)
- Gabor Karsai (https://github.com/gkarsai)
- Istvan Madari (https://github.com/imadari)
- Mary Metelko (https://github.com/MMetelko)
- Peter Volgyesi (https://github.com/volgy)


Some committers are specialized in some field: please refer to [the maintainers table](MAINTAINERS.md) before
submitting a pull request.

#### <a name="contributors">Contributors</a>

Contributors include anyone in the technical community that contributes code, documentation, or other technical
artifacts to the Project.

Anyone can become a contributor. There is no expectation of commitment to the project, no specific skill requirements
and no selection process. To become a contributor, a community member simply has to perform one or more actions that are beneficial to the project.

## <a name="roadmap">Roadmap</a>

### <a name="documentation">Documentation</a>
- Functional documentation;
- User stories;
- More and more tutorials;
- Even more demo applications.
