# Chapter 4

Version numbers in npm are parsed by a component called `semver`
(semantic versioning)

## Modules

If we want code to be visible outside of a module, the global variable `exports`
is used to allow us to import that code anywhere else in our project.

Example:

fortune.js

    exports.getFortune = function() {
      // Do stuff here
    }

meadowlar.js

    var fortune = require('./fortune.js')

Note the `./` in the require above. This signals to node that it shouldn't look
in the node_modules directory.