# Contributing to craft-master

We'd love to have your help improving craft-master! If you'd like to pitch in, you can do so in a number of ways:

1. Look through open [Issues](https://github.com/vigetlabs/craft-master/issues).
2. Review any open [Pull Requests](https://github.com/vigetlabs/craft-master/pulls).
3. [Fork craft-master](#getting-set-up-to-contribute) and fix an open Issue or add your own feature.
4. File new Issues if you have a good idea or see a bug and don't know how to fix it yourself. _Only do this after you've made sure the behavior or problem you're seeing isn't already documented in an open Issue._

We absolutely appreciate your interest in and help improving craft-master. We can't promise we'll accept every new feature or merge every pull request, but we'll do our best to give thoughtful consideration and time to everything you contribute.


## Getting set up to contribute

Contributing to craft-master is pretty straightforward:

1. Fork the craft-master repo and clone it.
2. Create a feature branch for the issue or new feature you're looking to tackle: `git checkout -b your-descriptive-branch-name`.
3. Commit your changes: `git commit -am 'Add some new feature or fix some issue'`.
4. Push the branch to your fork of craft-master: `git push origin your-descriptive-branch-name`.
5. Create a new Pull Request and we'll give it a look!


## Code Style

Everyone's got their own spin on code styling, so here's a rough outline of craft-master's approach:

- Since we're dealing with several languages, here are the whitespace rules:
	- Ruby and YAML files: 2 spaces
	- PHP and Markdown files: 1 hard tab
- No trailing whitespace and blank lines should have whitespace removed.
- Prefer single quotes over double quotes unless interpolating.
- Follow the conventions you see in the existing source code as best as you can.

Your bug fix or feature addition won't be rejected if it runs afoul of any (or all) of these guidelines, but it will definitely make everyone's lives a little easier.