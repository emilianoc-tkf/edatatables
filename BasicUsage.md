It's not 100% compatible with CGridView. I've decided not to alter the GET parameter names used by DataTables, so you have to use the provided EDTSort and EDTPagination classes as well as alter filter processing. See below.

# Installation #

  * Extract into the extensions directory, creating EDataTables dir
  * Import in config/main.php
```php

'import' => array(
...
'ext.EDataTables.*',
...
```

# Using #

Use similar to CGridView. If displayed in a normal call just run the widget. To fetch AJAX response send json encoded result of $widget->getFormattedData().

The action in a controller:
```php

$widget=$this->createWidget('ext.EDataTables.EDataTables', array(
'id'            => 'products',
'dataProvider'  => $dataProvider,
'ajaxUrl'       => Yii::app()->getBaseUrl().'/products/index',
'columns'       => $columns,
));
if (!Yii::app()->getRequest()->getIsAjaxRequest()) {
$this->render('index', array('widget' => $widget,));
return;
} else {
json_encode($widget->getFormattedData(intval($_REQUEST['sEcho'])));
Yii::app()->end();
}
```

The index view (for non-ajax requests):

```php

<?php $widget->run(); ?>
```

# Preparing the dataprovider #

To use features like sorting, pagination and filtering (by quick search field in the toolbar or a custom advanced search filter form) the dataprovider object passed to the widget must be prepared using provided EDTSort and EDTPagination class and CDbCriteria filled after parsing sent forms.

The simplest example:
```php

$criteria = new CDbCriteria;
// bro-tip: $_REQUEST is like $_GET and $_POST combined
if (isset($_REQUEST['sSearch']) && isset($_REQUEST['sSearch']{0})) {
// use operator ILIKE if using PostgreSQL to get case insensitive search
$criteria->addSearchCondition('textColumn', $_REQUEST['sSearch'], true, 'AND', 'ILIKE');
}

$sort = new EDTSort('ModelClass', $sortableColumnNamesArray);
$sort->defaultOrder = 'id';
$pagination = new EDTPagination();

$dataProvider = new CActiveDataProvider('ModelClass', array(
'criteria'      => $criteria,
'pagination'    => $pagination,
'sort'          => $sort,
))
```

Enabling search on a relation column:
```php

// eager loading, because sorting is done before pagination (OFFSET/LIMIT)
$columns = array(
'name:text',
'category.name:text',
);
$criteria->with = array('category');
$sort->attributes = array(
// atributes from $model
'name',
// a virtual attribute, key is the column name
'category' => array(
'asc' => '"category".name ASC',
'desc' => '"category".name DESC',
),
);
```
Remember, if using some complex db expressions in sort, you need to add all used columns to select part of the query, at least in PostgreSQL. You can do this like this:
```php

$criteria->sort = '*, column1_used_in_sort, column2_used_in_sort';
```

An advanced example would be based on a search form defined with a model and a view. Its attributes would be then put into a critieria and passed to a dataProvider.