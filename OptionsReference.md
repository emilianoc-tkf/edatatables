# EDataTables options #

List of attributes of the _EDataTables_ class.

Because _EDataTables_ extends _CGridView_ class, it can use its options too.

| **Name**             | **Type**  | **Default**    | **Description** |
|:---------------------|:----------|:---------------|:----------------|
| filterForm           | string    | null           | jQuery selector of a filter form that is serialized and appended to every Ajax request |
| serverData           | array     | array()        | Additional key/value pairs to be sent to the server data backend. If the value string starts with 'js:' it will be included as a callback, without quotes. |
| datatableTemplate    | string    | null           | Overrides the sDom DataTables option. |
| tableBodyCssClass    | string    | null           | CSS class set for tbody tag. |
| itemsCssClass        | string    | null           | CSS class set for the table tag. If null, defaults for JUI or Twitter Bootstrap are used. |
| newAjaxUrl           | string    | null           | URL used by the "new" toolbar button. |
| selectionChanged     | string    | null           | JS callback when selection changes. |
| editableSelectsRow   | boolean   | true           | If a change in an editable cell triggers selection of that row. |
| relatedOnlyOption    | boolean   | true           | Should a "related only" checkbox be added to the table's toolbar. |
| options              | array     | see below      |                 |
| unsavedChanges       | array     | see src/wiki   | see [stateSaving](http://code.google.com/p/edatatables/wiki/stateSaving) |
| buttons              | array     | see src/wiki   | see [Buttons](http://code.google.com/p/edatatables/wiki/Buttons) |
| bootstrap            | boolean   | false          | Should the Twitter Bootstrap theme be used instead of JUI Smoothness. |

# DataTables options #

List of options passed to the DataTables JavaScript plugin.

**WARNING** Some options, like fnServerParams or fnServerData are set explicitly. Your value WILL BE OVERWRITTEN. See below for explanation.

Those are only the defaults set by EDataTables. For a full reference please consult [the DataTables site](http://datatables.net/).

| **Name**          | **Default**                             |
|:------------------|:----------------------------------------|
| asStripClasses    | value of the rowCssClass property       |
| iDeferLoading     | false when there is no data to draw     |
| sAjaxSource       | value of the ajaxUrl property           |
| aoColumnDefs      | based on the columns property           |
| sDom              | based on the bootstrap or datatableTemplate property       |
| bScrollCollapse   | false                                   |
| bStateSave        | false                                   |
| bPaginate         | true                                    |
| sCookiePrefix     | 'edt_'_|
| bJQueryUI         | based on the bootstrap property         |
| oLanguage         | Localization using Yii::t('EDataTables.edt') module       |
| sPaginationType   | based on the bootstrap property         |
| oSearch           | set to $_GET['sSearch']_|
| oColReorder       | set if configure button is enabled      |
| fnServerParams    | set to custom function, see below       |
| fnServerData      | set to custom function, see below       |

## fnServerParams ##

A custom function is used explicitly, so you can't pass your own.

Use the _serverData_ property of EDataTables widget class.

The custom function adds

  * a serialized filter form,
  * visible columns indexes in order,
  * removes some dataTables params to make request smaller

List of removed params:

  * bRegex
  * iColumns
  * bRegex\_XX
  * bSearchable\_XX
  * sSearch\_XX
  * bSortable\_XX
  * mDataProp\_XX

## fnServerData ##

A custom function is used explicitly, so you can't pass your own.

Use the _beforeAjaxSuccess_ and _afterAjaxUpdate_ properties of EDataTables widget class.

The custom function:

  * aborts previous request
  * calls _beforeAjaxUpdate_
  * on success calls in order:
    1. _beforeAjaxSuccess_ that refresh keys div element
    1. dataTables callback
    1. _afterAjaxUpdate_