.. _first_steps:

First Steps
===========

.. meta::
    :description: First steps with Renku
    :keywords: hello world, first steps, starter, primer

Interaction with the platform takes place via the Python-based command-line
interface (CLI) and the Python API. We recommend using the officially-
recommended python dependency manager `pipenv
<https://docs.pipenv.org/basics/>`_ for python dependency installations. You
can install it with ``pip``:

.. code-block:: console

   $ pip install pipenv

We will use ``pipenv`` below to install the ``renku`` CLI.


Our first Renku project
---------------------------

First, create a project directory:

.. code-block:: console

    $ mkdir -p ~/renku-projects/myproject
    $ cd ~/renku-projects/myproject

Install ``renku`` and start the shell:

.. code-block:: console

    $ pipenv install --skip-lock renku
    $ pipenv lock --pre
    $ pipenv shell

Initialize the project as a Renku project:

.. code-block:: console

    $ renku init

This command created a git repository for your project and an additional
`.renku` sub-directory:

.. code-block:: console

    $ ls -la .renku
    total 8
    drwxr-xr-x  3 rok  staff  102 May 22 17:23 .
    drwxr-xr-x  8 rok  staff  272 May 22 17:23 ..
    -rw-r--r--  1 rok  staff  308 May 22 17:23 metadata.yml

Create a dataset and import data
--------------------------------

Creating datasets is useful to group pieces of data together for e.g. sharing
or publication.

.. code-block:: console

    $ renku dataset create mydataset
    Creating a dataset ... OK
    $ ls -l data
    total 0
    drwxr-xr-x  3 rok  staff  102 May 22 17:30 mydata

At this point, our dataset just consists of metadata in JSON-LD format:

.. code-block:: console

    $ cat data/mydata/metadata.yml
    '@context':
      added: http://schema.org/dateCreated
      affiliation: scoro:affiliate
      authors:
        '@container': '@list'
      created: http://schema.org/dateCreated
      dcterms: http://purl.org/dc/terms/
      dctypes: http://purl.org/dc/dcmitypes/
      email: dcterms:email
      files:
        '@container': '@index'
      foaf: http://xmlns.com/foaf/0.1/
      identifier:
        '@id': dctypes:Dataset
        '@type': '@id'
      name: dcterms:name
      prov: http://www.w3.org/ns/prov#
      scoro: http://purl.org/spar/scoro/
      url: http://schema.org/url
    '@type': dctypes:Dataset
    authors:
    - '@type': dcterms:creator
      affiliation: null
      email: roskarr@ethz.ch
      name: Rok Roskar
    created: 2018-05-22 15:30:06.071631
    files: {}
    identifier: 6a354882-8308-42c0-9516-0b3c55b81f53
    name: mydata

We can import data from a variety of sources: local directories, remote URLs,
local or remote git repositories or other renku project. Here, we will import the
`README` file of this repo from the web:

.. code-block:: console

    $ renku dataset add mydataset https://raw.githubusercontent.com/
    SwissDataScienceCenter/renku/master/README.rst

Until now, we have created a Renku project and populated it with a dataset and
some data. Next, we will see how to use Renku to create a repeatable workflow.


Running a reproducible analysis
-------------------------------

For the purpose of the tutorial, we will count the number of lines the words
"science" and "renku" appear on in our `README` document by using standard
UNIX commands `grep` and `wc`.

First, get all occurrences of "science" and "renku":

.. code-block:: console

    $ renku run grep -i science data/mydataset/README.rst > readme_science
    $ renku run grep -i renku data/mydataset/README.rst > readme_renku

Now, combine these intermediate outputs into our final calculation:

.. code-block:: console

    $ renku run wc readme_science readme_renku > wc.out

For each of our invocations of `renku run`, Renku recorded the command we
executed into a `Common Workflow Language <http://www.commonwl.org/>`_ (CWL)
step. Renku uses this information to keep track of the lineage of data. For
example, we can see the full lineage of `wc.out` using the `renku log`
command:

.. code-block:: console

    $ renku log wc.out
    *  c53dbfa0 wc.out
    *    c53dbfa0 .renku/workflow/80a3f98ede2346f6bc686200016b17d6_wc.cwl
    |\
    * |  18bb2c64 readme_science
    * |  18bb2c64 .renku/workflow/edb4c0b1b4b44d2fb2aff45a8960f905_grep.cwl
    | *  faa4f82a readme_renku
    | *  faa4f82a .renku/workflow/3b454003c5884ee8b5b8a943665447fe_grep.cwl
    |/
    @  c7b5f922 data/mydataset/README.rst


This sequence represents the basic building blocks of a reproducible
scientific analysis workflow enabled by Renku. Each component of the workflow
we produced is bundled with metadata that allows us to continue to track
its lineage and therefore to reuse it as a building block in other projects
and workflows.


Updating results based on new input data
----------------------------------------

Suppose our input data changes -- what are the consequences for the downstream
analysis? Renku gives you some simple tools to inspect the state of your
project and, if necessary, update results in response to new data or even
changed source code.

Lets modify one of the two files we are using here -- open a text editor and
simply remove the first few lines from ``data/mydataset/README.rst``. When you are done, commit your change with this command:

.. code-block:: console

    $ git commit -am 'modified README.rst'

To see what effect this has on the steps we have done so far, use the ``renku status`` command:

.. code-block:: console

    $ renku status
    On branch master
    Files generated from outdated inputs:
      (use "renku log <file>..." to see the full lineage)

          readme_renku: data/mydataset/README.rst#42a770ef
          readme_science: data/mydataset/README.rst#42a770ef
          wc.out: data/mydataset/README.rst#42a770ef, data/mydataset/README.rst#42a770ef

    Input files used in different versions:
      (use "renku log --revision <sha1> <file>" to see a lineage for the given revision)

          data/mydataset/README.rst: 998dd21c, 42a770ef

There is a lot of information here - first of all, we know that our outputs
are out of date. Renku tells us that ``readme_renku``, ``readme_science`` and
``wc.out`` are all outdated, and that the reason is that ``README.rst`` used
to create those outputs is different from the one currently in the repository.

Updating our result is simple -- since we recorded all of the steps along the way, Renku can generate a workflow to repeat the analysis on the new data. For this, we can use the ``update`` command:

.. code-block:: console

    $ renku update
    ...
    [job step_3] completed success
    [step step_3] completed success
    [workflow 13bbd0b4779246b5a8c2f2fd85fba962.cwl] completed success
    {
        "output_0": {
            "location": "file:///.../demo/readme_science",
            "basename": "readme_science",
            "class": "File",
            "checksum": "sha1$42513cc8dcaf92f33b9cec9f976ad7fe0554d0d5",
            "size": 147,
            "path": "/.../demo/readme_science"
        },
        "output_1": {
            "location": "file:///.../demo/readme_renku",
            "basename": "readme_renku",
            "class": "File",
            "checksum": "sha1$b73e7a9d94c6d24622ed6d48d0ea4e6dee5485b5",
            "size": 245,
            "path": "/.../demo/readme_renku"
        },
        "output_2": {
            "location": "file:///.../demo/wc.out",
            "basename": "wc.out",
            "class": "File",
            "checksum": "sha1$803e3e8c3c0f6d5985bc2192f4ac727bef6eaf5b",
            "size": 327,
            "path": "/.../demo/wc.out"
        }
    }
    Final process status is success
