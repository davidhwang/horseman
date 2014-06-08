# Dogfooding

In order for Drupal to be able to sustain both a decoupled frontend and an in-Drupal frontend, the way data gets sent and used by the presentation layer needs to be changed. In Drupal today, we have a tendency to conflate our data and our controllers which makes it an absolute mess for front end developers to reliably get the output they'd like without at least intermediate PHP skills and a full PHP debugger. While this doesn't sound unreasonable to the back end development community who already have these skills and tools, it's a large barrier to entry for those coming from any other front-end communities where their only required skills are HTML, CSS, JavaScript, and being able to read REST API documentation. In order to both attract new front end developers to Drupal and enable Drupal to power the webapps of the future, we need to both [simplify and ease](https://austin2014.drupal.org/session/stomp-complexity) working with our data models, aligning the interaction model for each.

## RESTful Data Model

The first step towards creating a system where a Drupal generated frontend and an API driven frontend is to make both of them driven by an identical, presentation independent data model. Our current data model is a fucking mess, from render arrays to magically passed around variables (better known as the hook system) to infinitely recursive structures, it's almost a miracle that frontend developers can produce *anything* with Drupal. Sometimes the template we want to use is in one place, sometimes it's in another. There are incorporeal additions to our data structures that manifest within unknown parts of the stack and more often than not are tied directly to HTML presentation. **This Shit Needs To Stop**.

Instead, our data models should adhere to the following guidelines:

* The only data that should be present in the data model should be raw data ready for filtering and the presentation layer.
* The data should be presentation (and thus context) independent. Context should be controlled by controllers
* Data should be stored in `key:value` pairs, not large arrays.
* Data models should contain no recursive structures
* The data model should follow the principles and constraints of [REST](http://en.wikipedia.org/wiki/Representational_state_transfer), allowing both headed and headless Drupal to absolutely share the same model without needing special case either.
* Alters should be exchanged for full replacements. To [quote Sam Boyer](https://twitter.com/sdboyer/status/475733416785510400), "You only get one good alter. After [the] first alter, [the] state is unknowable." This leads to a great deal of the headache involved in actually debugging and using the data model.