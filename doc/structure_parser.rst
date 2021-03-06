****************
Structure parser
****************

Parsing data can be a difficult challenge. One way to reduce the challenge is to 
reduce the scope of what is being parsed. With *NestedText* you can delegate the 
parsing the of the structure and instead focus on parsing individual values 
given as strings.  A transforming validator like `Voluptuous 
<https://github.com/alecthomas/voluptuous>`_ can greatly simply the process.

To use *Voluptuous* you would create a schema and then apply the schema 
to the data. The schema details what fields are expected, and what what 
kind of values they should contain. Normally the schema is used to 
validate the data, but with a little extra plumbing the data can be 
transformed to the needed form.  The following is a very simple example 
(see :ref:`cryptocurrency holdings <cryptocurrency example>` for a more 
complete example).

In order for *Voluptuous* to convert the data to the desired type, a 
converter function is helpful:

.. code-block:: pycon

    >>> import voluptuous

    >>> def coerce(type, msg=None):
    ...     """Coerce a value to a type.
    ...
    ...     If the type constructor throws a ValueError, the value will be
    ...     marked as Invalid.
    ...     """
    ...     def f(v):
    ...         try:
    ...             return type(v)
    ...         except ValueError:
    ...             raise voluptuous.Invalid(msg or ('expected %s' % type.__name__))
    ...     return f

The next step is to define a schema that declares the expected types of 
the various fields in the configuration file. For example, imagine the 
configuration file has has three values, *name*, *value*, and 
*editable*, the first of which must be a string, the second a float, 
and the third a boolean that is specified using either 'yes' or 'no'. 
This can be done as follows:

.. code-block:: pycon

    >>> import nestedtext as nt

    >>> def to_bool(v):
    ...     try:
    ...         v = v.lower()
    ...         assert v in ['yes', 'no']
    ...         return v == 'yes'
    ...     except:
    ...         raise ValueError("expected 'yes' or 'no'.")

    >>> config = """
    ... name: volume
    ... value: 50
    ... editable: yes
    ... """

    >>> config_data = nt.loads(config)
    >>> print(config_data)
    {'name': 'volume', 'value': '50', 'editable': 'yes'}

    >>> schema = voluptuous.Schema(
    ...     dict(name=str, value=coerce(float), editable=coerce(to_bool))
    ... )

    >>> settings = schema(config_data)
    >>> print(settings)
    {'name': 'volume', 'value': 50.0, 'editable': True}

Notice that a dictionary that contains the expected types and 
conversion functions is passed to *Schema*. Then the raw configuration 
is parsed for structure by *NestedText*, and the resulting data 
structure is processed by the schema to and converted to its final form.
