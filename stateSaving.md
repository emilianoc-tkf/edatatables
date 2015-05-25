# Introduction #

State saving allows to remember sorting, pagination, column order/visibility and filtering for current user between page reloads.

It can be done on the client side (cookies) or server side (session or database).

Note that saving state in a cookie on the client side will make the browser send that cookie with **every** request, even not related to EDataTables and in every action.
If you have multiple EDataTables on different pages that can lead to creating huge requests that the server will not be able to handle.

Thus, the server side method is recommended.

# Server side #

For server side state saving/restoring a static method called EDataTables::restoreStateSession() is provided.
It will detect when state options where passed in a request and save it or restore it if possible.

For saving/restoring sort and pagination options, you must use EDTSort and EDTPagination classes for the dataProvider.

Value of the quicksearch field from the table's toolbar will also be saved/restored.

It will also reoder the $columns array and set the 'visible' attribute where necessary.

```php

// note, the first argument is the id passed to EDataTables as well
EDataTables::restoreStateSession('report', null, null, $columns);
$dataProvider = $reportObject->search($_GET, $columns);
$this->getController()->createWidget('ext.EDataTables.EDataTables', array(
'id' => 'report',
'dataProvider' => $dataProvider,
'columns' => $columns,
);
```

# Client side (not recommended) #

While initializing the widget, set the _bStateSave_ option to _true_:
```php

'options' => array(
'bStateSave' => true,
),
```

This will tell dataTables to store current state in a cookie.

The method EDataTables::restoreState() reads that cookie and restore default sorting and pagination into the `$_GET` array. It must be called before fetching data from the dataProvider, for which CSort and CPagination (or EDTSort and EDTPagination) must parse `$_GET` data.
You **do not** need to call it if you don't mess with the dataProvider's getData/setData methods. It is called while initializing the widget.

Normally, you pass the dataProvider to the widget, which fetches data from it as late as necessary. If you need to fetch that data earlier and modify it, you need to call EDataTables::restoreState() yourself, like this:
```php

EDataTables::restoreState('raport');
$dataProvider = $reportObject->search($_GET, $columns);
$dataProvider->setData(messWithTheData($dataProvider->getData()));
$this->getController()->createWidget('ext.EDataTables.EDataTables', array(
'id' => 'raport',
'dataProvider' => $dataProvider,
'columns' => $reportObject->widgetColumns(),
'options' => array(
'bStateSave' => true,
),
);
```