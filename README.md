# sharepoint-angular-peoplepicker
A client-side people picker control for SharePoint wrapped in an AngularJS directive.

![SharePoint People Picker](https://github.com/jasonvenema/sharepoint-angular-peoplepicker/blob/master/screenshot.png)

##How To
This people picker directive is based on the client side people picker included in the Office Dev/PnP code samples (https://github.com/OfficeDev/PnP). It is entirely JavaScript based, and can be used in an Angular application like so:

```javascript
var app = angular.module('app', ['sp-peoplepicker']);
```

Then in your view, the directive can be instantiated like this:

```html
<sp-people-picker name="taskAssignee" id="taskAssignee" ng-model="$scope.taskAssignees" min-entries="1" max-entries="5" allow-duplicates="false" show-login="false" show-title="true" min-characters="2" app-web-url="$scope.spAppWebUrl" />
```

If you are using bootstrap and want to add some validation to your form, you can use:

```html
<form name="taskForm" role="form" ng-submit="$scope.ok(taskForm.$valid)" novalidate>
  <div class="form-group" ng-class="{ 'has-error' : taskForm.taskAssignee.$invalid && !taskForm.taskAssignee.$pristine }">
    <label for="taskAssignee">Assign To <span class="text-danger">*</span></label>
    <sp-people-picker name="taskAssignee" id="taskAssignee" ng-model="$scope.taskAssignees" max-entries="5" allow-duplicates="false" show-login="false" show-title="true" min-characters="2" min-entries="1" app-web-url="$scope.spAppWebUrl" />
    <p ng-show="taskForm.taskAssignee.$invalid && !taskForm.taskAssignee.$pristine" class="help-block">You must assign the task to at least 1, but not more than 5 people</p>
  </div>
</form>
```

To get the user values from the people picker and, for example, update a user field in SharePoint using the CSOM, you can do the following in your controller:

```javascript
angular.forEach($scope.taskAssignees, function (value, key) {
    users.push(SP.FieldUserValue.fromUser(value.Login));
});
listItem.set_item('AssignedTo', users);
listItem.update();
```

By way of a longer example, you could update the Tasks list in your app's Host Web by invoking the following function from your controller:

```javascript
function saveTask(task) {
  var dfd = $q.defer();
  var ctx = getSpCtx();
  var hostWeb = getHostWeb(ctx);
  var list = hostWeb.get_lists().getByTitle('Tasks');
  var listItemCreation = new SP.ListItemCreationInformation();
  var listItem = list.addItem(listItemCreation);
  listItem.set_item('Title', task.Title);
  listItem.set_item('Body', task.Body);
  listItem.set_item('DueDate', task.DueDate);
  listItem.set_item('_Category', task.Category);

  var users = [];
  angular.forEach(task.AssignedToId, function (value, key) {
      users.push(SP.FieldUserValue.fromUser(value.Login));
  });
  listItem.set_item('AssignedTo', users);
  listItem.update();
  ctx.load(listItem);
  ctx.executeQueryAsync(Function.createDelegate(this, saveComplete), Function.createDelegate(this, saveFailed));

  function saveComplete() {
      dfd.resolve(listItem);
  }

  function saveFailed(sender, args) {
      console.log("XHR failed for task create POST: " + args.get_message());
      dfd.reject(args.get_message());
  }

  return dfd.promise;
}

function getSpCtx() {
    // get SPAppWebUrl from the {StandardTokens} query parameters
    var ctx = new SP.ClientContext(spappcontext.hostWeb.SPAppWebUrl);
    var factory = new SP.ProxyWebRequestExecutorFactory(spappcontext.hostWeb.SPAppWebUrl);
    ctx.set_webRequestExecutorFactory(factory);
    return ctx;
}

function getHostWeb(ctx) {
    // get SPHostUrl from {StandardTokens} query parameters
    var hostWebctx = new SP.AppContextSite(ctx, spappcontext.hostWeb.SPHostUrl);
    var appWeb = ctx.get_web();
    var hostWeb = hostWebctx.get_web();
    return hostWeb;
}
```

##Arguments
* **ng-model**: Specifies the variable in the controller that will get the array of resolved useds in the people picker
* **max-entries**: The maximum number of people that an instance of the people picker will resolve. This is useful if you want to limit, say, the number of people that a task can be assigned to.
* **min-entries**: The minimum number of people that an instance of the people picker will resolve. A value of one or greater has the effect of making the people picker a required field in the form.
* **allow-duplicates**: Set to 'true' to allow duplicate names, otherwise set to 'false'
* **show-login**: Set to 'true' to show the login name of users in the autocomplete dropdown
* **show-title**: Set to 'true' to show the job title of users in the autocomplete dropdown, if available
* **min-characters**: The minimum number of characters the user must type before the people picker control will attempt to find matching user names
* **app-web-url**: The absolute URL to the App Web for the provider hosted app. More information on obtaining this below.

##Obtaining the App Web URL
The App Web URL for a SharePoint provider hosted app is normally passed to the application as a query parameter. Most provider hosted applications will need to have access to this URL for making API calls back to the App Web. If you are not receiving the App Web URL in your query parameters, you will want to check the AppManifest.xml file for your application and ensure that the **query string** is configured to pass the '{StandardTokens}' expression.
