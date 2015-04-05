# Quality Assurance (QA)

In web development, quality can be broken down into 4 dimensions:

1. Reach (SEO)

	Market penetration.

2. Functionality

	A site or service that works as advertised.

3. Usability

	Human-computer interaction.


4. Aesthetics

	Subjective.

## Overview of QA Techniques

1. Page Testing

	Tests the presentation and frontend functionality of a page. We'll be using [Mocha](http://mochajs.org/) for this.

2. Cross-page Testing

	Involves testing functionality that requires navigation from one pag to another. Since this kind of testing involves more than one component, it's generally considered integration testing. We'll be using [zombie.js](http://zombie.labnotes.org/) for this.

	Run tests:

```
	mocha -u tdd -R spec TEST_FILE.JS
```

`-u tdd` specifies that our interface is TDD (it defaults to BDD).

`-R spec` uses a reporter called the spec reporter which provides a little more information than the default reporter.

3. Logic Testing

	Disconnected from any presentation functionality. Unit and integration tests that test domain logic.

4. Linting

	Linting isn't about finding errors but about potential errors. The general concept of linting is that it identifies areas that could represent possible errors, or fragile constructs that could lead to errors in the future. We'll be using [JSHint](http://jshint.com/) for this.

5. Link Checking

	Making sure there are no broken links on the site. We'll be using [LinkChecker](http://wummel.github.io/linkchecker/) for this.

