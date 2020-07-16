# Page constructor
this constructor allows you to create web pages with unique blocks

## Code

### Controller
```` 
public function actionWidget($id, $section_id = false, $key = false)
    {
        $model = $this->findModel($id);

        if (Yii::$app->request->isPost) {

            $new_data = [];

            if (Yii::$app->request->post('widget')) {

                $data = Yii::$app->request->post('widget');

                foreach ($data as $k => $item) {
                    if (!isset($item['delete'])) {
                        $new_data[$k] = $item;
                    }
                }
            }
            $model->setData($model::ADDITIONAL_DATA_ITEMS, $new_data);

            $model->save();

            Yii::$app->session->setFlash('success', 'Changes saved.');
            return $this->redirect(['widget', 'id' => $model->id]);
        }

        return $this->render('widget', [
            'model' => $model,
            'section_key' => $key,
            'section_id' => $section_id
        ]);
    }
````

### View

````
<?php


use backend\modules\core\models\Page as CurrentModel;
use backend\modules\faq\models\Faq;
use backend\modules\section\models\SectionItem;
use backend\modules\slider\models\Slider;
use kartik\select2\Select2;
use yii\helpers\Html;
use backend\modules\core\models\Page;
use yii\helpers\StringHelper;
use yii\helpers\Url;
use yii\widgets\ActiveForm;
use yii\widgets\Pjax;

\backend\widgets\SortWidget::widget(['className' => CurrentModel::className()]);

$this->title = 'Update: ' . $model->title;
$this->params['breadcrumbs'][] = ['label' => 'Pages', 'url' => ['index']];
$this->params['breadcrumbs'][] = ['label' => $model->title, 'url' => ['view', 'id' => $model->id]];
$this->params['breadcrumbs'][] = 'Update';

$faq = Faq::find()->where(['status' => Faq::STATUS_ACTIVE, 'page_id' => $model->id])->orderBy('position')->all();

$items = $model->getData(Page::ADDITIONAL_DATA_ITEMS) ?: [];
$add = Yii::$app->request->get('add-items');
$form = ActiveForm::begin(['options' => ['id'=> 'form-common-info']]);

?>


<div class="page-update">

    <?= $this->render('_submenu', [
        'model' => $model
    ]) ?>
    <?php Pjax::begin(['id' => 'content-list']) ?>
    <div class="col-md-12">
        <table class="table table-custom dataTable no-footer"><thead>
            <tr>

                <th></th>
                <th>Виджет</th>
                <th >Секция</th>
                <th>Фон</th>
                <th>Удалить</th>
            </tr>
            </thead>
            <tbody class="ui-sortable">
            <?php foreach ($items as $key => $item) : ?>
            <tr data-key="2">
                <td class="sort-item ui-sortable-handle" style="width: 10px;">
                    <i class="fa fa-arrows-alt"> </i>
                </td>
                <td><?=  Html::dropDownList('widget['.$key.'][widget]', $item['widget'], SectionItem::getTypeList()); ?></td>
                <td>
                    <?php
                    $section = false;
                    if ($section_key == $key) {
                        $section = SectionItem::findOne($section_id);
                     } elseif ($item['section_id']) {
                        $section = SectionItem::findOne($item['section_id']);
                    }
                    ?>
                    <?php if ($section): ?>
                        <input type="hidden" name="widget[<?= $key ?>][section_id]" value="<?=  $section->id ?>">
                        <input type="hidden" name="widget[<?= $key ?>][alias]" value="<?= $section->alias ?>">
                        <a class="btn btn-primary modalButton" href="<?=Url::to(['/section/section-item/update',
                            'section_id'=>$model->id,
                            'id' => $section->id,
                            'widget_id' => $item['widget']
                        ]) ?>">
                            Редактировать: <?= StringHelper::truncate(strip_tags($section->title), 30 )?>
                        </a>
                    <?php else: ?>
                        <a class="btn btn-success modalButton" href="<?=Url::to(['/section/section-item/create',
                            'section_id'=>$model->id,
                            'key'=>$key,
                            'widget_id' => $item['widget']
                        ]) ?>">
                            Добавить секцию
                        </a>
                    <?php endif; ?>

                    <?php if ($item['widget'] == 18): // FAQ ?>
                        <br>
                        <br>
                        <div id="faq">
                            <?php foreach ($faq as $record): ?>

                                    <div class="faq-item"  style="
                                                                display: flex;
                                                                align-items: flex-start;
                                                            ">
                                        <a style="cursor: pointer" ><i class="glyphicon glyphicon-trash"></i> </a>
                                        <input type="text" name="faq[<?= $record->id ?>][title]" value="<?= $record->title ?>" style="width: 390px">
                                        <div><?= \vova07\imperavi\Widget::widget(['name' => 'faq[' . $record->id . '][text]',
                                            'value' =>  $record->text,
                                            'settings' => [
                                                'lang' => 'ru',
                                                'minHeight' => 100,
                                                'plugins' => [
                                                    'clips',
                                                    'fullscreen',
                                                ],
                                                'clips' => [
                                                    ['Lorem ipsum...', 'Lorem...'],
                                                    ['red', '<span class="label-red">red</span>'],
                                                    ['green', '<span class="label-green">green</span>'],
                                                    ['blue', '<span class="label-blue">blue</span>'],
                                                ],
                                            ],]) ?></div>
                                    </div>
                            <?php endforeach; ?>
                        </div>

                        <?= Html::a('<i class="fa fa-plus" aria-hidden="true"></i>', ['widget', 'id' => $model->id,  'add-items' => 'colum'], ['class' => 'btn btn-success btn-sm', 'id' => 'faq-btn']) ?>
                    <?php endif; ?>

                    <?php if ($item['widget'] == 9) { // Calendar
                       echo Select2::widget([
                            'name' => 'widget['.$key.'][calendar][]',
                            'value' => $item['calendar'],
                            'data' => \backend\modules\event\models\Event::getEventAll(true),
                            'options' => ['multiple' => true, 'placeholder' => 'Select states ...']
                        ]);
                    } ?>

                    <?php if (in_array($item['widget'], [19,16,11,6,23,26])): // With sliders ?>
                        <?=  Html::dropDownList('widget['.$key.'][slider]', $item['slider'], Slider::getSliderAll(true)); ?>
                        <?= Html::a('Перейти в слайдеры', ['/slider/default/index', 'page_id' => $model->id], ['class' => 'btn btn-success btn-sm']) ?>

                    <?php endif; ?>
                    <?=  Html::dropDownList('widget['.$key.'][form]', $item['form'], \backend\modules\custom_form\models\Form::getFormAll(true)); ?>
                </td>

                <td><input type="checkbox" name="widget[<?= $key ?>][color]" <?= $item['color'] ? 'checked' : '' ?>></td>
                <td><a href="#" class="widget-delete-item" title="Delete" ><i class="glyphicon glyphicon-trash"></i> </a></td>
            </tr>

            <?php endforeach; ?>
            <?php if ($add): ?>
                <?php $key = max(array_keys($items)) + 1; ?>
                <tr data-key="2">
                    <td class="sort-item ui-sortable-handle" style="width: 10px;">
                        <i class="fa fa-arrows-alt"> </i>
                    </td>
                    <td>
                        <input type="hidden" name="widget[<?= $key ?>][section_id]" >
                        <?=  Html::dropDownList('widget['.$key.'][widget]', $item['widget'], SectionItem::getTypeList()); ?>
                    </td>
                    <td><a class="btn btn-success modalButton" href="<?=Url::to(['/section/section-item/create',
                            'section_id'=>$model->id,
                            'widget_id' => $item['widget']
                        ]) ?>">
                            Добавить секцию
                        </a></td>
                    <td><?= Html::checkbox( 'widget['.$key.'][color]', '') ?></td>
                    <td><a href="#" class="widget-delete-item" title="Delete" ><i class="glyphicon glyphicon-trash"></i> </a></td>
                </tr>
                <?=$this->registerJs("
    $('select').on('change', function() {
        $('#save').click();
    }); 
", \yii\web\View::POS_READY) ?>

            <?php else:; ?>
                <hr>
                <?= Html::a('<i class="fa fa-plus" aria-hidden="true"></i> Add item', ['widget', 'id' => $model->id,  'add-items' => 'colum'], ['class' => 'btn btn-success btn-sm']) ?>
            <?php endif; ?>
             </tbody></table>
        <hr>
        <div class="form-group">
            <?= Html::submitButton('Save', ['class' => $model->isNewRecord ? 'btn btn-success' : 'btn btn-primary', 'id' => 'save']) ?>
        </div>
    </div>
    <?php Pjax::end() ?>
</div>
<?php ActiveForm::end();?>
<?=$this->registerJs("
    $('#menu-absolute_url').on('click', function() {
        if ($(this).prop('checked')) {
           $('#block-url').removeClass('hidden');
        } else {
            $('#block-url').addClass('hidden');
        }
    }); 
", \yii\web\View::POS_READY) ?>

<?=$this->registerJs("
    $('select').on('change', function() {
        $('#save').click();
    }); 
", \yii\web\View::POS_READY) ?>

<?=$this->registerJs("
   
   
    $('#faq-btn').on('click', function(e) {
        e.preventDefault();
        $('#faq').append('<div class=\"faq-item\">'+
                            '<a style=\"cursor: pointer\" ><i class=\"glyphicon glyphicon-trash\"></i> </a>'+
                            '<input type=\"text\" name=\"faq[0][title]\" style=\"width: 390px\">  <textarea name=\"faq[0][text]\" style=\" width: 373px\"></textarea>'+
                        '</div>');
        $(this).remove();
    });
    
     $('.faq-item .glyphicon-trash').on('click', function(e) {
        e.preventDefault();
        $(this).closest('.faq-item').html('');
    });
", \yii\web\View::POS_READY) ?>
````

### Image
![image](https://user-images.githubusercontent.com/68379812/87681122-f6e57780-c786-11ea-881a-e2223ca73b4b.png)
