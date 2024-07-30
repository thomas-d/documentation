==================================
Chapter 4: Extend the model family
==================================

In Odoo, data rarely exists in isolation. The true power of an application lies in its ability to
connect and relate different pieces of information. In this chapter, we'll explore the three
fundamental types of model relationships in Odoo: Many2One, One2Many, and Many2Many. Mastering these
connections will allow us to create rich, interconnected data structures that will form the backbone
of our real estate application.

.. _tutorials/server_framework_101/module_structure:

Module structure
================

As our `real_estate` module grows, you may notice that we've already created a dozen files for just
one model, along with its menu items, actions and views. With more models on the horizon, our module
directory could quickly become cluttered. To address this potential issue, Odoo provides **module
structure guidelines** that offer several benefits:

- **Improved maintainability**: A well-organized directory structure makes it easier to navigate the
  module and locate specific files.
- **Scalability**: Proper organization prevents the module from becoming cluttered as it grows in
  complexity and size.
- **Collaboration**: A standardized structure facilitates easier understanding among contributors
  and ensures easier integration with the Odoo ecosystem.

.. seealso::
   :ref:`Coding guidelines on module directories
   <contributing/coding_guidelines/module_structure/directories>`

.. example::
   Let's consider a possible structure for our example `product` module:

   .. code-block:: text

      product/
      │
      ├── data/
      │   └── product_data.xml
      │
      ├── models/
      │   ├── __init__.py
      │   └── product.py
      │
      ├── security/
      │   └── ir.model.access.csv
      │
      ├── views/
      │   ├── actions.xml
      │   ├── menus.xml
      │   └── product_views.xml
      │
      ├── static/
      │   ├── description/
      │   │   └── icon.png
      │   │
      │   └── img/
      │       ├── coffee_table.png
      │       └── t_shirt.png
      │
      ├── __init__.py
      └── __manifest__.py

   .. note::

      - The :file:`models` directory contains its own :file:`__init__.py` file, simplifying Python
        imports. The root :file:`__init__.py` file imports the :file:`models` Python package, which
        in turns imports individual model files.
      - Security-related files, such as :file:`ir.model.access.csv`, are placed in the dedicated
        :file:`security` directory.
      - UI files (:file:`actions.xml`, :file:`menus.xml`, and view definitions) are organized within
        the :file:`views` directory.
      - The app icon has resides in :file:`static/description`, while other image assets are stored
        in :file:`static/img`.
      - The :file:`__init__.py` and :file:`__manifest__.py` files remain in the module's root
        directory.

.. exercise::

   Restructure the `real_estate` module according to the guidelines.

   .. tip::
      Use `[CLN]` for your :ref:`commit message tag
      <contributing/git_guidelines/commit_tag_module>`.

.. spoiler:: Solution

   .. code-block:: text

      real_estate/
      │
      ├── data/
      │   └── real_estate_property_data.xml.xml
      │
      ├── models/
      │   ├── __init__.py
      │   └── real_estate_property.py
      │
      ├── security/
      │   └── ir.model.access.csv
      │
      ├── views/
      │   ├── actions.xml
      │   ├── menus.xml
      │   └── real_estate_property_views.xml
      │
      ├── static/
      │   ├── description/
      │   │   └── icon.png
      │   │
      │   └── img/
      │       ├── country_house.png.png
      │       ├── loft.png
      │       └── mixed_use_commercial.png.png
      │
      ├── __init__.py
      └── __manifest__.py

   .. code-block:: python
      :caption: `models/__init__.py`

      from . import real_estate_property

   .. code-block:: python
      :caption: `__init__.py`
      :emphasize-lines: 1

      from . import models

   .. code-block:: xml
      :caption: `data/real_estate_property_data.xml`
      :emphasize-lines: 3,9,15

      <record id="real_estate.country_house" model="real.estate.property">
          [...]
          <field name="image" type="base64" file="real_estate/static/img/country_house.png"/>
          [...]
      </record>

      <record id="real_estate.loft" model="real.estate.property">
          [...]
          <field name="image" type="base64" file="real_estate/static/img/loft.png"/>
          [...]
      </record>

      <record id="real_estate.mixed_use_commercial" model="real.estate.property">
          [...]
          <field name="image" type="base64" file="real_estate/static/img/mixed_use_commercial.png"/>
          [...]
      </record>

   .. code-block:: python
      :caption: `__manifest__.py`
      :emphasize-lines: 2-11

      'data': [
          # Model data
          'data/real_estate_property_data.xml',

          # Security
          'security/ir.model.access.csv',

          # Views
          'views/actions.xml',
          'views/menus.xml',  # Depends on `actions.xml`
          'views/real_estate_property_views.xml',
      ],

.. _tutorials/server_framework_101/many2one:

Many-to-one
===========

As promised at the end of :doc:`the previous chapter <03_build_user_interface>`, we'll now expand
our app's capabilities by adding new models to manage additional information. This expansion
naturally leads us to an important question: How will our `real.estate.property` model connect to
these new models?

In relational databases, including Odoo's, **many-to-one** relationships play a crucial role. These
relationships allow you to link *multiple* records in one model to a *single* record in another
model. In Odoo, many-to-one relationships are established by adding a `Many2one` field to the model
representing the *many* side of the relationship. In practice, the field is represented by a
`foreign key <https://en.wikipedia.org/wiki/Foreign_key>`_ that references the ID of the connected
record. By convention, `Many2one` field names end with the `_id` suffix, indicating that they store
the referenced record's ID.

.. seealso::
   :ref:`Reference documentation for Many2one fields <reference/fields/many2one>`

.. example::
   In the example below, the `Selection` field of the `product` model is replaced by a `Many2one`
   field to create a more flexible and scalable model structure.

   .. code-block:: py

      from odoo import fields, models


      class Product(models.Model):
          _name = 'product'
          _description = "Storable Product"

          [...]
          category_id = fields.Many2one(
              string="Category", comodel_name='product.category', ondelete='restrict', required=True
          )

      class ProductCategory(models.Model):
          _name = 'product'
          _category = "Product Category"

          name = fields.Char(string="Name")

   .. note::

      - The relationship only needs to be declared on the *many* side to be established.
      - The `ondelete` argument on the `Many2one` field defines what happens when the referenced
        record is deleted.

In our real estate app, we currently have a fixed set of property types. To increase flexibility,
let's replace the current `type` field with a many-to-one relationship to a separate model for
managing property types.

.. exercise::

   #. Create a new `real.estate.property.type` model.

      - Update the :file:`ir.model.access.csv` file to grant all database administrators access to
        the model.
      - Replace the dummy :guilabel:`Settings` menu item with a new :menuselection:`Configuration
        --> Property Types` menu item.
      - Create a window action to browse property types only in list view.
      - Create the list view for property types.
      - In a data file, describe at least as many default property types as the `type` field of the
        `real.estate.property` model supports.

   #. Replace the `type` field on the `real.estate.property` model by a many-to-one relationship to
      the `real.estate.property.type` model. Prevent deleting property types if a property
      references them.

   .. tip::

      - As the window action doesn't allow opening property types in form view, clicking the
        :guilabel:`New` button does nothing. To allow editing records in-place, rely on the
        reference documentation for :ref:`root attributes of list views
        <reference/view_architectures/list/root>`
      - The server will throw an error at start-up because it can't require a value for the new,
        currently empty field. To avoid fixing that manually in the database, run the command
        :command:`dropdb tutorials` to delete the database and start from scratch.

.. spoiler:: Solution

   .. code-block:: python
      :caption: `real_estate_property_type.py`

      from odoo import fields, models
      from odoo.tools import date_utils


      class RealEstatePropertyType(models.Model):
          _name = 'real.estate.property.type'
          _description = "Real Estate Property Type"

          name = fields.Char(string="Name", required=True)

   .. code-block:: py
      :caption: `__init__.py`
      :emphasize-lines: 2

      from . import real_estate_property
      from . import real_estate_property_type

   .. code-block:: csv
      :caption: `ir.model.access.csv`
      :emphasize-lines: 3

      id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
      real_estate_property_system,real.estate.property.system,model_real_estate_property,base.group_system,1,1,1,1
      real_estate_property_type_system,real.estate.property.type.system,model_real_estate_property_type,base.group_system,1,1,1,1

   .. code-block:: xml
      :caption: `menus.xml`
      :emphasize-lines: 3-9

      <menuitem id="real_estate.root_menu"> <!-- truncated -->
          <menuitem id="real_estate.properties_menu"/> <!-- truncated -->
          <menuitem id="real_estate.configuration_menu" name="Configuration" sequence="20">
              <menuitem
                  id="real_estate.property_types_menu"
                  name="Property Types"
                  action="real_estate.views_property_types_action"
              />
          </menuitem>
      </menuitem>

   .. code-block:: xml
      :caption: `actions.xml`

      <record id="real_estate.views_property_types_action" model="ir.actions.act_window">
          <field name="name">Property Types</field>
          <field name="res_model">real.estate.property.type</field>
          <field name="view_mode">tree</field>
      </record>

   .. code-block:: xml
      :caption: `real_estate_property_type_views.xml`

      <?xml version="1.0" encoding="utf-8"?>
      <odoo>

          <record id="real_estate.property_type_list" model="ir.ui.view">
              <field name="name">Property Type List</field>
              <field name="model">real.estate.property.type</field>
              <field name="arch" type="xml">
                  <tree editable="bottom">
                      <field name="name"/>
                  </tree>
              </field>
          </record>

      </odoo>

   .. code-block:: xml
      :caption: `real_estate_property_type_data.xml`

      <?xml version="1.0" encoding="utf-8"?>
      <odoo>

          <record id="real_estate.type_house" model="real.estate.property.type">
              <field name="name">House</field>
          </record>

          <record id="real_estate.type_apartment" model="real.estate.property.type">
              <field name="name">Apartment</field>
          </record>

          <record id="real_estate.type_office" model="real.estate.property.type">
              <field name="name">Office Building</field>
          </record>

          <record id="real_estate.type_retail" model="real.estate.property.type">
              <field name="name">Retail Space</field>
          </record>

          <record id="real_estate.type_warehouse" model="real.estate.property.type">
              <field name="name">Warehouse</field>
          </record>

      </odoo>

   .. code-block:: py
      :caption: `__manifest__.py`
      :emphasize-lines: 3,4,8

      'data': [
          # Model data
          'data/real_estate_property_type_data.xml',
          'data/real_estate_property_data.xml',  # Depends on `real_estate_property_type_data.xml`
          [...]
          # Views
          [...]
          'views/real_estate_property_type_views.xml',
      ],

   .. code-block:: py
      :caption: `real_estate_property.py`

      type_id = fields.Many2one(
          string="Type", comodel_name='real.estate.property.type', ondelete='restrict', required=True
      )

   .. code-block:: xml
      :caption: `real_estate_property_views.xml`
      :emphasize-lines: 5,14,27

      <record id="real_estate.property_list" model="ir.ui.view">
          [...]
              <tree>
                  [...]
                  <field name="type_id"/>
                  [...]
              </tree>
          </field>
      </record>

      <record id="real_estate.property_form" model="ir.ui.view">
          [...]
                          <group string="Listing Information">
                              <field name="type_id"/>
                              <field name="selling_price"/>
                              <field name="availability_date"/>
                              <field name="active"/>
                          </group>
          [...]
      </record>

      <record id="real_estate.property_search" model="ir.ui.view">
          [...]
              <search>
                  [...]
                  <filter name="group_by_state" context="{'group_by': 'state'}"/>
                  <filter name="group_by_type" context="{'group_by': 'type_id'}"/>
              </search>
          </field>
      </record>

   .. code-block:: xml
      :caption: `real_estate_property_data.xml`
      :emphasize-lines: 3,9,15

      <record id="real_estate.country_house" model="real.estate.property">
          [...]
          <field name="type_id" ref="real_estate.type_house"/>
          [...]
      </record>

      <record id="real_estate.loft" model="real.estate.property">
          [...]
          <field name="type_id" ref="real_estate.type_apartment"/>
          [...]
      </record>

      <record id="real_estate.mixed_use_commercial" model="real.estate.property">
          [...]
          <field name="type_id" ref="real_estate.type_retail"/>
          [...]
      </record>

.. _tutorials/server_framework_101/generic_models:

Generic models
--------------

In the previous exercise, we created a many-to-many relationship with a custom model within the
`real_estate` module. However, Odoo provides several generic models that can extend your app's
capabilities without defining new models. These generic models are part of the default `base` module
and are typically prefixed with `res` or `ir`.

Two frequently used models in Odoo are:

- `res.users`: Represents user accounts in the database. They determine access rights to records and
  can be `internal` (have access to the backend), `portal` (have access to the portal, e.g., to view
  their invoices), or `public` (not logged in).
- `res.partner`: Represents physical or legal entities. They can be a company, an individual, or a
  contact address.

.. seealso::
   `The list of generic models in the base module <{GITHUB_PATH}/odoo/addons/base/models>`_

To make our real estate properties more informative, let's add two pieces of information: the seller
of the property and the salesperson managing the property.

.. exercise::

   #. Add the following fields to the `real.estate.property` model:

      - Seller (required): The person putting their property on sale; it be any individual.
      - Salesperson: The employee of the real estate agency overseeing the sale of the property.

   #. Modify the form view of properties to include a notebook component. The property description
      should be in the first tab, and the two new fields should be in the second tab.

   .. tip::
      You don't need to define any new UI component to browse the seller you assigned to your
      default properties! Just go to :menuselection:`Apps` and install the :guilabel:`Contacts` app.

.. spoiler:: Solution

   .. code-block:: python
      :caption: `real_estate_property.py`
      :emphasize-lines: 1-2

      seller_id = fields.Many2one(string="Seller", comodel_name='res.partner', required=True)
      salesperson_id = fields.Many2one(string="Salesperson", comodel_name='res.users')

   .. code-block:: xml
      :caption: `real_estate_property_views.xml`
      :emphasize-lines: 3-18

      <record id="real_estate.property_form" model="ir.ui.view">
          [...]
                      <notebook>
                          <page string="Description">
                              <field
                                  name="description"
                                  placeholder="Write a description about this property."
                              />
                          </page>
                          <page string="Other Info">
                              <group>
                                  <group>
                                      <field name="seller_id"/>
                                      <field name="salesperson_id"/>
                                  </group>
                              </group>
                          </page>
                      </notebook>
                  </sheet>
              </form>
          </field>
      </record>

   .. code-block:: xml
      :caption: `res_partner_data.xml`

      <?xml version="1.0" encoding="utf-8"?>
      <odoo>

          <record id="real_estate.bafien_carpink" model="res.partner">
              <field name="name">Bafien Carpink</field>
          </record>

          <record id="real_estate.antony_petisuix" model="res.partner">
              <field name="name">Antony Petisuix</field>
          </record>

          <record id="real_estate.amyfromthevideos" model="res.partner">
              <field name="name">AmyFromTheVideos</field>
          </record>

      </odoo>

   .. code-block:: xml
      :caption: `real_estate_property_data.xml`
      :emphasize-lines: 3,8,13

      <record id="real_estate.country_house" model="real.estate.property">
          [...]
          <field name="seller_id" ref="real_estate.amyfromthevideos"/>
      </record>

      <record id="real_estate.loft" model="real.estate.property">
          [...]
          <field name="seller_id" ref="real_estate.antony_petisuix"/>
      </record>

      <record id="real_estate.mixed_use_commercial" model="real.estate.property">
          [...]
          <field name="seller_id" ref="real_estate.bafien_carpink"/>
      </record>

   .. code-block:: py
      :caption: `__manifest__.py`
      :emphasize-lines: 3,5,6

      'data': [
          # Model data
          'data/res_partner_data.xml',
          'data/real_estate_property_type_data.xml',
          # Depends on `res_partner_data.xml`, `real_estate_property_type_data.xml`
          'data/real_estate_property_data.xml',
          [...]
      ],

.. _tutorials/server_framework_101/one2many:

One-to-many
===========

One2Many fields are the inverse of Many2One fields. They allow you to display and manage multiple related records from the "one" side of the relationship. Let's add a One2Many field to our real.estate.property.type model to show all properties of a certain type:

By convention, one2many fields have the _ids suffix.

.. seealso::
   :ref:`Reference documentation for One2many fields <reference/fields/one2many>`

.. exercise::

   #. add `real.estate.offer` model with menu item, action, and list and form views (search?)
   #. no menu or action required
   #. add Many2one and One2many fields to connect to the `real.estate.property` model
   #. add the field to the form view of properties in a new Offers notebook page

.. spoiler:: Solution

   .. code-block:: python
      :caption: `__manifest__.py`
      :emphasize-lines: 1

      todo

.. exercise::

   #. add M2O field on the `real.estate.offer` model to the `res.partner` model

.. spoiler:: Solution

   .. code-block:: python
      :caption: `__manifest__.py`
      :emphasize-lines: 1

      todo

.. _tutorials/server_framework_101/many2many:

Many-to-many
============

Many2Many relationships allow for complex associations where multiple records from one model can be linked to multiple records from another model. Let's add a Many2Many field to our real.estate.property model to associate multiple tags with each property:

By convention, many2many fields have the _ids suffix.

.. seealso::
   :ref:`Reference documentation for Many2many fields <reference/fields/many2many>`

.. exercise::

   #. add a `real.estate.tag` model (cozy, renovated...) with `name` and `color` with menu item, action, and list-only views
   #. add M2M field to the `real.estate.property` and `real.estate.tag` models
   #. add the field to the form view of properties with many2many_tags widget

.. spoiler:: Solution

   .. code-block:: python
      :caption: `__manifest__.py`
      :emphasize-lines: 1

      todo

----

Congratulations! You've learned the art of forging connections between your Odoo models. You're now
well-equipped to build complex, interconnected data structures. In the next chapter, we'll
:doc:`add custom business logic to the models <05_business_logic>`, turning your application from a
simple data management tool into a smart, automated system that can handle complex business
processes.
