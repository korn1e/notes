
### Code Review Report (2023-01-15) 1

When opening edit PO page, following code snippet is called:

```javascript
ahmgabst002TableAssignmentEdit: function (event, id, bunused) {
    //...code removed

    var asgmntData = [],
        appExt = [],
        appInt = [];

    $.ajax({
        async: false,
        // ...code removed
        success: function (response) {
            asgmntData = response.data[0];
        }
    });
    $.ajax({
        async: false,
        //...code removed
        success: function (response) {
            appExt = response.data.sort((a, b) => {
                // ...code removed
            });
        }
    });
    $.ajax({
        async: false,
        // ...code removed
        success: function (response) {
            appInt = response.data.sort((a, b) => {
                // ...code removed
            });
        }
    });

    var obj = {
        "asgmntData": asgmntData,
        "appExt": appExt,
        "appInt": appInt
    };
    // ...
    ahmgabst002.ahmgabst002BindTableAssignmentEdit(obj);
}
```

Ajax call is asynchronous, even though using `async: false`(deprecated), and it may not work.  
Try to promisify the ajax call and use `async` `await` to call them.  
Try following when declaring object (without quotes on field name):
```javascript
var obj = {
    asgmntData: asgmntData,
    appExt: appExt,
    appInt: appInt
};
```