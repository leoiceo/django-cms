#######
Wizards
#######

****************************
CMS Content Creation Wizards
****************************

Creating a CMS content creation wizard for your own module is fairly easy.

To begin, create a folder in the root level of your module called
`cms_wizards.py`. In this file, import Wizard as follows::

    from cms.wizards.wizard_base import Wizard
    from cms.wizards.wizard_pool import wizard_pool

Then, simply subclass Wizard, instantiate it, then register it. If you were to
do this for `MyApp`, it might look like this::

    # my_apps/forms.py

    from django import forms

    class MyAppWizardForm(forms.ModelForm):
        model = MyApp
        exclude = []


    # my_apps/cms_wizards.py

    from cms.wizards.wizard_base import Wizard
    from cms.wizards.wizard_pool import wizard_pool

    from .forms import MyAppWizardForm
    from .models import MyApp

    class MyAppWizard(Wizard):
        pass

    my_app_wizard = MyAppWizard(
        title="New MyApp",
        weight=200,
        form=MyAppWizardForm,
        description="Create a new MyApp instance",
    )

    wizard_pool.register(my_app_wizard)

That is it!

.. note::

    the module name `cms_wizards` is special, in that any such-named modules in
    your project's python path will automatically be loaded, triggering the
    registration of any wizards found in them. Wizards may be declared and
    registered in other modules, but they might not be automatically loaded.

The above example is using a ModelForm, but you can also use `forms.Form`.
In this case, you **must** provide the model class as another keyword argument
when you instantiate the Wizard object.

You must subclass `cms.wizards.wizard_base.Wizard` to use it. This is because
each wizard's uniqueness is determined by its class and module name.

The keyword arguments accepted for instantiating a Wizard object are:

    :title: The title of the wizard. This will appear in a large font size on
            the wizard "menu"
    :weight: The "weight" of the wizard when determining the sort-order.
    :form: The form to use for this wizard. This is mandatory, but can be
           subclassed from `django.forms.form` or `django.forms.ModelForm`.
    :model: If a Form is used above, this keyword argument must be supplied and
            should contain the model class. This is used to determine the unique
            wizard "signature" amongst other things.
    :template_name: An optional template can be supplied.
    :description: The description is optional, but if it is not supplied, the
                  CMS will create one from the pattern:
                  "Create a new «model.verbose_name» instance."

discovered
==========

wizard_pool includes a read-only property `discovered` which returns the Boolean
True if wizard-discovery has already occurred and False otherwise.


is_registered
=============

Sometimes, it may be necessary to check to see if a specific wizard has been
registered. To do this, simply call::

    value = wizard_pool.is_registered(«wizard»)


register
========

You may notice from the example above that the last line in the sample code is::

    wizard_pool.register(my_app_wizard)

This sort of thing should look very familiar, as a similar approach is used for
cms_apps, template tags and even Django's admin.

Calling the wizard pool's `register` method will register the provided wizard
into the pool, unless there is already a wizard of the same module and class
name. In this case, the register method will raise a
`cms.wizards.wizard_pool.AlreadyRegisteredException`.


unregister
==========

It may be useful to unregister wizards that have already been registered into
the pool. To do this, simply call::

    value = wizard_pool.unregister(«wizard»)

The value returned will be a Boolean. True if a wizard was successfully
unregistered or False otherwise.


get_entry
=========

If you would like to get a reference to a specific wizard in the pool, just call
`get_entry()` as follows::

    wizard = wizard_pool.get_entry(my_app_wizard)


get_entries
===========

`get_entries()` is useful if it is required to have a list of all registered
wizards. Typically, this is used to iterate over them all. Note that they will
be returned in the order of their "weight": smallest numbers for weight are
returned first.::

    for wizard in wizard_pool.get_entries():
        # do something with a wizard...
