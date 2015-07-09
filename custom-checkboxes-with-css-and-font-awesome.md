## Introduction

This article will show you how to customize checkbox styles using icon fonts.

## The classic view

Often we want to create a list of checkboxes with their corresponding labels.

[code language="html"]

	  <ul>
	    <li>
	      <input id="checkbox1" type="checkbox"/> 
	      <label for="checkbox1">Checkbox One</label>
	    </li>
	    <li>
	      <input id="checkbox2" type="checkbox" checked/> 
	      <label for="checkbox2">Checkbox Two</label>
	    </li>
	  </ul>
	
		
[/code]

## Adding icon fonts

To replace our checkboxes with icon fonts we will first hide our current checkboxes.

[code language="html" highlight="3, 7"]

	  <ul>
	    <li>
	      <input id="checkbox1" type="checkbox" hidden /> 
	      <label for="checkbox1">Checkbox One</label>
	    </li>
	    <li>
	      <input id="checkbox2" type="checkbox" checked hidden /> 
	      <label for="checkbox2">Checkbox Two</label>
	    </li>
	  </ul>
	
		
[/code]

We will then add CSS to replace the checkbox visual. Please note this example is using [Font Awesome's icons](http://fortawesome.github.io/Font-Awesome). For this example to work you will need to include their font on your page.

[code language="css"]

	input[type=checkbox] + label::before
	{
	    color: black;
	    font-family: FontAwesome;
	    font-weight: normal;
	    font-size: 22px;
	    height: 22px;
	    width: 22px;
	    display:inline-block;
	    content:"\f096";
	}
	
	input[type=checkbox] + label:hover::before
	{
	    font-weight: bold;
	}
	
	input[type=checkbox]:checked + label::before
	{
	    content:"\f14a";
	}
		
[/code]

I have provided a complete example [on JS Bin](http://jsbin.com/maxoqoyole/1/edit?html,css,output).