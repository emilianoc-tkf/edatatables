# Introduction #

You can easily place buttons for pre-defined or custom actions in the table's header or a button column.

# Header #

Many times we need to perform custom actions on selected or all rows used to fill our table, such as:

  * export data from table to CSV or other format
  * print, that means opening a HTML page with CSS suited for printing or a generated PDF
  * create a new record
  * render a plot with contents of the table
  * mass actions on selected or all found records, like deletion or edit

EDataTables support configuring buttons with custom JS actions binded to them, that would display in the toolbar located in table header.

Buttons are initialized as JUI or Twitter Bootstrap buttons, hence one can use icons supplied with JUI for most common actions to create nice looking, space saving, intuitive buttons.

There are 5 pre-defined buttons available:
  * refresh - for reloading the contents of the table via ajax, could be used when an external filter is changed and that did not trigger an automatic refresh
  * configure - for showing a [select2](http://ivaynberg.github.com/select2/) control to change column order or visibility
  * print - open an PDF with contents of the table, currently no default callback is provided
  * export - download a CSV file with contents of the table, currently no default callback is provided
  * new - open a JUI or Twitter Bootstrap modal dialog with a form from the update action for creating a new record, close after successful validation and saving or cancelling

## Configuration ##

Buttons can be configured via the _buttons_ option:
```php

'buttons' => array(
'print' => null,
'export' => null,
'new' => array(
'callback' => 'js:function(){return alert("I beg you, never use alerts!");}',
),
),
```

By default, the only visible button is a refresh button, which reloads the content of a table, applying any currently set sorting, pagination and filtering options.

To turn off the default refresh button, set its value to null in the 'button' property:
```php

'buttons' => array(
'refresh' => null,
)
```

To add more buttons pass the 'buttons' option, specified by an array of:
```php

'buttonId' => array(
'label' => 'Print',
'text' => false,
'htmlClass' => 'printButton',
'icon' => 'ui-icon-print',
'callback' => 'js:functionNameOrDefinition',
```

where:
  * 'label' - label text
  * 'text' - if is set to true, label text will be visible in addition to the icon
  * 'htmlClass' - self explanatory, added to the &lt;button&gt; tag
  * 'icon' - used as the primary icon in jui button 'icon' property
  * 'callback' - custom onclick event, binded to the &lt;button&gt; tag; leave null for pre-defined buttons to use build-in callback function

## Examples ##

Each callback redirects the current page passing the values of an advanced search form as GET parameters:
```php

'buttons' => array(
'update' => array(
'label' => 'Update',
'text' => false,
'htmlClass' => 'editButton',
'icon' => 'icon-pencil',
'callback' => 'js:showMassEditModal',
)
```

# Button column #

EButtonColumn class provides same default buttons as CButtonColumn, so if you want a default CRUD operations button column set just the _class_ attribute.

In addition to standard CRUD buttons defined in CButtonColumn, EButtonColumn adds a _history_ button that is intented to open a log of record modifications.

Add a button column to the columns array:
```php

array(
'class' => 'ext.EDataTables.EButtonColumn',
'template' => '{view} {update}',
'header' => Yii::t('app','Operations'),
'buttons' => array(
'view' => array(
'label' => 'Check me out',
'imageUrl' => false,
'options' => array(
'class' => 'some-css-class-drawing-pretty-icons',
),
),
'update' => array(
'label' => 'Push me around',
'imageUrl' => false,
'options' => array(
'class' => 'some-css-class-drawing-pretty-icons',
),
'visible' => 'Yii::app()->user->checkAccess(\'update Chicks\',array(\'model\'=>$data))'
),
),
),
```