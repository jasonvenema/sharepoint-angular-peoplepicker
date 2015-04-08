# sharepoint-angular-peoplepicker
A client-side people picker control for SharePoint wrapped in an AngularJS directive.

![SharePoint People Picker](https://github.com/jasonvenema/sharepoint-angular-peoplepicker/blob/master/screenshot.png)

##How To
This people picker directive is based on the client side people picker included in the Office Dev/PnP code samples (https://github.com/OfficeDev/PnP). It is entirely JavaScript based, and can be used in an Angular application like so:

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
The App Web URL for a SharePoint provider hosted app is normally passed to the application as a query parameter. Most provider hosted applications will need to have access to this URL for making API calls back to the App Web. If you are not receiving the App Web URL in your query parameters, you will want to check the AppManifest.xml file for your application aand ensure that the **Query string** is configured to pass the '{StandardTokens}' expression.
