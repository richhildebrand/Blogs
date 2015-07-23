## Introduction

This article is intended to provide an elegant workaround to the missing dataValueField in the KendoUI Autocomplete widget.

## The Expectation

It is a very common desire to display text while binding to an id property. In most KendoUI widgets this would be accomplished through the dataValueField.

[code language="html" highlight="11, 12"]

	  <div id="view">
		<div>
		  <h4 data-bind="text: startingQuarterBack"></h3>
		</div>

	    <span> Pick your starting QB: </span>
	    <input data-role="autocomplete"
	           data-placeholder=""
	           data-value-primitive="true"
	           data-text-field="name"
			   data-value-field="id"
	           data-bind="value: startingQuarterBack,
	                      source: brownsQuarterBacks" />
	  </div>
		
[/code]

[code language="javascript" highlight="10"]

	var viewModel = kendo.observable({
	  brownsQuarterBacks: [
	    { id:"1", name: "Johnny Manziel" },
	    { id:"2", name: "Connor Shaw" },
	    { id:"3", name: "Josh McCown" },
	    { id:"4", name: "Thad Lewis" },
	    { id:"5", name: "Tyler Thigpen" },
	  ],
	  
	  startingQuarterBack: undefined,
	});
	
	var view = $("#view");
	kendo.bind(view, viewModel);
		
[/code]

Unfortunately the KendoUI Autocomplete widget does not support this field. This means when the observable is bound to the view, the value property, in this case startingQuarterBack, will be set to the dataTextField as seen [in this JSBin](http://jsbin.com/poqoqotabu/1/edit?html,js,output).

## The Solution
	
The workaround is to set dataValuePrimitive to false. This will return the entire data item instead of the textFieldValue.


[code language="html" highlight="3"]

    <input data-role="autocomplete"
           data-placeholder=""
           data-value-primitive="false"
           data-text-field="name"
		   data-value-field=id"
           data-bind="value: startingQuarterBack,
                      source: brownsQuarterBacks" />
		
[/code]

With dataValuePrimitive as false you can now bind to, and retrieve, the id property off of your value binding.

[code language="html" highlight="2"]
	
	   <div>
	      <h4 data-bind="text: startingQuarterBack.id"></h3>
	  </div>
			
[/code]

A complete example can be found [in this JSBin](http://jsbin.com/cigiliyegu/1/edit?html,js,output).