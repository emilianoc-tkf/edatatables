# Introduction #

Selected rows' keys and values of editable cells are stored in a similar way, in a hidden field rendered right after (next to) the table.

When a user selects or deselects a row, key value associated with that row is added into one of those hidden fields.

If the table is a part of a form, those fields will be contained in the $_POST array and can be processed on the server-side._

Their names and ids are
  * widgetID-all-selected - takes values of: an empty string, 'select' or 'deselect'
  * widgetId-selected for new selections, empty if all-selected == 'select'
  * widgetId-deselected for deselected rows (exceptions) when everything was selected, empty if all-selected != 'select'
  * widgetId-disconnect for deselected rows, which were selected on page load, empty if all-selected != 'select'
  * widgetId-values for contents of editable columns' cells.

where widgetId is the id property of the EDataTables widget.

It will be described below how to read this server-side.

While drawing the table some rows can be marked as selected by evaluating the "checked" property of a checkbox column.

For easier client-side processing, all those fields have lists binded through $.data() function, check out the source of assets/jdatatable.js file to see how they are accessed.

# Configuring #

## Selectable rows ##

Set the _selectableRows_ option to 1 for single-selection and 2 for multiple-selection.

Add a ECheckBoxColumn to the columns array. You can put a PHP expression into the 'checked' attribute to mark rows as selected when drawing the table/fetching data to fill it.

Rest will be handled by EDataTables, including changing the CSS class of selected rows.

## Editable columns ##

Add a column of EEditableColumn class to the columns list, set its _type_ and _value_ attributes. Check out the source in the EEditableColumn.php file for available types and how input controls are drawn.

### Options ###

  * type - defines what control will be used, see below
  * dynamicType - an expression, which result will be used instead of _type_; if the result starts with 'readonly' it will be stripped and a normal cell will be rendered
  * enabled - true/false or a string with an expression returning true/false

### Supported types ###

  * readonly - renders the cell in same way EDataColumn would
  * text - ordinary text input
  * password - password input, will display asterisks instead of every character
  * integer, double, dec2, dec3, dec5 - a shorter text input
  * flags, set, object - renders a dropdown list using the [select2 extension](http://www.yiiframework.com/extension/select2), should work on relation columns
  * boolean - a checkbox
  * date, time, datetime - a datepicker, currently renders a TbDatepicker widget


# Processing #

One of edatatables uses could be assigning HAS\_MANY or MANY\_MANY related models in an update form.
Additionaly, if it's used for a MANY\_MANY relation, the pivot model could hold some relation parameters in addition to the foreign keys. You want to fill it and save it while assigning the models.

When a user clicks on the checkbox column header, selection options pop-up.
Those are:
  * Select all
  * Deselect all
  * Select all on page
  * Deselect all on page

When selecting or deselecting everything, EDataTables switches between three modes. This was implemented, because the table can show thousands of records and it's not efficient to store and send key of every row if everything was selected. In such case, we can use same filters used to display data to associate or disconnect relations with only few keys as exceptions.

Selecting or deselecting whole page is treated as the user would click on every row by hand.

## Default mode ##

This corresponds to an empty value of the 'all-selected' hidden field.

It's the default mode after page loads.

When a row is selected, it is added to the 'selected' hidden field.

When a row that was initially selected (on page load) is deselected, it is added to the 'disconnect' hidden field.

When processing this data on the server side, we have to:
  * associate current record with rows from the relation matching keys specified in 'select' field;
  * disconnect relations rows matching keys from 'disconnect' field;

## Select all mode ##

This corresponds to 'select' value of the 'all-selected' hidden field.

All rows are considered selected by default, so no keys are added to the 'select' hidden field.

Deselected rows are treated as exceptions, and their keys are added to the 'deselect' hidden field.

Rows selected on page load are added to 'disconnect' hidden field when deselected.

When processing this data on the server side, we have to:
  * associate current record with every row from the relation, except keys specified in 'deselect' field
  * disconnect relations rows matching keys from 'disconnect' field;

## Deselect all mode ##

This corresponds to 'deselect' value of the 'all-selected' hidden field.

All rows are considered deselected by default, that means all existing relations should be cleared.

Selected rows' keys are added to the 'select' hidden field unless they were already selected on page load.

When processing this data on the server side, we have to:
  * associate current records with relations rows matching keys from 'select' field, skipping already associated;
  * disconnect current record with every row from the relation, except keys specified in 'select' field


## Example ##

Use this code to preprocess received data and put it in arrays:
```php

$relation = 'widgetId';

$update = array();
$sel = trim($_POST[$relation.'-selected'],', ');
$des = trim($_POST[$relation.'-deselected'],', ');
if (isset($_POST[$relation.'-values'])) parse_str($_POST[$relation.'-values'], $update);
$relatedData[$relation] = array(
'insert' => empty($sel) ? array() : explode(',',$sel),
'delete' => empty($des) ? array() : explode(',',$des),
'update' => $update,
);
```

Now, the $relatedData array holds this information:
  * 'insert' - keys of models that needs to be assigned to model which was edited, that is updating a foreign key for a HAS\_MANY relation or inserting a row into a pivot table for a MANY\_MANY relation.
  * 'delete' -  similar to above, but for setting a foreign key to default value or removing whole row for HAS\_MANY relation and removing a row from a pivot table for a MANY\_MANY relation.
  * 'update' - extra parameters to put into a pivot table for a MANY\_MANY relation.

# Restoring on page re-draw #

When submitting a form with datatables, which fails validation and needs to be redrawn, the edatatables widget can reset selected rows and editable cells' values.
Just pass the option below when initializing the widget:
```php

'unsavedChanges' => array(
'all-selected' => (isset($_GET['tableId-all-selected']) ? $_GET['tableId-all-selected'] : ''),
'selected' => (isset($_GET['tableId-selected']) ? $_GET['tableId-selected'] : ''),
'deselected' => (isset($_GET['tableId-deselected']) ? $_GET['tableId-deselected'] : ''),
'disconnect' => (isset($_GET['tableId-disconnect']) ? $_GET['tableId-disconnect'] : ''),
'values' => (isset($_GET['tableId-values']) ? $_GET['tableId-values'] : ''),
),
```