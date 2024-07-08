
.. _odoosh-advanced-frequent_technical_questions:

============================
Frequent Technical Questions
============================

"Scheduled actions do not run at the exact time they were expected"
-------------------------------------------------------------------

On the Odoo.sh platform, we cannot guarantee an exact running time for scheduled actions.

This is due to the fact that there might be multiple customers on the same server, and we must guarantee a fair share of the server for every customer. Scheduled actions are therefore implemented slightly differently than on a regular Odoo server, and are run on a *best effort* policy.

.. warning::
    Do not expect any scheduled action to be run more often than every 5 min.

Are there "best practices" regarding scheduled actions?
-------------------------------------------------------

**Odoo.sh always limits the execution time of scheduled actions (*aka* crons).**
Therefore, you must keep this fact in mind when developing your own crons.

We advise that:

- Your scheduled actions should work on small batches of records.
- Your scheduled actions should commit their work after processing each batch;
  this way, if they get interrupted by the time-limit, there is no need to start over.
- Your scheduled actions should be
  `idempotent <https://stackoverflow.com/a/1077421/3332416>`_: they must not
  cause side-effects if they are started more often than expected.

Ip address modification?
------------------------

**Odoo.sh notifies instances on ip modification.**
When an instance has been moved for maintenance purposes an GET HTTP request in sent on the route /_odoo.sh/ip-change
with the new ip as the GET attribute. It allows you to decide what behavior you want within your production instance,
for example:

.. code-block:: python

    from odoo import http

    class MyController(http.Controller):

        @http.route('/_odoo.sh/ip-change', auth='public')
        def ip_change(self, ip=None):
            # Update an ir.config_paramater, send an email, notify another service, etc ...

The security is handled by the platform: access from the outside is blocked.
The route is only callable from within the platform.
