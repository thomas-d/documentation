=========================
Chapter 5: Business logic
=========================

tmp

.. todo: constraints, defaults, onchanges, computes
.. todo: model actions ("assign myself as salesperson" action, "view offers" statbutton)
.. todo: salesperson_id = fields.Many2one(default=lambda self: self.env.user)
.. todo: explain the env (self.env.cr, self.env.uid, self.env.user, self.env.context, self.env.ref(xml_id), self.env[model_name])
.. todo: explain the thing about `self`
.. todo: explain magic commands
.. todo: copy=False on some fields

.. todo: introduce lambda functions for defaults

There is a problem with the way we defined our Date fields: their default value relies on
:code:`fields.Date.today()` or some other static method. When the code is loaded into memory, the
date is computed once and reused for all newly created records until the server is shut down. You
probably didn't notice it, unless you kept your server running for several days, but it would be
much more visible with Datetime fields, as all newly created records would share the same timestamp.

That's where lambda functions come in handy. As they generate an anonymous function each time
they're evaluated at runtime, they can be used in the computation of default field values to return
an updated value for each new record.

.. todo: odoo-bin shell section

----

.. todo: add incentive for chapter 6
