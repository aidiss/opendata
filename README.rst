.. default-role:: literal

Lithuanian Open Data Manifest
#############################

Here you can request data for your project or tell what data your project
already use and how things are going for the project.

This is a non-official community effort. There is no guarantee that requested
data will ever be opened. But knowing what data are needed is important.


What's it all about?
====================

Currently Lithuanian government is working on a open data project. The idea of
this repository is to help gevernment decide what data to open first and how to
transform data so that final result would be as close as posible to what is
needed for data users.

Without knowing who is going to use the data, how it will be used and what data
is needed, there are number of issues:

- Priorities of what to open first are not clear.

- It is not clear what transformations are needed to best meet the demand.

- What economical or social implact opened data will do.

In other words, without clearly knowing what data users need, it is eassy to
open datasets, that will never be used in practice. And that would be waste of
time and effort.

So please tell us, what data you need and hopefully government will do their
best to give you exactly what you need. Wouldn't it be great?


How to submit request for data?
===============================

Requests for data are submitted as `GitHub pull requests`_. One pull request
should describe data needed for a project in YAML_ format.

YAML_ files are organized into this structure::

  vocabulary/
    <object>.yml
  providers/
    <provider>.yml
  sources/
    <provider>/
      <source>.yml
  projects/
    <project>.yml
  media/
    providers/
      <provider>/
        logo.png
    projects/
      <project>/
        logo.png


Here is an example, how a project could request the data:

.. code-block:: yaml

  # projects/manopozicija.lt.yml
  id: "projects/manopozicija.lt"
  title: "ManoPozicija.lt"
  type: "project"
  since: "2015-07-21"
  impact:
    - {year: 2015, users: 10, revenue: 0, employees: 0}
    - {year: 2016, users:  0, revenue: 0, employees: 0}
    - {year: 2017, users:  0, revenue: 0, employees: 0}
    - {year: 2018, users: 10, revenue: 0, employees: 0}
    - {year: 2019, users: 20, revenue: 0, employees: 0}
  objects:
    seimo_narys:
      properties:
        first_name:
          type: "string"
          source: "gov/lrs/ad"
        last_name:
          type: "string"
          source: "gov/lrs/ad"

`impact` parameter is used to describe social and economical impact. Both
future and past dates can be provided for estimated and retrospective impact.
This parameter helps to prioritize what data needs to be opened first. Projects
with a higher impact should be supplied with the data first.

`source` parameter is optional, but if you know who owns the data, you can also
specify the source.

As you can see data structure here looks a bit similar to json-schema_, meaning
that some fields are well defined in json-schema_ specifications.


What is the purpose of vocabulary?
==================================

It is very likely that many projects will use same data fields. In order to
know how many projects use the same data fields, all data fields are defined in
one places called vocabulary.

Here is example how vocabulary file looks:

.. code-block:: yaml

  # vocabulary/seimo_narys.yml
  id: "seimo_narys"
  title: "Member of Parliament"
  description: ""
  type: "vocabulary"
  properties:
    first_name:
      title: "First name"
      type: "string"
    last_name:
      title: "Last name"
      type: "string"

All object and property names must be defined in vocabulary file, befere using
those names in data or source files.


How to describe a data source?
==============================

If you know who has the data you can also describe the data source. Here is
example how this could be done:

.. code-block:: yaml

  # sources/gov/lrs/ad.yml
  id: "gov/lrs/ad"
  title: "Members of Parliament (XML)"
  description: "XML file containing data about members of parliament."
  type: "source"
  source: "http://apps.lrs.lt/sip/p2b.ad_seimo_nariai"
  provider: "gov/lrs"
  since: "2016-01-01"
  objects:
    seimo_narys:
      source: "/SeimoInformacija/SeimoKadencija/SeimoNarys"
      properties:
        first_name:
          type: "string"
          source: "@vardas"
        last_name:
          type: "string"
          source: "@pavardė"

Defining a source is the most complicated part, but luckily this part is
optional!

Here `source` parameter is optional. It is used just to demonstrate complete
example of how things look.

The idea with sources, is that you can specify exact location of the data. Just
by using descriptions provided in `source` fields, data can be extracted in a
fully automated way. Well at least in simple cases. In addition this detailed
source description can be used to validate if source data are really there.

But in most cases we will not have direct access to data, so that's why
`source` parameter is optional. It is enough to just specify a URL and list
properties that we think are provided by the source.

`gov/lrs` parameter points to another YAML file where provider is defined. Here
is how this file looks:

.. code-block:: yaml

  # providers/gov/lrs.yml
  id: "gov/lrs"
  title: "Lietuvos Respublikos Seimas"
  type: "provider"
  logo: "logo.png"

`logo` property here points to `media/providers/gov/lrs/logo.png` file.


I don't know how to create a pull request
=========================================

If you don't know how to use git and don't know YAML_, then you can simply
`create a task`_ and if your project idea will be worth addeng, then someone
alse will take care of describing you data needs in machine readable format as
explained above.


Automated checks
================

Once pull request is created, automated scripts will check if everything is OK,
then a human will review pull request and if everything is OK, then pull
request will be accepted.

If you want to check yaml files locally, you can run this command::

  make check


Data sources
============

Here I will try to explain, how `source` parameter works.

`source` parameter can be defined in three different places:

.. code-block:: yaml

  source: # dataset scope
  objects:
    object:
      source: # object scope
      properties:
        field:
          source: # field scope

`source` parameter can have short and log forms, short form looks like this:

.. code-block:: yaml

  source: "https://example.com"
  objects:
    object:
      source: "data.csv"
      properties:
        field:
          source: "column"

And exactly same thing can be written as long form:

.. code-block:: yaml

  source:
    dsn: "https://example.com"
    type: "csv"
    delim: ","
  objects:
    object:
      source:
        dsn: "data.csv"
      properties:
        field:
          source:
            dsn: "column"

As you can see in dataset scope, you define dataset specific properties all
those properties will be inherited in narrower scopes.

Depending on type, short form `source` value has different meaning, for `csv`
type, in dataset scope it means base URL, in object scope - relative or full
URL, in field scope it means column name.


XML source
----------

.. code-block:: yaml

  source: "https://example.com/data.xml"
  objects:
    object:
      source: "//object"
      properties:
        field:
          source: "@attribute"


JSON source
-----------

.. code-block:: yaml

  ---
  id: "com/example/items"
  source: "https://example.com/items.json"
  objects:
    object:
      source: "items"
      properties:
        id:
          source: "id"
  ---
  id: "com/example/item"
  source: "https://example.com/items/{com/example/items/object/id}.json"
  objects:
    object:
      source: []
      properties:
        id:
          source: "id"
        field1:
          source: ["some", "nested", "field"]


PostgreSQL source
-----------------

.. code-block:: yaml

  source: "postgresql://localhost/dbname"
  objects:
    object:
      source: "tablename"
      properties:
        field:
          source: "fieldname"

Another example with a query:

.. code-block:: yaml

  source: "postgresql://localhost/dbname"
  objects:
    object:
      source:
        query: >
          SELECT *
          FROM table1
          JOIN table2 ON (table1.id = table2.id)
          WHERE table1.param > 42
          ORDER BY table2.param
      properties:
        field:
          source: "fieldname"


HTML table source
-----------------

.. code-block:: yaml

  source: "https://example.com/some/page.html"
  objects:
    object:
      source:
        type: htmltable
        cols: 4
      properties:
        field:
          source: "Some column name"


OpenDocument Spreadsheet
------------------------

.. code-block:: yaml

  source: "https://example.com/data.ods"
  objects:
    object:
      source: "SheetName"
      properties:
        field:
          source: "A"


.. _GitHub pull requests: https://help.github.com/articles/creating-a-pull-request/
.. _YAML: https://en.wikipedia.org/wiki/YAML
.. _json-schema: https://en.wikipedia.org/wiki/JSON#JSON_Schema
.. _create a task: https://github.com/sirex/opendata/issues/new
