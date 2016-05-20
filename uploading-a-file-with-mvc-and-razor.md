## Introduction

Often I need to upload a file to MVC. Usually these days I am doing so with JavaScript, but every once in a while I have the requirement to do so with razor. 

## The problem

Uploading a file with razor is something MVC has made fairly simple. It works much like a normal form post except that you must include `multipart/form-data` when creating your form.

## The razor

[code language="html"]

	@using (@Html.BeginForm("UploadAction", "UploadController", FormMethod.Post, new { enctype = "multipart/form-data" }))
    {
    	<input type="file" name="fileToUpload" id="fileToUpload" />
    	<input class="btn btn-primary" type="submit" value="Rename People" />
    }

[/code]

## The controller

[code language="csharp"]

	[HttpPost]
    public ActionResult UploadAction(HttpPostedFileBase fileToUpload)
    {
        return View();
    }

[/code]