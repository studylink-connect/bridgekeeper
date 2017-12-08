Defining Permissions
====================

In this tutorial, we'll be using a example app, an online stock management portal for shrubberies; we'll define some permissions for it in this section, then use them in views in the next section. It has a single app called ``shrubberies``, with a ``models.py`` looks something like this:

.. literalinclude:: ../testproject/shrubberies/models.py
    :caption: shrubberies/models.py

First, we'll define permissions for our ``Store`` model. We'll define them in ``permissions.py``, because that's where Bridgekeeper will automatically load them from.

:ref:`Permissions in Django <django:topic-authorization>` typically have names of the form ``appname.action_modelname``, where ``appname`` is the app name (``shrubbery`` in our case), ``modelname`` is the lowercased version of the model name, and ``action`` is one of ``create``, ``update`` or ``delete``. So, the permissions we want to create, we'll call ``shrubbery.create_store``, ``shrubbery.update_store`` and ``shrubbery.delete_store``.

In Bridgekeeper, permissions are made up of **predicates**, which you can think of as questions to ask about the user that is trying to gain access, and the objects they're trying to gain access to. For the ``Store`` model, we only want the administrators of our marketplace to be able to manipulate stores, so we can use the built-in :data:`~bridgekeeper.predicates.is_staff` predicate, which asks "Is the user a staff user?":

.. code-block:: python
    :caption: shrubberies/permissions.py

    from bridgekeeper.registry import registry
    from bridgekeeper.predicates import is_staff

    registry['shrubbery.create_store'] = is_staff
    registry['shrubbery.update_store'] = is_staff
    registry['shrubbery.delete_store'] = is_staff

We turn predicates into permissions by putting them into the :data:`~bridgekeeper.registry.registry`, which is a Python dictionary that maps permission names to their predicates. (It's actually a custom subclass of :class:`dict` with a few small changes, but you can treat it as if it's a normal dictionary anyway.)

These permissions are now fully working; if you wanted, you could skip right through to the next section to see how to use them in your views. Don't, though, because we've just scratched the surface.

.. _tutorial-ambient:

Ambient Predicates
::::::::::::::::::

Ambient predicates are predicates whose outcome is only affected by the user. We said earlier that predicates are *questions to ask about the user that is trying to gain access, and the objects they're trying to gain access to*; ambient predicates are *questions to ask about the user that is trying to gain access*, without regard for what they're accessing.

The built-in predicate :data:`~bridgekeeper.predicates.is_staff` is an ambient predicate, as are :data:`~bridgekeeper.predicates.is_authenticated`, :data:`~bridgekeeper.predicates.is_superuser` and :data:`~bridgekeeper.predicates.is_active`.

We can define our own, too, by using the :class:`~bridgekeeper.predicates.ambient` decorator to wrap a function that takes a user and returns a boolean:

.. code-block:: python
    :caption: shrubberies/predicates.py

    from bridgekeeper.predicates import ambient

    @ambient
    def is_apprentice(user):
        return user.profile.role == 'apprentice'

    @ambient
    def is_shrubber(user):
        return user.profile.role == 'shrubber'

If we wanted to restrict the ability to edit shrubberies in our app to only users that have the Shrubber role, we could write something like this:

.. code-block:: python
    :caption: shrubberies/permissions.py

    from .predicates import has_role

    registry['shrubbery.update_shrubbery'] = has_role('shrubber')