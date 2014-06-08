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

## Unmushed Controllers

Drupal right now has a tendency to mush controllers and models together into a hybrid much that's hard to deal with. It often leads to annoying and hard to debug instances where HTML of some kinds is being directly into a page without a proper separation of concerns. In order to resolve this, and to enforce the best practices of separation of presentation and content in modules that's needed for a headless state, anything that gets printed to a page is required to have a controller that is separate from the data model. This controller's sole purpose is to decide what data is available, and what view is being used. It itself doesn't hold content, isn't in charge of manipulating content, only determining what content is available to the view of the content, and how to update the model's state (*i.e.* form entry, AJAX requests, etc…).

In Drupal, the most logical concept to become a controller is the `block`. Instead of having view modes for entities and blocks as content and some blocks that aren't content but everything winds up getting displayed as blocks anyway, simply make the Block the single place to store all interactions between a model and a view. Then, larger blocks can be made up of smaller blocks with any content available in that block, without jumping through hoops to get it there.

Practically speaking, this would be that Views (and other like tools), instead of being a full MVC stack on their own, would exist to solely generate different data streams (that would work equally well for both headed and headless Drupal), and a block would be created separately to display that data stream, potentially in a variety of different ways.