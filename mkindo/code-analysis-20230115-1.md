
# Code Review Report (2023-01-15) 1

## Symptoms
When opening edit PO page, error 400 dialog box will be appeared due to invalid URLs (nulls):  
`/ahmgabst-aws/rest/bst002/get-listitem-po/null?_=1673775271626`  
`/ahmgabst-aws/rest/bst002/get-nomorpoByID/null?_=1673775271625`
## Findings
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

Ajax call is asynchronous, even though `async: false`(deprecated) is used, it may not work.  
At the end of the function it will call another function passing object that still has empty values. This will result incorrect REST calls ("null" path):  
`/ahmgabst-aws/rest/bst002/get-listitem-po/null?_=1673775271626`  
`/ahmgabst-aws/rest/bst002/get-nomorpoByID/null?_=1673775271625`

## Suggestions
### 1. Chaining the ajax call using different methods
```javascript
method3 = (r1, r2) => {
    $.ajax({
        success: function (response) {
            result = response.data;
            var obj = {
                asgmntData: r1,
                appExt: r2,
                appInt: response.data
            };
            ahmgabst002.ahmgabst002BindTableAssignmentEdit(obj);
        }
    });
}
method2 = (r1) => {
    $.ajax({
        success: function (response) {
            result = response.data;
            method3(r1, r2);
        }
    });
}
method1 = () => {
    $.ajax({
        success: function (response) {
            result = response.data;
            method2(result);
        }
    });
}
```
### 2. Promisify the ajax call and use `async`/`await` to call them.
```javascript
getAsgmntData = () => {
    return new Promise((resolve) => {
        $.ajax({
            success: function (data) {
                result = response.data;
                resolve(result)
            }
        })
    })
}
getAppExt = function => {
    return new Promise((resolve) => {
        $.ajax({
            success: function (data) {
                result = response.data;
                resolve(result)
            }
        })
    })
}
getAppInt = () => {
    return new Promise((resolve) => {
        $.ajax({
            success: function (data) {
                result = response.data;
                resolve(result)
            }
        })
    })
}

ahmgabst002TableAssignmentEdit = async () => {
    const asgmntData = await getAsgmntData();
    const appExt = await getAppExt();
    const appInt = await getAppInt();
    const obj = {
        asgmntData: asgmntData,
        appExt: appExt,
        appInt: appInt
    };
    ahmgabst002.ahmgabst002BindTableAssignmentEdit(obj);
}
```  

## Additional notes
In addition, use following when declaring object (without quotes on field name):
```javascript
var obj = {
    asgmntData: asgmntData,
    appExt: appExt,
    appInt: appInt
};
```
