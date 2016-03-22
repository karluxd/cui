#Courselets Chat UI

- Demo and user experience
- Installing dependencies and building
- A quick test
- Data flow
  - Message types
  - Input types
  - Selectables
  - Actions
- Project structure
  - index.html
  - js/app.js
  - js/cui.js
  - js/helpers.js
  - js/models
  - js/presenters
  - js/views
  - js/plugins & bower_components/
- API endpoints and responses
  - GET /history
  - GET /messages
  - PUT /messages/:id
  - GET /progress
- Example markup and snippets
- Error handling
- Browser support
- External dependencies
- Future improvements



## Demo and user experience
Test the demo [here](http://s205922.gridserver.com/courselets/probability) to get an idea of how the chat works and how students submit answers, self-assess, and get extra lessons based on their performance. More information about the features and user experience can be found  [here](https://docs.google.com/document/d/1E-gqJMT0LaBFLXuvmSFVSvfKnpoREF1xN2vOpZThyyQ/#heading=h.ovvz3e7krlkg) and JSDocs are [here](http://s205922.gridserver.com/courselets/docs/).



## Installing dependencies and building
Clone the project and install [npm](https://docs.npmjs.com/getting-started/installing-node), [bower](http://bower.io/#install-bower), and gulp `$ npm install gulp -g`.

Go to the project directory and run `$ npm install` followed by `$ bower install`. That's about it.

`gulpfile.js` holds watch and build tasks. Run `$ gulp watch` to watch for changes and automatically compile Sass and Handlebars templates. `$ gulp build` builds the project to the dist directory. Edit the files in the src folder and push dist to production. The docs folder holds auto-generated documentation.



## A quick test
The src folder comes with basic static json files to test with until the API endpoints are ready. `index.html` outputs config values that point to these files so that you can test the UI right away. Replace these config values with the API endpoints when they are done.



## Data flow
The data flow in the chat is fairly simple.
- 1. We start with loading history data that returns the user's history or the first set of messages. It includes an input object that sets the user's input settings (text or predefined options) and a url for us to send input to.
- 2. The user posts input to this url and gets a new url in return where the next set of messages will be fetched, or it will return an error message that is displayed as a notification.
- 3. If we did not get an error in the previous step, we call the returned url to fetch the next set of messages, input settings, and the input url. The user posts input to the new url and this loop continues until the user reaches the end of the courselet.
- 4. A custom input option is displayed at the end and it usually holds a button linking to a different page.

Task flow: https://drive.google.com/open?id=0B9iTo36lSJEjQkNObzdVVVRCM0E

### Message types
There are three message types that can be displayed in the chat:
- **Message**. The default option that can hold any type of content but it is generally formatted text.
- **Media**. These hold media like images, video, or audio. They have two additional properties: `thumbnail` and `caption`.
- **Breakpoint**. Breakpoints are simple dividers with titles to represent the start of a new lesson in the chat.

### Input types
There are three different types of input:
- **Text**. The user enters free text in a textarea.
- **Options**. The user can select from one or more options.
- **Custom**. Custom HTML is displayed in the input area. This is generally only used at the end of a courselet.

### Selectables
Some input options are too large or complex to display in the input area. Selectables allow us to embed selectable elements within messages and pass them on with the input. They are generally used together with an input option to submit the selection. We use selectables to display error models when self-assessing.

### Actions
Actions are unique in that they can get called at any time, for any message, and independently from the current input option or url. Actions can be added to an overflow menu for a message and a click will fetch new messages and input settings from the attached url. Actions can be used for features like help where we load more information about a concept.



## Project structure

### index.html
Contains markup for the introduction and the chat UI. It contains a script tag where config values (history url etc) can be outputted, and it loads all required css and javascript files.

### js/app.js
Handles page preloading and binds event listeners for starting the chat.

### js/cui.js
Defines the CUI namespace and this is where generic messages in the chat can be edited.

### js/helpers.js
Contains polyfills and a last resort for catching errors. It currently shows a friendly notification to the user and logs the error to the console. We should log these errors with ajax as well to get notified when something goes wrong.

### js/models
This folder holds models for entities generated by the chat.

### js/presenters
Contains controllers that manages the user interface and they are often linked to a model. `js/presenters/cui.ChatPresenter` is the workhorse of the application that handles most DOM events and communication with the API.

#### js/views
Handlebars templates that contains the markup used when generating new elements.

### js/plugins & bower_components/
Libraries and plugins used by the presenters.



## API endpoints and responses
While the API is in development, these are the endpoints the chat UI needs to function.

### GET /history
Returns a user’s history of messages in the courselet, the next messages to display, and the input settings.
- [GET History](https://gist.github.com/karlwho/48ce4c2d461b8550b117)

### GET /messages
Returns a user’s next set of messages and input settings. It can also return arrays of messages to be updated or deleted.
- [GET Messages input.type:text](https://gist.github.com/karlwho/cde1722b96d766dce490)
- [GET Messages input.type:options](https://gist.github.com/karlwho/72db63f44a189b9b14d5)
- [GET Messages input.type:custom](https://gist.github.com/karlwho/29246a31fe41a71c1f98)

### PUT /messages/:id
Validates and saves user input. It returns the url for fetching the next set of messages or an error message.

**Arguments**
```
{
  "text": "Submitted text",
  "option": "A submitted option",
  "selected": {
    // Grouped by message ID
    "127": {
      "errorModel": ["2", "4"]
    }
  }
}
```

- [PUT Messages success](https://gist.github.com/karlwho/f068f1b028d9003c6c49)
- [PUT Messages error](https://gist.github.com/karlwho/0fc96fc3e14cac81eba6)

### GET /progress
Returns a user’s progress in the courselet and the list of lessons.
- [GET Progress](https://gist.github.com/karlwho/6026acefffa7082f1725)



## Example markup and snippets
[This gist](https://gist.github.com/karlwho/360a418324cdb27c84bd) contains example HTML markup. You can view the json for these snippets in `src/json/history.json` and see the output by loading it in the chat or visit [this link](http://s205922.gridserver.com/courselets/content/).



## Error handling
The chat leaves validation to the `PUT /messages/:id` endpoint. Returned error messages will be displayed as notifications to the user. If an error occurs due to network problems, misformatted data, etc., we recover if possible and ask the user to try again. We ask the user to refresh the page and try again later if we can not recover. Errors are logged to the console but it would be great to log these errors with ajax as well in helpers.js.



## Browser support
The chat has been tested in modern browsers (Chrome, Safari, Firefox, Opera, IE10+) and most features may work in legacy browsers. A message asks users to upgrade when using a legacy browser.



## External dependencies
I host the fonts on my paid http://www.fonts.com/ account.



## Future improvements
On top of my list for future improvements:
- Log JavaScript errors with ajax in js/helpers.js.
- Responsive design and support for mobile devices.
- Scrollable sidebar with a custom styled scrollbar.
- Increase the padding in the chat when the textarea grows so that you can read all messages while writing long messages.
- Add support for drawing.
- Make the user interface more accessible to screen readers and other assistive technology by implementing ARIA live regions https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Live_Regions.
- Better error handling for things like network problems.
- Porting the codebase to a framework like Ember as the project grows and needs more advanced features, powerful data models, url routes, etc. It should be easy to map the existing functionality to most JavaScript frameworks.
