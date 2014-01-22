Yii EchMultiSelect Widget
==============================

_EchMultiSelect_ is a simple Wrapper Widget for the jQuery UI MultiSelect Widget by Eric Hynds.

##Requirements

Tested with Yii 1.1.9.

##Unpack the widget

Extract the contents of the zip file directly into the `protected/extensions/` folder of your Yii application.

##Use the widget

###1. Basic Use with `model` and `dropDownAttribute`
Now you can use the widget in your view file, for example like this:
```php
$data= CHtml::listData(Color::model()->findAll(), 'ID', 'Name');
$this->widget('ext.EchMultiSelect.EchMultiSelect', array(
	'model' => $model,
	'dropDownAttribute' => 'color',		
	'data' => $data,
	'dropDownHtmlOptions'=> array(
		'style'=>'width:378px;',
	),
));
```

This Yii widget creates a 'drop down list' with the given data and applies the JQuery widget to it, which turns the ordinary HTML select control into an elegant MultiSelect list of checkboxes with themeroller support.

To avoid confusion: I refer to the initial select element (that is subsequently hidden) as the 'drop down list'; and to the eventual input element created by the JQuery Widget as the 'MultiSelect list'.

The provided 'dropDownHtmlOptions' will be adopted by the MultiSelect list. If you for example provide a style for the drop down list, it will also applied to the resulting MiltiSelect list.

The drop down list will be hidden directly after it is created: I placed a Javascript code to hide the element directly after it, to prevent it from being displayed while the page is loading, only to be hidden after the page has been loaded (as suggested [here](http://www.electrictoolbox.com/jquery-hide-text-page-load-show-later/ "")). I truly dislike that appearing/disappearing on page load, so I didn'd wait for the MultiSelect widget to hide the drop down list.

**Note**: You could also hide the drop down list by adding the style `display:none` to the `dropDownHtmlOptions`. But hiding it via JS-Code provides backward compatibility: if the user has JS disabled, multiselect will not work, but the original select element will stay visible.

###2. Basic Use with `name` and `value`
```php
$colors = array('red','yellow','orange','black','green','blue');
$this->widget('ext.EchMultiSelect.EchMultiSelect', array(
	'name'=>'colors[]',
	'data'=>$colors,
	'value'=>array(0,3),
	'dropDownHtmlOptions'=> array(
		'class'=>'span-10',
		'id'=>'colors',
	)
)); 
```
Here, `name` is the name of the drop down list. This must be set if `model` and `dropDownAttribute` are not set.
`value` contains the pre-selected input value(s). In this case, 'red' and 'yellow' will be pre-selected by default. `value` is used only if `model` is not set.
If you select red, green, blue and submit, `$_POST['colors']` will be an array containing the selected values, like: `Array([0] => 0,[1] => 4, [2] => 5)`.

If `model` and `dropDownAttribute` are specified, they are used to generate the name and id of the drop down list. Note that if `name` is specified in addition, it overrides the automatically generated name. Similarly, if `id` is specified in the `dropDownHtmlOptions`, it overrides the generated id.

###3. Use as a filter in `CGridView`
The following example was provided by [jeremy ](http://www.yiiframework.com/user/1627/ "jeremy") (see comments)
```php
$this->widget('zii.widgets.grid.CGridView', array(
	....
	'columns' => array (
		'firstColumn',
		'secondColumn',
		// use EchMultiSelect for the next column
		array (
			'name'=>'thirdColumn',
			'filter'=> $this->widget('ext.EchMultiSelect.EchMultiSelect', array(
				'model' => $model,
				'dropDownAttribute' => 'thirdColumn',
				'data' => $colors,
				'options' => array('buttonWidth' => 80, 'ajaxRefresh' => true),
			),
			true // capture output; needed so the widget displays inside the grid
		),
	),
));
```

By setting `'ajaxRefresh'=>true`, we indicate that the widget should be 'refreshed' after every ajax request that occurs on the current page.

Note that you have to change the original search criteria in the model file: 
```php
// $criteria->compare($alias.'.thirdColumn',$this->thirdColumn,true); // <-- original
// Now, that `$this->thirdColumn` is an array, you should use something like this:
if(!empty($this->thirdColumn)) {
	foreach($this->thirdColumn as $v) {
		$criteria->compare($alias.'.thirdColumnn', $v, false, 'OR');
	}
}
```

**Note:** In your controller file where you render the grid, you should make the 
following changes if you have pre-selected values 
(thanks to [Chris Backhouse](http://www.yiiframework.com/user/34858/ "Chris Backhouse") for pointing out this [issue](http://www.yiiframework.com/extension/echmultiselect/#c11243 'comment #11243')):
```php
public function actionAdmin()
{
	$model=new SomeModel('search');
	$model->unsetAttributes();  // clear any default values
	$model->thirdColumn = array('red','yellow'); // <-- pre-selected values.
	if(isset($_GET['SomeModel'])) {
		$model->attributes=$_GET['SomeModel'];
		if(!isset($_GET['SomeModel']['thirdColumn']))
			$model->thirdColumn = array(); // <-- clear pre-selected values
	}
}
```

###4. Overriding default `options` and `filterOptions`
You can provide optional parameters for the JQuery widget, if you want to change their default settings. Here's an example: 
```php
$data= CHtml::listData(Color::model()->findAll(), 'ID', 'Name');
$this->widget('ext.EchMultiSelect.EchMultiSelect', array(
	'model' => $model,
	'dropDownAttribute' => 'color',		
	'data' => $data,	
	'dropDownHtmlOptions'=> array(
		'style'=>'width:378px;',
	),
	'options' => array( 
		'header'=> Yii::t('EchMultiSelect.EchMultiSelect','Choose an Option!'),
		'minWidth'=>350,
		'position'=>array('my'=>'left bottom', 'at'=>'left top'),
		'filter'=>true,
	),
	'filterOptions'=> array(
		'width'=>150,
	),
));
```
Now the default “check all”, “uncheck all”, and “close” links in the header will be replaced with the specified text 'Choose an Option!, the widget will have a minimum width of 350px (instead of the default 225px), and the filter plugin will be applied.

Details about the available 'options' and 'filterOptions' for the jQuery UI MultiSelect Widget can be found on the [Project page](http://www.erichynds.com/jquery/jquery-ui-multiselect-widget/ ""). Here's a list of the options and their default values for quick reference:
```php
// 'options':
'header'=> true,
'height'=>175,
'minWidth'=>225,
'position'=>'',
'checkAllText' => Yii::t('EchMultiSelect.EchMultiSelect','Check all'),
'uncheckAllText' => Yii::t('EchMultiSelect.EchMultiSelect','Uncheck all'),
'selectedText' =>Yii::t('EchMultiSelect.EchMultiSelect','# selected'),
'selectedList'=>false,
'show'=>'',
'hide'=>'',
'autoOpen'=>false,
'noneSelectedText'=>'-- ' . Yii::t('EchMultiSelect.EchMultiSelect','Select Options') . ' --',
'multiple'=>true,
'classes'=>'',
'filter'=>false,
// 'filterOptions':
'label' => Yii::t('EchMultiSelect.EchMultiSelect','Filter:'),
'width'=>100,
'placeholder'=>Yii::t('EchMultiSelect.EchMultiSelect','Enter keywords'),
'autoReset'=>false,
```


**The filter plugin** is disabled by default. To enable it, you have to set the `filter` attribute to true. The `filterOptions` will have no effect, if `filter` is set to false, i.e. the filter plugin is disabled.

Note that the default options are already **translated** with Yii::t() where necessary.

For details about **the `position` option** see the [jQuery page for the Position utility](http://jqueryui.com/demos/position/#option-at ""). The default position of the Multiselect list `array('my'=>'left top', 'at'=>'left bottom')`.

###5. `import` widgets in config file to simplify calls

As an alternative, you can import the path of your widgets in your config file `protected/config/main.php`:
```php
'import'=>array(
	...
	'ext.EchMultiSelect.*',
	...
),
```
Note that `ext` is short for `application.extensions`. Now you can call this widget (and any other widgets you may have placed under `extensions/widgets/` ) with only its name, without specifying its path, i.e. like this:
```php
$this->widget('EchMultiSelect',...);
```

##Changes
* **Dec 17, 2012** (v1.3)
   * Added the changes suggested by jeremy in the comments, to make this widget work as a 'filter' in CGridView (added the `ajaxRefresh` and `buttonWidth` options). Thanks to [jeremy](http://www.yiiframework.com/user/1627/ "jeremy") for providing the code!
   * Added translation files. You can add your own translations to the folder `EchMultiSelect/messages`.
* **Mar 12, 2012** (v1.2)
   * Zipped the Widget-files directly, without putting them into an additional `EchMultiSelect` directory. That is: only directory structure changed. **No changes in the code!** (Reason of the change: The contents of the zip file are often extracted directly into the `extensions` directory, which resulted in the need to call the widget with `$this->widget('ext.widgets.EchMultiSelect.EchMultiSelect', array(...);`)
* **Feb 06, 2012**  (v1.1)
   * Added the ability to pass options to the MultiSelect Filter Widget via parameter 'filterOptions' (thanks to [sucotronic](http://www.yiiframework.com/forum/index.php?/user/62367-sucotronic/ "") for pointing this out in the forum).
   * Made sure 'multiple' attribute of drop down list is set to 'true', even if not explicitly set in 'dropDownHtmlOptions' (only if 'multiple'=>true in 'options' of course).
   * Fixed a bug regarding the 'name' parameter
  

##Resources

 * jQuery UI MultiSelect [Project Page](http://www.erichynds.com/jquery/jquery-ui-multiselect-widget/ "jQuery UI MultiSelect Project Page")
 * jQuery UI MultiSelect [Demo page](http://www.erichynds.com/examples/jquery-ui-multiselect-widget/demos/#filter "jQuery UI MultiSelect Demo Page")
 * jQuery UI MultiSelect [GitHub Page](https://github.com/ehynds/jquery-ui-multiselect-widget "jQuery UI MultiSelect GitHub Page")
 * EchMultiSelect Widget [Yii Extension Page](http://www.yiiframework.com/extension/echmultiselect/ "EchMultiSelect Yii Extension Page")
 * EchMultiSelect Widget [Forum Page](http://www.yiiframework.com/forum/index.php?/topic/28524-echmultiselect-widget/ "EchMultiSelect Forum Page")
 * EchMultiSelect Widget [GitHub Page](https://github.com/c-cba/ech-multi-select "EchMultiSelect GitHub Page")