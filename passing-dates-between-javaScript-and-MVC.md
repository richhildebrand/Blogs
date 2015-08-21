Frequently I want to pass a date between javaScript and MVC. It would of course be nice if MVC could read a standard javaScript date, but this is unfortunately not the case.

## Passing a date from javaScript to MVC

To format a javaScript date in a way that MVC will accept it, you can use the method `toISOString`.

Here is a complete example utilizing `toISOString` to POST a javaScript date to an MVC controller.

[code language="javaScript" highlight="4"]

    var sendToday = function () {
        var currentTicks = Date.now();
        var today = new Date(currentTicks);
        var formattedDate = today.toISOString();

        $.ajax({
            type: "POST",
            url: "Demo/CatchToday",
            data: { today: formattedDate },
            dataType: "json",
        });
    };

[/code]

And here is a controller action to catch the posted date.

[code language="csharp"]

    [HttpPost]
    public void CatchToday(DateTime today)
    {
        Debug.WriteLine(today);
    }

[/code]

## Passing a date from MVC to javaScript

Of course I also frequently want to send a date from MVC to javaScript.

To do so you can simply call `new Date()` with the MVC response string.

Here is an example of the Ajax Request asking for the date.

[code language="javascript" highlight="6"]

    var getToday = function () {
        $.ajax({
            url: "Demo/GetToday",
        }).done(function (unparsedDate) {
            console.log(unparsedDate);
            var parsedDate = new Date(unparsedDate);
            console.log(parsedDate);
        });
    };

[/code]

As well as the controller action that will return the date.

[code language="csharp"]

    [HttpGet]
    public DateTime GetToday()
    {
        return DateTime.Today;
    }

[/code]
