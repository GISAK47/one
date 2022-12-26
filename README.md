# one
<Description DefaultValue="Allows users to access their GitHub gists."/>
Test the generated add-in
Before going any further, let's test the basic add-in that the generator created to confirm that the project is set up correctly.

 Note

Office Add-ins should use HTTPS, not HTTP, even when you are developing. If you are prompted to install a certificate after you run one of the following commands, accept the prompt to install the certificate that the Yeoman generator provides. You may also have to run your command prompt or terminal as an administrator for the changes to be made.

Run the following command in the root directory of your project. When you run this command, the local web server starts and your add-in will be sideloaded.

command line

Copy
npm start
In Outlook, open an existing message and select the Show Taskpane button.

When prompted with the WebView Stop On Load dialog box, select OK.

 Note

If you select Cancel, the dialog won't be shown again while this instance of the add-in is running. However, if you restart your add-in, you'll see the dialog again.

If everything's been set up correctly, the task pane will open and render the add-in's welcome page.

The Show Taskpane button and Git the gist task pane added by the sample.

Define buttons
Now that you've verified the base add-in works, you can customize it to add more functionality. By default, the manifest only defines buttons for the read message window. Let's update the manifest to remove the buttons from the read message window and define two new buttons for the compose message window:

Insert gist: a button that opens a task pane

Insert default gist: a button that invokes a function

Remove the MessageReadCommandSurface extension point
Open the manifest.xml file and locate the <ExtensionPoint> element with type MessageReadCommandSurface. Delete this <ExtensionPoint> element (including its closing tag) to remove the buttons from the read message window.

Add the MessageComposeCommandSurface extension point
Locate the line in the manifest that reads </DesktopFormFactor>. Immediately before this line, insert the following XML markup. Note the following about this markup.

The <ExtensionPoint> with xsi:type="MessageComposeCommandSurface" indicates that you're defining buttons to add to the compose message window.

By using an <OfficeTab> element with id="TabDefault", you're indicating you want to add the buttons to the default tab on the ribbon.

The <Group> element defines the grouping for the new buttons, with a label set by the groupLabel resource.

The first <Control> element contains an <Action> element with xsi:type="ShowTaskPane", so this button opens a task pane.

The second <Control> element contains an <Action> element with xsi:type="ExecuteFunction", so this button invokes a JavaScript function contained in the function file.

XML

Copy
<!-- Message Compose -->
<ExtensionPoint xsi:type="MessageComposeCommandSurface">
  <OfficeTab id="TabDefault">
    <Group id="msgComposeCmdGroup">
      <Label resid="GroupLabel"/>
      <Control xsi:type="Button" id="msgComposeInsertGist">
        <Label resid="TaskpaneButton.Label"/>
        <Supertip>
          <Title resid="TaskpaneButton.Title"/>
          <Description resid="TaskpaneButton.Tooltip"/>
        </Supertip>
        <Icon>
          <bt:Image size="16" resid="Icon.16x16"/>
          <bt:Image size="32" resid="Icon.32x32"/>
          <bt:Image size="80" resid="Icon.80x80"/>
        </Icon>
        <Action xsi:type="ShowTaskpane">
          <SourceLocation resid="Taskpane.Url"/>
        </Action>
      </Control>
      <Control xsi:type="Button" id="msgComposeInsertDefaultGist">
        <Label resid="FunctionButton.Label"/>
        <Supertip>
          <Title resid="FunctionButton.Title"/>
          <Description resid="FunctionButton.Tooltip"/>
        </Supertip>
        <Icon>
          <bt:Image size="16" resid="Icon.16x16"/>
          <bt:Image size="32" resid="Icon.32x32"/>
          <bt:Image size="80" resid="Icon.80x80"/>
        </Icon>
        <Action xsi:type="ExecuteFunction">
          <FunctionName>insertDefaultGist</FunctionName>
        </Action>
      </Control>
    </Group>
  </OfficeTab>
</ExtensionPoint>
Update resources in the manifest
The previous code references labels, tooltips, and URLs that you need to define before the manifest will be valid. You'll specify this information in the <Resources> section of the manifest.

Locate the <Resources> element in the manifest file and delete the entire element (including its closing tag).

In that same location, add the following markup to replace the <Resources> element you just removed.

XML

Copy
<Resources>
  <bt:Images>
    <bt:Image id="Icon.16x16" DefaultValue="https://localhost:3000/assets/icon-16.png"/>
    <bt:Image id="Icon.32x32" DefaultValue="https://localhost:3000/assets/icon-32.png"/>
    <bt:Image id="Icon.80x80" DefaultValue="https://localhost:3000/assets/icon-80.png"/>
  </bt:Images>
  <bt:Urls>
    <bt:Url id="Commands.Url" DefaultValue="https://localhost:3000/commands.html"/>
    <bt:Url id="Taskpane.Url" DefaultValue="https://localhost:3000/taskpane.html"/>
  </bt:Urls>
  <bt:ShortStrings>
    <bt:String id="GroupLabel" DefaultValue="Git the gist"/>
    <bt:String id="TaskpaneButton.Label" DefaultValue="Insert gist"/>
    <bt:String id="TaskpaneButton.Title" DefaultValue="Insert gist"/>
    <bt:String id="FunctionButton.Label" DefaultValue="Insert default gist"/>
    <bt:String id="FunctionButton.Title" DefaultValue="Insert default gist"/>
  </bt:ShortStrings>
  <bt:LongStrings>
    <bt:String id="TaskpaneButton.Tooltip" DefaultValue="Displays a list of your gists and allows you to insert their contents into the current message."/>
    <bt:String id="FunctionButton.Tooltip" DefaultValue="Inserts the content of the gist you mark as default into the current message."/>
  </bt:LongStrings>
</Resources>
Save your changes to the manifest.

Reinstall the add-in
You must reinstall the add-in for the manifest changes to take effect.

If the web server is running, close the node command window.

Run the following command to start the local web server and automatically sideload your add-in.

command line

Copy
npm start
After you've reinstalled the add-in, you can verify that it installed successfully by checking for the commands Insert gist and Insert default gist in a compose message window. Note that nothing will happen if you select either of these items, because you haven't yet finished building this add-in.

If you're running this add-in in Outlook 2016 or later on Windows, you should see two new buttons in the ribbon of the compose message window: Insert gist and Insert default gist.

The ribbon overflow menu in Outlook on Windows with the add-in's buttons highlighted.

If you're running this add-in in Outlook on the web, you should see a new button at the bottom of the compose message window. Select that button to see the options Insert gist and Insert default gist.

The message compose form in Outlook on the web with the add-in button and pop-up menu highlighted.

Implement a first-run experience
This add-in needs to be able to read gists from the user's GitHub account and identify which one the user has chosen as the default gist. In order to achieve these goals, the add-in must prompt the user to provide their GitHub username and choose a default gist from their collection of existing gists. Complete the steps in this section to implement a first-run experience that will display a dialog to collect this information from the user.

Collect data from the user
Let's start by creating the UI for the dialog itself. Within the ./src folder, create a new subfolder named settings. In the ./src/settings folder, create a file named dialog.html, and add the following markup to define a basic form with a text input for a GitHub username and an empty list for gists that'll be populated via JavaScript.

HTML

Copy
<!DOCTYPE html>
<html>

<head>
  <meta charset="UTF-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=Edge" />
  <title>Settings</title>

  <!-- Office JavaScript API -->
  <script type="text/javascript" src="https://appsforoffice.microsoft.com/lib/1.1/hosted/office.js"></script>

<!-- For more information on Fluent UI, visit https://developer.microsoft.com/fluentui. -->
  <link rel="stylesheet" href="https://static2.sharepointonline.com/files/fabric/office-ui-fabric-core/9.6.1/css/fabric.min.css"/>

  <!-- Template styles -->
  <link href="dialog.css" rel="stylesheet" type="text/css" />
</head>

<body class="ms-font-l">
  <main>
    <section class="ms-font-m ms-fontColor-neutralPrimary">
      <div class="not-configured-warning ms-MessageBar ms-MessageBar--warning">
        <div class="ms-MessageBar-content">
          <div class="ms-MessageBar-icon">
            <i class="ms-Icon ms-Icon--Info"></i>
          </div>
          <div class="ms-MessageBar-text">
            Oops! It looks like you haven't configured <strong>Git the gist</strong> yet.
            <br/>
            Please configure your GitHub username and select a default gist, then try that action again!
          </div>
        </div>
      </div>
      <div class="ms-font-xxl">Settings</div>
      <div class="ms-Grid">
        <div class="ms-Grid-row">
          <div class="ms-TextField">
            <label class="ms-Label">GitHub Username</label>
            <input class="ms-TextField-field" id="github-user" type="text" value="" placeholder="Please enter your GitHub username">
          </div>
        </div>
        <div class="error-display ms-Grid-row">
          <div class="ms-font-l ms-fontWeight-semibold">An error occurred:</div>
          <pre><code id="error-text"></code></pre>
        </div>
        <div class="gist-list-container ms-Grid-row">
          <div class="list-title ms-font-xl ms-fontWeight-regular">Choose Default Gist</div>
          <form>
            <div id="gist-list">
            </div>
          </form>
        </div>
      </div>
      <div class="ms-Dialog-actions">
        <div class="ms-Dialog-actionsRight">
          <button class="ms-Dialog-action ms-Button ms-Button--primary" id="settings-done" disabled>
            <span class="ms-Button-label">Done</span>
          </button>
        </div>
      </div>
    </section>
  </main>
  <script type="text/javascript" src="../../node_modules/jquery/dist/jquery.js"></script>
  <script type="text/javascript" src="../helpers/gist-api.js"></script>
  <script type="text/javascript" src="dialog.js"></script>
</body>

</html>
You may have noticed that the HTML file references a JavaScript file, gist-api.js, that doesn't yet exist. This file will be created in the Fetch data from GitHub section below.

Next, create a file in the ./src/settings folder named dialog.css, and add the following code to specify the styles that are used by dialog.html.

CSS

Copy
section {
  margin: 10px 20px;
}

.not-configured-warning {
  display: none;
}

.error-display {
  display: none;
}

.gist-list-container {
  margin: 10px -8px;
  display: none;
}

.list-title {
  border-bottom: 1px solid #a6a6a6;
  padding-bottom: 5px;
}

ul {
  margin-top: 10px;
}

.ms-ListItem-secondaryText,
.ms-ListItem-tertiaryText {
  padding-left: 15px;
}
Now that you've defined the dialog UI, you can write the code that makes it actually do something. Create a file in the ./src/settings folder named dialog.js and add the following code. Note that this code uses jQuery to register events and uses the messageParent method to send the user's choices back to the caller.

JavaScript

Copy
(function(){
  'use strict';

  // The Office initialize function must be run each time a new page is loaded.
  Office.initialize = function(reason){
    jQuery(document).ready(function(){
      if (window.location.search) {
        // Check if warning should be displayed.
        const warn = getParameterByName('warn');
        if (warn) {
          $('.not-configured-warning').show();
        } else {
          // See if the config values were passed.
          // If so, pre-populate the values.
          const user = getParameterByName('gitHubUserName');
          const gistId = getParameterByName('defaultGistId');

          $('#github-user').val(user);
          loadGists(user, function(success){
            if (success) {
              $('.ms-ListItem').removeClass('is-selected');
              $('input').filter(function() {
                return this.value === gistId;
              }).addClass('is-selected').attr('checked', 'checked');
              $('#settings-done').removeAttr('disabled');
            }
          });
        }
      }

      // When the GitHub username changes,
      // try to load gists.
      $('#github-user').on('change', function(){
        $('#gist-list').empty();
        const ghUser = $('#github-user').val();
        if (ghUser.length > 0) {
          loadGists(ghUser);
        }
      });

      // When the Done button is selected, send the
      // values back to the caller as a serialized
      // object.
      $('#settings-done').on('click', function() {
        const settings = {};

        settings.gitHubUserName = $('#github-user').val();

        const selectedGist = $('.ms-ListItem.is-selected');
        if (selectedGist) {
          settings.defaultGistId = selectedGist.val();

          sendMessage(JSON.stringify(settings));
        }
      });
    });
  };

  // Load gists for the user using the GitHub API
  // and build the list.
  function loadGists(user, callback) {
    getUserGists(user, function(gists, error){
      if (error) {
        $('.gist-list-container').hide();
        $('#error-text').text(JSON.stringify(error, null, 2));
        $('.error-display').show();
        if (callback) callback(false);
      } else {
        $('.error-display').hide();
        buildGistList($('#gist-list'), gists, onGistSelected);
        $('.gist-list-container').show();
        if (callback) callback(true);
      }
    });
  }

  function onGistSelected() {
    $('.ms-ListItem').removeClass('is-selected').removeAttr('checked');
    $(this).children('.ms-ListItem').addClass('is-selected').attr('checked', 'checked');
    $('.not-configured-warning').hide();
    $('#settings-done').removeAttr('disabled');
  }

  function sendMessage(message) {
    Office.context.ui.messageParent(message);
  }

  function getParameterByName(name, url) {
    if (!url) {
      url = window.location.href;
    }
    name = name.replace(/[\[\]]/g, "\\$&");
    const regex = new RegExp("[?&]" + name + "(=([^&#]*)|&|#|$)"),
      results = regex.exec(url);
    if (!results) return null;
    if (!results[2]) return '';
    return decodeURIComponent(results[2].replace(/\+/g, " "));
  }
})();
Update webpack config settings
Finally, open the webpack.config.js file found in the root directory of the project and complete the following steps.

Locate the entry object within the config object and add a new entry for dialog.

JavaScript

Copy
dialog: "./src/settings/dialog.js",
After you've done this, the new entry object will look like this:

JavaScript

Copy
entry: {
  polyfill: ["core-js/stable", "regenerator-runtime/runtime"],
  taskpane: "./src/taskpane/taskpane.js",
  commands: "./src/commands/commands.js",
  dialog: "./src/settings/dialog.js",
},
Locate the plugins array within the config object. In the patterns array of the new CopyWebpackPlugin object, add new entries for taskpane.css and dialog.css.

JavaScript

Copy
{
  from: "./src/taskpane/taskpane.css",
  to: "taskpane.css",
},
{
  from: "./src/settings/dialog.css",
  to: "dialog.css",
},
After you've done this, the new CopyWebpackPlugin object will look like this:

JavaScript

Copy
new CopyWebpackPlugin({
  patterns: [
  {
    from: "./src/taskpane/taskpane.css",
    to: "taskpane.css",
  },
  {
    from: "./src/settings/dialog.css",
    to: "dialog.css",
  },
  {
    from: "assets/*",
    to: "assets/[name][ext][query]",
  },
  {
    from: "manifest*.xml",
    to: "[name]." + buildType + "[ext]",
    transform(content) {
      if (dev) {
        return content;
      } else {
        return content.toString().replace(new RegExp(urlDev, "g"), urlProd);
      }
    },
  },
]}),
In the same plugins array within the config object, add this new object to the end of the array.

JavaScript

Copy
new HtmlWebpackPlugin({
  filename: "dialog.html",
  template: "./src/settings/dialog.html",
  chunks: ["polyfill", "dialog"]
})
After you've done this, the new plugins array will look like this:

JavaScript

Copy
plugins: [
  new HtmlWebpackPlugin({
    filename: "taskpane.html",
    template: "./src/taskpane/taskpane.html",
    chunks: ["polyfill", "taskpane"],
  }),
  new CopyWebpackPlugin({
    patterns: [
      {
        from: "./src/taskpane/taskpane.css",
        to: "taskpane.css",
      },
      {
        from: "./src/settings/dialog.css",
        to: "dialog.css",
      },
      {
        from: "assets/*",
        to: "assets/[name][ext][query]",
      },
      {
        from: "manifest*.xml",
        to: "[name]." + buildType + "[ext]",
        transform(content) {
          if (dev) {
            return content;
          } else {
            return content.toString().replace(new RegExp(urlDev, "g"), urlProd);
          }
        },
      },
    ],
  }),
  new HtmlWebpackPlugin({
    filename: "commands.html",
    template: "./src/commands/commands.html",
    chunks: ["polyfill", "commands"],
  }),
  new HtmlWebpackPlugin({
    filename: "dialog.html",
    template: "./src/settings/dialog.html",
    chunks: ["polyfill", "dialog"]
  })
],
Fetch data from GitHub
The dialog.js file you just created specifies that the add-in should load gists when the change event fires for the GitHub username field. To retrieve the user's gists from GitHub, you'll use the GitHub Gists API.

Within the ./src folder, create a new subfolder named helpers. In the ./src/helpers folder, create a file named gist-api.js, and add the following code to retrieve the user's gists from GitHub and build the list of gists.

JavaScript

Copy
function getUserGists(user, callback) {
  const requestUrl = 'https://api.github.com/users/' + user + '/gists';

  $.ajax({
    url: requestUrl,
    dataType: 'json'
  }).done(function(gists){
    callback(gists);
  }).fail(function(error){
    callback(null, error);
  });
}

function buildGistList(parent, gists, clickFunc) {
  gists.forEach(function(gist) {

    const listItem = $('<div/>')
      .appendTo(parent);

    const radioItem = $('<input>')
      .addClass('ms-ListItem')
      .addClass('is-selectable')
      .attr('type', 'radio')
      .attr('name', 'gists')
      .attr('tabindex', 0)
      .val(gist.id)
      .appendTo(listItem);

    const descPrimary = $('<span/>')
      .addClass('ms-ListItem-primaryText')
      .text(gist.description)
      .appendTo(listItem);

    const descSecondary = $('<span/>')
      .addClass('ms-ListItem-secondaryText')
      .text(' - ' + buildFileList(gist.files))
      .appendTo(listItem);

    const updated = new Date(gist.updated_at);

    const descTertiary = $('<span/>')
      .addClass('ms-ListItem-tertiaryText')
      .text(' - Last updated ' + updated.toLocaleString())
      .appendTo(listItem);

    listItem.on('click', clickFunc);
  });  
}

function buildFileList(files) {

  let fileList = '';

  for (let file in files) {
    if (files.hasOwnProperty(file)) {
      if (fileList.length > 0) {
        fileList = fileList + ', ';
      }

      fileList = fileList + files[file].filename + ' (' + files[file].language + ')';
    }
  }

  return fileList;
}
Run the following command to rebuild the project.

command line

Copy
npm run build
Implement a UI-less button
This add-in's Insert default gist button is a UI-less button that will invoke a JavaScript function, rather than open a task pane like many add-in buttons do. When the user selects the Insert default gist button, the corresponding JavaScript function will check whether the add-in has been configured.

If the add-in has already been configured, the function will load the content of the gist that the user has selected as the default and insert it into the body of the message.

If the add-in hasn't yet been configured, then the settings dialog will prompt the user to provide the required information.

Update the function file (HTML)
A function that's invoked by a UI-less button must be defined in the file that's specified by the <FunctionFile> element in the manifest for the corresponding form factor. This add-in's manifest specifies https://localhost:3000/commands.html as the function file.

Open the file ./src/commands/commands.html and replace the entire contents with the following markup.

HTML

Copy
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=Edge" />

    <!-- Office JavaScript API -->
    <script type="text/javascript" src="https://appsforoffice.microsoft.com/lib/1.1/hosted/office.js"></script>

    <script type="text/javascript" src="../../node_modules/jquery/dist/jquery.js"></script>
    <script type="text/javascript" src="../../node_modules/showdown/dist/showdown.min.js"></script>
    <script type="text/javascript" src="../../node_modules/urijs/src/URI.min.js"></script>
    <script type="text/javascript" src="../helpers/addin-config.js"></script>
    <script type="text/javascript" src="../helpers/gist-api.js"></script>
</head>

<body>
  <!-- NOTE: The body is empty on purpose. Since functions in commands.js are
       invoked via a button, there is no UI to render. -->
</body>

</html>
You may have noticed that the HTML file references a JavaScript file, addin-config.js, that doesn't yet exist. This file will be created in the Create a file to manage configuration settings section later in this tutorial.

Update the function file (JavaScript)
Open the file ./src/commands/commands.js and replace the entire contents with the following code. Note that if the insertDefaultGist function determines the add-in has not yet been configured, it adds the ?warn=1 parameter to the dialog URL. Doing so makes the settings dialog render the message bar that's defined in ./src/settings/dialog.html, to tell the user why they're seeing the dialog.

JavaScript

Copy
let config;
let btnEvent;

// The initialize function must be run each time a new page is loaded.
Office.initialize = function () {
};

function showError(error) {
  Office.context.mailbox.item.notificationMessages.replaceAsync('github-error', {
    type: 'errorMessage',
    message: error
  }, function(result){
  });
}

let settingsDialog;

function insertDefaultGist(event) {

  config = getConfig();

  // Check if the add-in has been configured.
  if (config && config.defaultGistId) {
    // Get the default gist content and insert.
    try {
      getGist(config.defaultGistId, function(gist, error) {
        if (gist) {
          buildBodyContent(gist, function (content, error) {
            if (content) {
              Office.context.mailbox.item.body.setSelectedDataAsync(content,
                {coercionType: Office.CoercionType.Html}, function(result) {
                  event.completed();
              });
            } else {
              showError(error);
              event.completed();
            }
          });
        } else {
          showError(error);
          event.completed();
        }
      });
    } catch (err) {
      showError(err);
      event.completed();
    }

  } else {
    // Save the event object so we can finish up later.
    btnEvent = event;
    // Not configured yet, display settings dialog with
    // warn=1 to display warning.
    const url = new URI('dialog.html?warn=1').absoluteTo(window.location).toString();
    const dialogOptions = { width: 20, height: 40, displayInIframe: true };

    Office.context.ui.displayDialogAsync(url, dialogOptions, function(result) {
      settingsDialog = result.value;
      settingsDialog.addEventHandler(Office.EventType.DialogMessageReceived, receiveMessage);
      settingsDialog.addEventHandler(Office.EventType.DialogEventReceived, dialogClosed);
    });
  }
}

// Register the function.
Office.actions.associate("insertDefaultGist", insertDefaultGist);

function receiveMessage(message) {
  config = JSON.parse(message.message);
  setConfig(config, function(result) {
    settingsDialog.close();
    settingsDialog = null;
    btnEvent.completed();
    btnEvent = null;
  });
}

function dialogClosed(message) {
  settingsDialog = null;
  btnEvent.completed();
  btnEvent = null;
}
Create a file to manage configuration settings
The HTML function file references a file named addin-config.js, which doesn't yet exist. In the ./src/helpers folder, create a file named addin-config.js and add the following code. This code uses the RoamingSettings object to get and set configuration values.

JavaScript

Copy
function getConfig() {
  const config = {};

  config.gitHubUserName = Office.context.roamingSettings.get('gitHubUserName');
  config.defaultGistId = Office.context.roamingSettings.get('defaultGistId');

  return config;
}

function setConfig(config, callback) {
  Office.context.roamingSettings.set('gitHubUserName', config.gitHubUserName);
  Office.context.roamingSettings.set('defaultGistId', config.defaultGistId);

  Office.context.roamingSettings.saveAsync(callback);
}
Create new functions to process gists
Next, open the ./src/helpers/gist-api.js file and add the following functions. Note the following:

If the gist contains HTML, the add-in will insert the HTML as is into the body of the message.

If the gist contains Markdown, the add-in will use the Showdown library to convert the Markdown to HTML, and will then insert the resulting HTML into the body of the message.

If the gist contains anything other than HTML or Markdown, the add-in will insert it into the body of the message as a code snippet.

JavaScript

Copy
function getGist(gistId, callback) {
  const requestUrl = 'https://api.github.com/gists/' + gistId;

  $.ajax({
    url: requestUrl,
    dataType: 'json'
  }).done(function(gist){
    callback(gist);
  }).fail(function(error){
    callback(null, error);
  });
}

function buildBodyContent(gist, callback) {
  // Find the first non-truncated file in the gist
  // and use it.
  for (let filename in gist.files) {
    if (gist.files.hasOwnProperty(filename)) {
      const file = gist.files[filename];
      if (!file.truncated) {
        // We have a winner.
        switch (file.language) {
          case 'HTML':
            // Insert as is.
            callback(file.content);
            break;
          case 'Markdown':
            // Convert Markdown to HTML.
            const converter = new showdown.Converter();
            const html = converter.makeHtml(file.content);
            callback(html);
            break;
          default:
            // Insert contents as a <code> block.
            let codeBlock = '<pre><code>';
            codeBlock = codeBlock + file.content;
            codeBlock = codeBlock + '</code></pre>';
            callback(codeBlock);
        }
        return;
      }
    }
  }
  callback(null, 'No suitable file found in the gist');
}
Test the Insert default gist button
Save all of your changes and run npm start from the command prompt, if the server isn't already running. Then complete the following steps to test the Insert default gist button.

Open Outlook and compose a new message.

In the compose message window, select the Insert default gist button. You should see a dialog where you can configure the add-in, starting with the prompt to set your GitHub username.

The dialog prompt to configure the add-in.

In the settings dialog, enter your GitHub username and then either Tab or click elsewhere in the dialog to invoke the change event, which should load your list of public gists. Select a gist to be the default, and select Done.

The add-in's settings dialog.

Select the Insert default gist button again. This time, you should see the contents of the gist inserted into the body of the email.

 Note

Outlook on Windows: To pick up the latest settings, you may need to close and reopen the compose message window.

Implement a task pane
This add-in's Insert gist button will open a task pane and display the user's gists. The user can then select one of the gists to insert into the body of the message. If the user hasn't yet configured the add-in, they'll be prompted to do so.

Specify the HTML for the task pane
In the project that you've created, the task pane HTML is specified in the file ./src/taskpane/taskpane.html. Open that file and replace the entire contents with the following markup.

HTML

Copy
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=Edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Contoso Task Pane Add-in</title>

    <!-- Office JavaScript API -->
    <script type="text/javascript" src="https://appsforoffice.microsoft.com/lib/1.1/hosted/office.js"></script>

   <!-- For more information on Fluent UI, visit https://developer.microsoft.com/fluentui. -->
    <link rel="stylesheet" href="https://static2.sharepointonline.com/files/fabric/office-ui-fabric-core/9.6.1/css/fabric.min.css"/>

    <!-- Template styles -->
    <link href="taskpane.css" rel="stylesheet" type="text/css" />
</head>

<body class="ms-font-l ms-landing-page">
  <main class="ms-landing-page__main">
    <section class="ms-landing-page__content ms-font-m ms-fontColor-neutralPrimary">
      <div id="not-configured" style="display: none;">
        <div class="centered ms-font-xxl ms-u-textAlignCenter">Welcome!</div>
        <div class="ms-font-xl" id="settings-prompt">Please choose the <strong>Settings</strong> icon at the bottom of this window to configure this add-in.</div>
      </div>
      <div id="gist-list-container" style="display: none;">
        <form>
          <div id="gist-list">
          </div>
        </form>
      </div>
      <div id="error-display" style="display: none;" class="ms-u-borderBase ms-fontColor-error ms-font-m ms-bgColor-error ms-borderColor-error">
      </div>
    </section>
    <button class="ms-Button ms-Button--primary" id="insert-button" tabindex=0 disabled>
      <span class="ms-Button-label">Insert</span>
    </button>
  </main>
  <footer class="ms-landing-page__footer ms-bgColor-themePrimary">
    <div class="ms-landing-page__footer--left">
      <img src="../../assets/logo-filled.png" />
      <h1 class="ms-font-xl ms-fontWeight-semilight ms-fontColor-white">Git the gist</h1>
    </div>
    <div id="settings-icon" class="ms-landing-page__footer--right" aria-label="Settings" tabindex=0>
      <i class="ms-Icon enlarge ms-Icon--Settings ms-fontColor-white"></i>
    </div>
  </footer>
  <script type="text/javascript" src="../../node_modules/jquery/dist/jquery.js"></script>
  <script type="text/javascript" src="../../node_modules/showdown/dist/showdown.min.js"></script>
  <script type="text/javascript" src="../../node_modules/urijs/src/URI.min.js"></script>
  <script type="text/javascript" src="../helpers/addin-config.js"></script>
  <script type="text/javascript" src="../helpers/gist-api.js"></script>
  <script type="text/javascript" src="taskpane.js"></script>
</body>

</html>
Specify the CSS for the task pane
In the project that you've created, the task pane CSS is specified in the file ./src/taskpane/taskpane.css. Open that file and replace the entire contents with the following code.

css

Copy
/* Copyright (c) Microsoft. All rights reserved. Licensed under the MIT license. See full license in root of repo. */
html, body {
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
  overflow: auto; }

body {
  position: relative;
  font-size: 16px; }

main {
  height: 100%;
  overflow-y: auto; }

footer {
  width: 100%;
  position: relative;
  bottom: 0;
  margin-top: 10px;}

p, h1, h2, h3, h4, h5, h6 {
  margin: 0;
  padding: 0; }

ul {
  padding: 0; }

#settings-prompt {
  margin: 10px 0;
}

#error-display {
  padding: 10px;
}

#insert-button {
  margin: 0 10px;
}

.clearfix {
  display: block;
  clear: both;
  height: 0; }

.pointerCursor {
  cursor: pointer; }

.invisible {
  visibility: hidden; }

.undisplayed {
  display: none; }

.ms-Icon.enlarge {
  position: relative;
  font-size: 20px;
  top: 4px; }

.ms-ListItem-secondaryText,
.ms-ListItem-tertiaryText {
  padding-left: 15px;
}

.ms-landing-page {
  display: -webkit-flex;
  display: flex;
  -webkit-flex-direction: column;
          flex-direction: column;
  -webkit-flex-wrap: nowrap;
          flex-wrap: nowrap;
  height: 100%; }

.ms-landing-page__main {
  display: -webkit-flex;
  display: flex;
  -webkit-flex-direction: column;
          flex-direction: column;
  -webkit-flex-wrap: nowrap;
          flex-wrap: nowrap;
  -webkit-flex: 1 1 0;
          flex: 1 1 0;
  height: 100%; }

.ms-landing-page__content {
  display: -webkit-flex;
  display: flex;
  -webkit-flex-direction: column;
          flex-direction: column;
  -webkit-flex-wrap: nowrap;
          flex-wrap: nowrap;
  height: 100%;
  -webkit-flex: 1 1 0;
          flex: 1 1 0;
  padding: 20px; }

.ms-landing-page__content h2 {
  margin-bottom: 20px; }

.ms-landing-page__footer {
  display: -webkit-inline-flex;
  display: inline-flex;
  -webkit-justify-content: center;
          justify-content: center;
  -webkit-align-items: center;
          align-items: center; }

.ms-landing-page__footer--left {
  transition: background ease 0.1s, color ease 0.1s;
  display: -webkit-inline-flex;
  display: inline-flex;
  -webkit-justify-content: flex-start;
          justify-content: flex-start;
  -webkit-align-items: center;
          align-items: center;
  -webkit-flex: 1 0 0px;
          flex: 1 0 0px;
  padding: 20px; }

.ms-landing-page__footer--left:active {
  cursor: default; }

.ms-landing-page__footer--left--disabled {
  opacity: 0.6;
  pointer-events: none;
  cursor: not-allowed; }

.ms-landing-page__footer--left--disabled:active, .ms-landing-page__footer--left--disabled:hover {
  background: transparent; }

.ms-landing-page__footer--left img {
  width: 40px;
  height: 40px; }

.ms-landing-page__footer--left h1 {
  -webkit-flex: 1 0 0px;
          flex: 1 0 0px;
  margin-left: 15px;
  text-align: left;
  width: auto;
  max-width: auto;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis; }

.ms-landing-page__footer--right {
  transition: background ease 0.1s, color ease 0.1s;
  padding: 29px 20px; }

.ms-landing-page__footer--right:active, .ms-landing-page__footer--right:hover {
  background: #005ca4;
  cursor: pointer; }

.ms-landing-page__footer--right:active {
  background: #005ca4; }

.ms-landing-page__footer--right--disabled {
  opacity: 0.6;
  pointer-events: none;
  cursor: not-allowed; }

.ms-landing-page__footer--right--disabled:active, .ms-landing-page__footer--right--disabled:hover {
  background: transparent; }
Specify the JavaScript for the task pane
In the project that you've created, the task pane JavaScript is specified in the file ./src/taskpane/taskpane.js. Open that file and replace the entire contents with the following code.

JavaScript

Copy
(function(){
  'use strict';

  let config;
  let settingsDialog;

  Office.initialize = function(reason){

    jQuery(document).ready(function(){

      config = getConfig();

      // Check if add-in is configured.
      if (config && config.gitHubUserName) {
        // If configured, load the gist list.
        loadGists(config.gitHubUserName);
      } else {
        // Not configured yet.
        $('#not-configured').show();
      }

      // When insert button is selected, build the content
      // and insert into the body.
      $('#insert-button').on('click', function(){
        const gistId = $('.ms-ListItem.is-selected').val();
        getGist(gistId, function(gist, error) {
          if (gist) {
            buildBodyContent(gist, function (content, error) {
              if (content) {
                Office.context.mailbox.item.body.setSelectedDataAsync(content,
                  {coercionType: Office.CoercionType.Html}, function(result) {
                    if (result.status === Office.AsyncResultStatus.Failed) {
                      showError('Could not insert gist: ' + result.error.message);
                    }
                });
              } else {
                showError('Could not create insertable content: ' + error);
              }
            });
          } else {
            showError('Could not retrieve gist: ' + error);
          }
        });
      });

      // When the settings icon is selected, open the settings dialog.
      $('#settings-icon').on('click', function(){
        // Display settings dialog.
        let url = new URI('dialog.html').absoluteTo(window.location).toString();
        if (config) {
          // If the add-in has already been configured, pass the existing values
          // to the dialog.
          url = url + '?gitHubUserName=' + config.gitHubUserName + '&defaultGistId=' + config.defaultGistId;
        }

        const dialogOptions = { width: 20, height: 40, displayInIframe: true };

        Office.context.ui.displayDialogAsync(url, dialogOptions, function(result) {
          settingsDialog = result.value;
          settingsDialog.addEventHandler(Office.EventType.DialogMessageReceived, receiveMessage);
          settingsDialog.addEventHandler(Office.EventType.DialogEventReceived, dialogClosed);
        });
      })
    });
  };

  function loadGists(user) {
    $('#error-display').hide();
    $('#not-configured').hide();
    $('#gist-list-container').show();

    getUserGists(user, function(gists, error) {
      if (error) {

      } else {
        $('#gist-list').empty();
        buildGistList($('#gist-list'), gists, onGistSelected);
      }
    });
  }

  function onGistSelected() {
    $('#insert-button').removeAttr('disabled');
    $('.ms-ListItem').removeClass('is-selected').removeAttr('checked');
    $(this).children('.ms-ListItem').addClass('is-selected').attr('checked', 'checked');
  }

  function showError(error) {
    $('#not-configured').hide();
    $('#gist-list-container').hide();
    $('#error-display').text(error);
    $('#error-display').show();
  }

  function receiveMessage(message) {
    config = JSON.parse(message.message);
    setConfig(config, function(result) {
      settingsDialog.close();
      settingsDialog = null;
      loadGists(config.gitHubUserName);
    });
  }

  function dialogClosed(message) {
    settingsDialog = null;
  }
})();
Test the Insert gist button
Save all of your changes and run npm start from the command prompt, if the server isn't already running. Then, complete the following steps to test the Insert gist button.

Open Outlook and compose a new message.

In the compose message window, select the Insert gist button. You should see a task pane open to the right of the compose form.

In the task pane, select the Hello World Html gist and select Insert to insert that gist into the body of the message.

The add-in task pane and the selected gist content displayed in the message body.

Next steps
In this tutorial, you've created an Outlook add-in that can be used in message compose mode to insert content into the body of a message. To learn more about developing Outlook add-ins, continue to the following article.


See also
Outlook add-in manifests
Outlook add-in design guidelines
Add-in commands for Outlook
Debug function commands in Outlook add-ins
Recommended content
Insert data in the body in an Outlook add-in - Office Add-ins
Learn how to insert data into the body of a message or appointment in an Outlook add-in.
Configure your Outlook add-in for event-based activation - Office Add-ins
Learn how to configure your Outlook add-in for event-based activation.
Contextual Outlook add-ins - Office Add-ins
Initiate tasks related to a message without leaving the message itself to result in an easier and richer user experience.
Outlook add-in APIs - Office Add-ins
Learn how to reference the Outlook add-in APIs and declare permissions in your Outlook add-in.
Feedback
Submit and view feedback for

 
 View all page feedback
In this article

Prerequisites
Setup
Create an Outlook add-in project
Define buttons
