
Model Fields
============

.. php:class:: Field

Model uses objects to describe each of it's field. Therefore when you
create a model with 10 fields, it would populate at least 12 objects:

Model's object. One object for each field. DSQL object for a model.
Fields object play a very direct role in creating queries for the model
such as select, update and insert queries.

Normally you would rely on the standard Model Field class, which is
populated when you call ``addField()``, however you should know that
it's possible to extend fields significantly.

Field Meta-information
~~~~~~~~~~~~~~~~~~~~~~

One of he significant roles carried out by Model Field is collecting of
the information about the field. Information such as caption, type,
possible values, associated model is stored in Field's properties and
can be extended to hold even more information such as caption,
validation rules etc.

Query Generation
~~~~~~~~~~~~~~~~

Model will operate with internal DSQL object. It is however
responsibility for the field object to populate queries with field
information.

Basic Use
~~~~~~~~~

You would typically create a new field by calling :php:class:`Model::addField`
method. This method is a shortcut for ``add('Field', $name)``.
but should be used for standard field definition. If you are adding non-standard
field class (such as Field_File), you can use :php:class:`AbstractObject::add`.

Here is how we add few fields to our model::

    $this->addField('name');
    $this->addField('gender')->enum(array('m','f'));
    $this->addfield('language')->setValueList($languages);

There are several classes which extend “Field”, such as:

-  Field\_Reference - implements one-to-many relation field
-  Field\_Expression - implements a custom SQL query readonly field
-  Field\_Deleted - implements field used for soft-delete
-  filestore/Field\_File - implements field for referencing uploaded
   files
-  filestore/File\_Image - implements field for images and thumbnails

Some of them will automatically be used by the Model when you call a
dedicated method (``addExpression()``, ``hasOne()``), others can me
manually added using ``add()`` method.

The short\_name of the field object corresponds to the field name and
will be used for internal referencing as well as in result sets. If you
pass second argument to ``addField()``, then you can specify a different
actual database field.

::

    $this->addField('name','full_name');
    $field = $this->getElement('name');

Anything what was said in the AbstractObject section still applies to
the field, for example - you can remove fields from model:
$field->destroy();

Relational Model Fields and DSQL
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Since fields are associated with models and models know which table they
work with, you can use Fields of Relational Models inside SQL
expressions.

::

    $model->dsql()
        ->set($model->getElement('name'),'John')
        ->update();

It's highly advisable that if you use field of a model
:math:`model, you should use the query from ``\ model->dsql();\`

Definitions: type
~~~~~~~~~~~~~~~~~

Several methods of the field are acting to describe model type.

-  ``type()`` - will specify the primary type of the field. Default
   value is text, but could also be: int, date, money, boolean. Views
   will attempt to properly display recognized fields, for example, date
   will be converted to a regional format on Grid and will use
   date-picker inside a form.
-  ``display()`` - specifies the exact field / column class to use for
   this field. When Form displays fields it relies to objects extended
   from Form\_Field. By specifying value here, you can define how to
   display the value. ``$model->addField('pass')->display('password');``
   If you specify the array, you can define the particular value for
   each view: ``display(array('form'=>'password', 'grid'=>'text'));``
   You may also spec- ify fields from add-ons in the format of
   'addon/field\_name'.
-  ``allowHTML()`` - can be set to "true" if you want field to be able
   and store HTML. By default forms will strip HTML from the input for
   security. If you wish to display HTML in-line, you might need to use
   ``setHTML()`` method of the template engine.
-  ``mandatory()`` - setting this to true will present the field as a
   mandatory on forms.
-  ``defaultValue()`` - specified value will be set to the field by
   default.

You must remember, that properties of a model serve the purpose of
configuring Views to comply with your configuration. They will not
restrict or validate the actual model. For example - you can still
create model entry with a empty mandatory field.

If you wish to validate fields on the model level, you will need to use
beforeSave hook, this will have a system-wide access regardless of who
is accessing the model. Even if you create a validation like that, you
can still use model's DSQL to manually create a query which would bypass
the validation.


.. _actual fields:

Actual Fields
~~~~~~~~~~~~~

Typically there are many more fields in a model than you would need to
display on the forms. With actual fields, you can specify which fields
you need to display to user and in which order.

While actual fields primarily is a functionality of a respective view
(such as defining which columns are visible inside grid), it also has
impact on the query. Agile Toolkit models will never use "\*" to load
model data.

::

    $page->add('Grid')
        ->setModel('User',array('name','email'));

This code will display grid with only two fields in exactly the
specified order. Not always you would wan to specify a list of fields.
If field list is omitted, then model will attempt to determine which
fields to display based on number of flags.

-  ``system()`` - field will be loaded by model always , even if not
   present in actual fields. Setting system to true will also hide the
   field, but it can be un-hidden.
-  ``hidden()`` - field does not appear in the grid, form etc unless
   specifically added or requested by actual fields
-  ``editable()`` / ``visible()`` - overrides hidden but for only
   particular widgets. E.g. if field is hidden, but editable it will
   appear on the form, but not in grid. Set- ting editable(false) will
   hide field from form even if it’s not hidden. • ``readonly()`` - the
   field will appear on the editable form but will be displayed using
   read-only field.

These methods can accept "true", "false" or "**undefined**\ " value.

Grid-related attributes
~~~~~~~~~~~~~~~~~~~~~~~

Several attributes change how fields are displayed inside a grid or
filter:

-  ``searchable()`` - makes field visible in filter form.
-  ``sortable()`` - turns on sorting control on the grid column.

Value lists and foreign keys
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Some fields contain one value, but the value needs to be displayed
differently. For example it's typical to have 'm' for Male and 'f' for
Female.

-  ``listData()`` - Specify associative array used to decode the value
   of the field. Inside a Grid, the key value will be substituted with
   the string: ``array('m'=>'Male');`` Inside a form, the field will be
   presented as a drop-down.
-  ``enum()`` - specify array (without keys) if you only only want a
   drop-down but do not want value substitution:
   ``enum(array('small','medium','large'));``
-  ``emptyValue()`` - if your list value is not mandatory, it will allow
   user to select an empty value inside a drop-down. This method allows
   you to specify the value which will be presented to the user on that
   option: ``emptyValue('Select Size');``

Low-level properties
~~~~~~~~~~~~~~~~~~~~

There are some properties which are used on a low-level and allows to
change the way how field queries are created. Do not call those methods
directly unless you know what you are doing:

-  ``from()`` - specify joined table to which field belongs. Instead you
   can call ``addField()`` method on a join object.
-  ``actual()`` - used to change the actual name of the field inside a
   table. Using second argument to ``addField()`` allows you to specify
   this value.

Specifying additional attributes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You may specify additional attributes by calling ``setterGetter()``
method::

    $model->addField('bio')->setterGetter('hint','Type your bio here');

You would need to extend views to make them capable of interpreting your
additional parameters. You can also access properties directly if you
prefer it that way.

Creating Your Own Field Class
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In several situations, you would want to have your own field class, such
as:

-  You are willing to change many properties to a different default at
   the same time.
-  You would like to use custom behavior for query generations
-  You wish to place hooks inside the models

If you are willing to create your own field, be sure to extend it from
the "Field" class.

