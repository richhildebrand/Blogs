## Introduction

This article will show you how to use the knockout foreach binding to create a simple grid.

## The View

First lets setup a simple grid.

[code language="html"]

	<!DOCTYPE html>
	<html>
	<head>
	    <script src="https://cdnjs.cloudflare.com/ajax/libs/knockout/3.3.0/knockout-min.js"></script>
	</head>
	<body id="View">
	  <table>
      	<thead>
      		<tr>
				<th>First name</th>
				<th>Last initial</th>
			</tr>
      	</thead>
      	<tbody>
          	<tr>
              	<td>Will be bound to the first name</td>
              	<td>Will be bound to the last initial</td>
          	</tr>
      	</tbody>
	  </table>
	  </body>
	</html>
		
[/code]

## The View Model

Now lets add a knockout view model to bind to our view.

[code language="javascript"]

	myApp = {} || myApp;
	
	myApp.viewModel = {
	  people: ko.observableArray([{
	    firstName: 'Rich',
	    lastInitial: 'H'
	  }, {
	    firstName: 'Steve',
	    lastInitial: 'S'
	  }, {
	    firstName: 'Jeffrey',
	    lastInitial: 'P'
	  }])
	};
		
[/code]

## The Data Binding
	
Now that we have our view and our view model we can bind them together using knockout.


[code language="html" highlight="6, 14, 16, 17"]

	<!DOCTYPE html>
	<html>
	<head>
	    <script src="https://cdnjs.cloudflare.com/ajax/libs/knockout/3.3.0/knockout-min.js"></script>
	</head>
	<body id="View">
	  <table>
      	<thead>
      		<tr>
				<th>First name</th>
				<th>Last initial</th>
			</tr>
      	</thead>
      	<tbody data-bind="foreach: people">
          	<tr>
              	<td data-bind="text: firstName"></td>
              	<td data-bind="text: lastInitial"></td>
          	</tr>
      	</tbody>
	  </table>
	  </body>
	</html>
		
[/code]

[code language="javascript" highlight="16"]
	
	myApp.viewModel = {
	  people: ko.observableArray([{
	    firstName: 'Rich',
	    lastInitial: 'H'
	  }, {
	    firstName: 'Steve',
	    lastInitial: 'S'
	  }, {
	    firstName: 'Jeffrey',
	    lastInitial: 'P'
	  }])
	};

	myApp.view = document.getElementById("View");

	ko.applyBindings(myApp.viewModel, myApp.view);
		
[/code]

These changes should give us the following grid.

	First name	Last initial
	Rich		H
	Steve		S
	Jeffrey		P	

## Calling functions with the bound data

We may also want to delete items from the grid. Lets add a reusable function to help knockout.

[code language="javascript"]
	
	myApp.koHelpers = {
	  removeItem: function(observableArray, itemToRemove, event) {
	    observableArray.remove(itemToRemove);
	  }
	};
		
[/code]

And the HTML to bind to our new function.

[code language="html" highlight="6"]

	<tbody data-bind="foreach: people">
		<tr>
		  	<td data-bind="text: firstName"></td>
		  	<td data-bind="text: lastInitial"></td>
			<td>
            	<button data-bind="click: myApp.koHelpers.removeItem.bind($data, $parent.people)"> delete </button>
			</td>
		</tr>
	</tbody>

[/code]

Please note that the parameter order is not quite what you would expect. $data represents the itemToRemove and $parent.people is the observableArray.

## Summary

Everything combined will appear as follows.

[code language="html"]

	<!DOCTYPE html>
	<html>
	<head>
	    <script src="https://cdnjs.cloudflare.com/ajax/libs/knockout/3.3.0/knockout-min.js"></script>
	</head>
	<body id="View">
	  <table>
	      <thead>
	          <tr><th>First name</th><th>Last initial</th></tr>
	      </thead>
	      <tbody data-bind="foreach: people">
	          <tr>
	              <td data-bind="text: firstName"></td>
	              <td data-bind="text: lastInitial"></td>
	              <td>
	                <button data-bind="click: myApp.koHelpers.removeItem.bind($data, $parent.people)">delete</button>
	              </td>
	          </tr>
	      </tbody>
	      <tfoot>
	        <tr>
	          <td>
	            
	          </td>
	        </tr>
	      </tfoot>
	  </table>
	  </body>
	</html>

[/code]

[code language="javascript"]
	
	
	myApp = {} || myApp;
	window.myApp = myApp;
	
	myApp.view = document.getElementById("View");
	
	myApp.viewModel = {
	  people: ko.observableArray([{
	    firstName: 'Rich',
	    lastInitial: 'H'
	  }, {
	    firstName: 'Steve',
	    lastInitial: 'S'
	  }, {
	    firstName: 'Jeffrey',
	    lastInitial: 'P'
	  }])
	};
	
	myApp.koHelpers = {
	  removeItem: function(observableArray, itemToRemove, event) {
	    observableArray.remove(itemToRemove);
	  }
	};
	
	ko.applyBindings(myApp.viewModel, myApp.view);
		
[/code]