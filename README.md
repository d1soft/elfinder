# elFinder file manager for Yii 2

This extension integrates an [elFinder 2.1](http://studio-42.github.io/elFinder/) file manager into
[Yii framework 2.0](http://www.yiiframework.com). It is version with added editors src to Asset.

[![Latest Stable Version](https://img.shields.io/packagist/v/d1soft/yii2-elfinder.svg)](https://packagist.org/packages/d1soft/yii2-elfinder)
[![Total Downloads](https://img.shields.io/packagist/dt/d1soft/yii2-elfinder.svg)](https://packagist.org/packages/d1soft/yii2-elfinder)
[![License](https://img.shields.io/github/license/d1soft/yii2-elfinder.svg)](https://raw.githubusercontent.com/d1soft/yii2-elfinder/master/LICENSE)
[![Build Status](https://travis-ci.org/d1soft/yii2-elfinder.svg?branch=master)](https://travis-ci.org/d1soft/yii2-elfinder)

## Installation

Install extension through [composer](http://getcomposer.org/):

```
composer require d1soft/yii2-elfinder
```

## Usage

### Configure actions

For using elFinder you must create and configure controller. See complete example with actions for elFinder's connector,
`InputFile` widget, CKEditor `filebrowser*` options and TinyMCE `file_picker_callback` option:

```php
<?php

namespace app\controllers;

use d1soft\elfinder\CKEditorAction;
use d1soft\elfinder\ConnectorAction;
use d1soft\elfinder\InputFileAction;
use d1soft\elfinder\TinyMCEAction;
use Yii;
use yii\web\Controller;

class ElfinderController extends Controller
{
    public function actions()
    {
        return [
            'connector' => [
                'class' => ConnectorAction::className(),
                'options' => [
                    'roots' => [
                        [
                            'driver' => 'LocalFileSystem',
                            'path' => Yii::getAlias('@webroot') . DIRECTORY_SEPARATOR . 'uploads',
                            'URL' => Yii::getAlias('@web') . '/uploads/',
                            'mimeDetect' => 'internal',
                            'imgLib' => 'gd',
                            'accessControl' => function ($attr, $path) {
                                // hide files/folders which begins with dot
                                return (strpos(basename($path), '.') === 0) ?
                                    !($attr == 'read' || $attr == 'write') :
                                    null;
                            },
                        ],
                    ],
                ],
            ],
            'input' => [
                'class' => InputFileAction::className(),
                'connectorRoute' => 'connector',
            ],
            'ckeditor' => [
                'class' => CKEditorAction::className(),
                'connectorRoute' => 'connector',
            ],
            'tinymce' => [
                'class' => TinyMCEAction::className(),
                'connectorRoute' => 'connector',
            ],
        ];
    }
}
```

Reed [elFinder docs](https://github.com/Studio-42/elFinder/wiki/Connector-configuration-options-2.1) to configure
connector options.

### InputFile widget

Example of `InputFile` widget with enabled mime filter and preview:

```php
<?= d1soft\elfinder\InputFile::widget([
    'name' => 'attributeName',
    'clientRoute' => 'elfinder/input',
    'filter' => ['image'],
    'preview' => function ($value) {
        return yii\helpers\Html::img($value, ['width' => 200]);
    },
]) ?>
```

*Note 1:* Filter option is using to display only certain files based on their mime type. Check `onlyMimes` option
in [elFinder docs](https://github.com/Studio-42/elFinder/wiki/Client-configuration-options-2.1#onlyMimes).

*Note 2:* Preview displays only predefined (saved earlier) input value and not updating on the fly after new selection.

If you want to use the `InputFile` widget in `ActiveForm`, it can be done like this:

```php
<?= $form->field($model, 'attributeName')
    ->widget(d1soft\elfinder\InputFile::className(), [
        'clientRoute' => 'elfinder/input',
    ]) ?>
```

Using textarea instead text input (can be useful with enabled multiple selection):

```php
<?= d1soft\elfinder\InputFile::widget([
    'name' => 'attributeName',
    'clientRoute' => 'elfinder/input',
    'textarea' => true,
    'textareaRows' => 3, // default is 5
]) ?>
```

Enable multiple selection to select more then one file in one input:

```php
<?= d1soft\elfinder\InputFile::widget([
    'name' => 'attributeName',
    'clientRoute' => 'elfinder/input',
    'multiple' => true,
]) ?>
```

Default paths separator for text input is comma and newline character for textarea.
You can change them in `InputFileAction` configuration:

```php
class ElfinderController extends Controller
{
    public function actions()
    {
        return [
            // ...
            'input' => [
                'class' => InputFileAction::className(),
                'connectorRoute' => 'connector',
                'separator' => ',',
                'textareaSeparator' => '\n', // newline character in javascript
            ],
            // ...
        ];
    }
}
```

### Integration with CKEditor

For using elFinder with CKEditor widget (like [this one](https://github.com/d1soft/yii2-ckeditor)) you need to
specify options `filebrowserBrowseUrl` and (or) `filebrowserImageBrowseUrl`:

```php
<?= d1soft\ckeditor\CKEditor::widget([
    'name' => 'attributeName',
    'clientOptions' => [
        // ...
        'filebrowserBrowseUrl' => yii\helpers\Url::to(['elfinder/ckeditor']),
        'filebrowserImageBrowseUrl' => yii\helpers\Url::to(['elfinder/ckeditor', 'filter' => 'image']),
    ],
]) ?>
```

*Note:* For `filebrowserImageBrowseUrl` we use filter query param to display only images.

### Integration with TinyMCE 4

For using elFinder with TinyMCE 4 widget (like [this one](https://github.com/d1soft/yii2-tinymce)) you need to
specify option `file_picker_callback`:

```php
<?= d1soft\tinymce\TinyMce::widget([
    'name' => 'attributeName',
    'clientOptions' => [
        // ...
        'file_picker_callback' => d1soft\elfinder\TinyMCE::getFilePickerCallback(['elfinder/tinymce']),
    ],
]) ?>
```

*Note:* option `file_picker_callback` available since 4.1.0 version of TinyMCE js plugin.

With second param in `getFilePickerCallback()` you can set additional `tinymce.WindowManager.open` settings:

```php
TinyMCE::getFilePickerCallback(['elfinder/tinymce'], ['width' => 1200, 'height' => 600])
```

### Standalone file manager

Add `ElFinder` widget to any view:

```php
<?= d1soft\elfinder\ElFinder::widget([
    'connectorRoute' => ['elfinder/connector'],
    'settings' => [
        'height' => 640,
    ],
    'buttonNoConflict' => true,
]) ?>
```

*Note:* If you are using Bootstrap 3 enable `buttonNoConflict` option to resolve conflict between
Bootstrap and jQuery UI buttons.

## Links

* [elFinder Wiki](https://github.com/Studio-42/elFinder/wiki)
* [CKEditor widget](https://github.com/d1soft/yii2-ckeditor) by [d1soft](https://github.com/d1soft)
* [TinyMCE widget](https://github.com/d1soft/yii2-tinymce) by [d1soft](https://github.com/d1soft)
* [CKEditor widget](https://github.com/2amigos/yii2-ckeditor-widget) by [2amigos](https://github.com/2amigos)
* [TinyMCE widget](https://github.com/2amigos/yii2-tinymce-widget) by [2amigos](https://github.com/2amigos)
* [Yii framework site](http://www.yiiframework.com)
