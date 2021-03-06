DoctrinePHPCRBundle
===================

The `DoctrinePHPCRBundle <https://github.com/doctrine/DoctrinePHPCRBundle>`_
provides integration with the PHP content repository and optionally with
Doctrine PHPCR-ODM to provide the ODM document manager in symfony.

.. index:: DoctrinePHPCRBundle, PHPCR, ODM

.. Tip::

    This reference only explains the Symfony2 integration of PHPCR and PHPCR-ODM.
    To learn how to use PHPCR refer to `the PHPCR website <http://phpcr.github.com/>`_ and for
    Doctrine PHPCR-ODM to the `PHPCR-ODM documentation <http://docs.doctrine-project.org/projects/doctrine-phpcr-odm/en/latest/>`_.

This bundle is based on the AbstractDoctrineBundle and thus is similar to the
configuration of the Doctrine ORM and MongoDB bundles.

Setup
-----

See :doc:`../tutorials/installing-configuring-doctrine-phpcr-odm`


Configuration
-------------

.. Tip::

    If you want to only use plain PHPCR without the PHPCR-ODM, you can simply not
    configure the ``odm`` section to avoid loading the services at all. Note that most
    CMF bundles by default use PHPCR-ODM documents and thus need ODM enabled.


PHPCR Session Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The session needs a PHPCR implementation specified in the ``backend`` section
by the ``type`` field, along with configuration options to bootstrap the
implementation. Currently we support ``jackrabbit``, ``doctrinedbal`` and ``midgard2``.
Regardless of the backend, every PHPCR session needs a workspace, username and
password.

.. Tip::

    Every PHPCR implementation should provide the workspace called *default*, but you
    can choose a different one. There is the ``doctrine:phpcr:workspace:create``
    command to initialize a new workspace. See also :ref:`reference-phpcr-commands`.

The username and password you specify here are what is used on the PHPCR layer in the
``PHPCR\SimpleCredentials``. They will usually be different from the username
and password used by Midgard2 or Doctrine DBAL to connect to the
underlying RDBMS where the data is actually stored.

If you are using one of the Jackalope backends, you can also specify ``options``.
They will be set on the Jackalope session. Currently this can be used to tune
pre-fetching nodes by setting ``jackalope.fetch_depth`` to something bigger than
0.

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        doctrine_phpcr:
            session:
                backend:
                    # see below for how to configure the backend of your choice
                workspace: default
                username: admin
                password: admin
                ## tweak options for jackrabbit and doctrinedbal (all jackalope versions)
                # options:
                #    'jackalope.fetch_depth': 1



PHPCR Session with Jackalope Jackrabbit
"""""""""""""""""""""""""""""""""""""""

The only setup required is to install Apache Jackrabbit (see :ref:`installing Jackrabbit <tutorials-installing-phpcr-jackrabbit>`).

The configuration needs the ``url`` parameter to point to your jackrabbit. Additionally you can
tune some other jackrabbit-specific options, for example to use it in a load-balanced setup or to fail
early for the price of some round trips to the backend.

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        doctrine_phpcr:
            session:
                backend:
                    type: jackrabbit
                    url: http://localhost:8080/server/
                    ## jackrabbit only, optional. see https://github.com/jackalope/jackalope/blob/master/src/Jackalope/RepositoryFactoryJackrabbit.php
                    # default_header: ...
                    # expect: 'Expect: 100-continue'
                    # enable if you want to have an exception right away if PHPCR login fails
                    # check_login_on_server: false
                    # enable if you experience segmentation faults while working with binary data in documents
                    # disable_stream_wrapper: true
                    # enable if you do not want to use transactions and you neither want the odm to automatically use transactions
                    # its highly recommended NOT to disable transactions
                    # disable_transactions: true

.. _reference-phpcr-doctrinedbal:

PHPCR Session with Jackalope Doctrine DBAL
""""""""""""""""""""""""""""""""""""""""""

This type uses Jackalope with a Doctrine database abstraction layer transport
to provide PHPCR without any installation requirements beyond any of the RDBMS
supported by Doctrine.

You need to configure a Doctrine connection according to the DBAL section in
the `Symfony2 Doctrine documentation <http://symfony.com/doc/current/book/doctrine.html>`_.

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        doctrine_phpcr:
            session:
                backend:
                    type: doctrinedbal
                    connection: doctrine.dbal.default_connection
                    # enable if you want to have an exception right away if PHPCR login fails
                    # check_login_on_server: false
                    # enable if you experience segmentation faults while working with binary data in documents
                    # disable_stream_wrapper: true
                    # enable if you do not want to use transactions and you neither want the odm to automatically use transactions
                    # its highly recommended NOT to disable transactions
                    # disable_transactions: true


Once the connection is configured, you can create the database and you *need*
to initialize the database with the ``doctrine:phpcr:init:dbal`` command.

.. code-block:: bash

    app/console doctrine:database:create
    app/console doctrine:phpcr:init:dbal

.. Tip::

    Of course, you can also use a different connection instead of the default.
    It is recommended to use a separate connection to a separate database if
    you also use Doctrine ORM or direct DBAL access to data, rather than mixing
    this data with the tables generated by jackalope-doctrine-dbal.
    If you have a separate connection, you need to pass the alternate
    connection name to the ``doctrine:database:create`` command with the
    ``--connection`` option. For doctrine PHPCR commands, this parameter is not
    needed as you configured the connection to use.


PHPCR Session with Midgard2
"""""""""""""""""""""""""""

Midgard2 is an application that provides a compiled PHP extension. It
implements the PHPCR API on top of a standard RDBMS.

To use the Midgard2 PHPCR provider, you must have both the [midgard2 PHP extension](http://midgard-project.org/midgard2/#download)
and [the midgard/phpcr package](http://packagist.org/packages/midgard/phpcr) installed.
The settings here correspond to Midgard2 repository parameters as explained in [the getting started document](http://midgard-project.org/phpcr/#getting_started).

The session backend configuration looks as follows:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        doctrine_phpcr:
            session:
                backend:
                    type: midgard2
                    db_type: MySQL
                    db_name: midgard2_test
                    db_host: "0.0.0.0"
                    db_port: 3306
                    db_username: ""
                    db_password: ""
                    db_init: true
                    blobdir: /tmp/cmf-blobs

For more information, please refer to the `official Midgard PHPCR documentation <http://midgard-project.org/phpcr/>`_.

.. _reference-phpcr-odm-configuration:

Doctrine PHPCR-ODM Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This configuration section manages the Doctrine PHPCR-ODM system. If you do not
configure anything here, the ODM services will not be loaded.

If you enable ``auto_mapping``, you can place your mappings in
``<Bundle>/Resources/config/doctrine/<Document>.phpcr.xml`` resp. ``...yml`` to
configure mappings for documents you provide in the ``<Bundle>/Document``
folder. Otherwise you need to manually configure the mappings section.

If ``auto_generate_proxy_classes`` is false, you need to run the ``cache:warmup``
command in order to have the proxy classes generated after you modified a
document. You can also tune how and where to generate the proxy classes with the
``proxy_dir`` and ``proxy_namespace`` settings. The the defaults are usually fine
here.

You can also enable `metadata caching <http://symfony.com/doc/master/reference/configuration/doctrine.html>`_.

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        doctrine_phpcr:
            odm:
                configuration_id:     ~
                auto_mapping: true
                mappings:
                    <name>:
                        mapping:              true
                        type:                 ~
                        dir:                  ~
                        alias:                ~
                        prefix:               ~
                        is_bundle:            ~
                auto_generate_proxy_classes: %kernel.debug%
                proxy_dir:            %kernel.cache_dir%/doctrine/PHPCRProxies
                proxy_namespace:      PHPCRProxies

                metadata_cache_driver:
                    type:                 array
                    host:                 ~
                    port:                 ~
                    instance_class:       ~
                    class:                ~
                    id:                   ~



Translation configuration
"""""""""""""""""""""""""

.. index:: I18N, Multilanguage

If you are using multilingual documents, you need to configure the available
languages. For more information on multilingual documents, see the
`PHPCR-ODM documentation on Multilanguage <http://docs.doctrine-project.org/projects/doctrine-phpcr-odm/en/latest/reference/multilang.html>`_.

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        doctrine_phpcr:
            odm:
                ...
                locales:
                    en: [de, fr]
                    de: [en, fr]
                    fr: [en, de]

This block defines the order of alternative locales to look up if a document is
not translated to the requested locale.


General Settings
~~~~~~~~~~~~~~~~

If the `jackrabbit_jar` path is set, you can use the `doctrine:phpcr:jackrabbit`
console command to start and stop jackrabbit.

You can tune the output of the `doctrine:phpcr:dump` command with
`dump_max_line_length`.

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        doctrine_phpcr:
            jackrabbit_jar:       /path/to/jackrabbit.jar
            dump_max_line_length:  120

.. _multiple-phpcr-sessions:

Configuring Multiple Sessions
-----------------------------

If you need more than one PHPCR backend, you can define ``sessions`` as child
of the ``session`` information. Each session has a name and the configuration
as you can use directly in ``session``. You can also overwrite which session
to use as ``default_session``.


.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        doctrine_phpcr:
            session:
                default_session:      ~
                sessions:
                    <name>:
                        workspace:            ~ # Required
                        username:             ~
                        password:             ~
                        backend:
                            # as above
                        options:
                            # as above

If you are using the ODM, you will also want to configure multiple document managers.

Inside the odm section, you can add named entries in the ``document_managers``.
To use the non-default session, specify the session attribute.

.. configuration-block::

    .. code-block:: yaml

        odm:
            default_document_manager:  ~
            document_managers:
                <name>:
                    # same keys as directly in odm, see above.
                    session: <sessionname>


A full example looks as follows:

.. configuration-block::

    .. code-block:: yaml

        doctrine_phpcr:
            # configure the PHPCR sessions
            session:
                sessions:

                    default:
                        backend: %phpcr_backend%
                        workspace: %phpcr_workspace%
                        username: %phpcr_user%
                        password: %phpcr_pass%

                    website:
                        backend:
                            type: jackrabbit
                            url: %magnolia_url%
                        workspace: website
                        username: %magnolia_user%
                        password: %magnolia_pass%

                    dms:
                        backend:
                            type: jackrabbit
                            url: %magnolia_url%
                        workspace: dms
                        username: %magnolia_user%
                        password: %magnolia_pass%
            # enable the ODM layer
            odm:
                document_managers:
                    default:
                        session: default
                        mappings:
                            SandboxMainBundle: ~
                            SymfonyCmfContentBundle: ~
                            SymfonyCmfMenuBundle: ~
                            SymfonyCmfRoutingExtraBundle: ~

                    website:
                        session: website
                        configuration_id: sandbox_magnolia.odm_configuration
                        mappings:
                            SandboxMagnoliaBundle: ~

                    dms:
                        session: dms
                        configuration_id: sandbox_magnolia.odm_configuration
                        mappings:
                            SandboxMagnoliaBundle: ~

                auto_generate_proxy_classes: %kernel.debug%

.. tip::

    This example also uses different configurations per repository (see the
    ``repository_id`` attribute). This case is explained in
    :doc:`../cookbook/phpcr-odm-custom-documentclass-mapper`.

.. _reference-phpcr-commands:


Services
--------

You can access the PHPCR services like this:

.. code-block:: php

    <?php

    namespace Acme\DemoBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class DefaultController extends Controller
    {
        public function indexAction()
        {
            // PHPCR session instance
            $session = $this->container->get('doctrine_phpcr.default_session');
            // PHPCR ODM document manager instance
            $documentManager = $this->container->get('doctrine_phpcr.odm.default_document_manager');
        }
    }


Events
------

You can tag services to listen to Doctrine PHPCR events. It works the same way
as for Doctrine ORM. The only differences are

* use the tag name ``doctrine_phpcr.event_listener`` resp. ``doctrine_phpcr.event_subscriber`` instead of ``doctrine.event_listener``.
* expect the argument to be of class Doctrine\ODM\PHPCR\Event\LifecycleEventArgs rather than in the ORM namespace.

You can register for the events as described in `the PHPCR-ODM documentation <http://docs.doctrine-project.org/projects/doctrine-phpcr-odm/en/latest/reference/events.html>`_.

    services:
        my.listener:
            class: Acme\SearchBundle\Listener\SearchIndexer
                tags:
                    - { name: doctrine_phpcr.event_listener, event: postPersist }

More information on the doctrine event system integration is in this `Symfony cookbook entry <http://symfony.com/doc/current/cookbook/doctrine/event_listeners_subscribers.html>`_.



Doctrine PHPCR Commands
-----------------------

All commands about PHPCR are prefixed with ``doctrine:phpcr`` and you can use
the --session argument to use a non-default session if you configured several
PHPCR sessions.

Some of these commands are specific to a backend or to the ODM. Those commands
will only be available if such a backend is configured.

Use ``app/console help <command>`` to see all options each of the commands has.

- ``doctrine:phpcr:workspace:create``  Create a workspace in the configured repository
- ``doctrine:phpcr:workspace:list``  List all available workspaces in the configured repository
- ``doctrine:phpcr:purge``  Remove content from the repository
- ``doctrine:phpcr:register-system-node-types``  Register system node types in the PHPCR repository
- ``doctrine:phpcr:register-node-types``  Register node types in the PHPCR repository
- ``doctrine:phpcr:fixtures:load``  Load data fixtures to your PHPCR database.
- ``doctrine:phpcr:import``  Import xml data into the repository, either in JCR system view format or arbitrary xml
- ``doctrine:phpcr:export``  Export nodes from the repository, either to the JCR system view format or the document view format
- ``doctrine:phpcr:dump``  Dump the content repository
- ``doctrine:phpcr:query``  Execute a JCR SQL2 statement
- ``doctrine:phpcr:mapping:info``  Shows basic information about all mapped documents


.. note::

    To use the ``doctrine:phpcr:fixtures:load`` command, you additionally need to install the
    `DoctrineFixturesBundle <http://symfony.com/doc/current/bundles/DoctrineFixturesBundle/index.html>`_
    and its dependencies. See that documentation page for how to use fixtures.


Jackrabbit specific commands
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you are using jackalope-jackrabbit, you also have a command to start and stop the
jackrabbit server:

-  ``jackalope:run:jackrabbit``  Start and stop the Jackrabbit server


Doctrine DBAL specific commands
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you are using jackalope-doctrine-dbal, you have a command to initialize the
database:

- ``jackalope:init:dbal``   Prepare the database for Jackalope Doctrine DBAL

Note that you can also use the doctrine dbal command to create the database.


Some example command runs
~~~~~~~~~~~~~~~~~~~~~~~~~

Running `SQL2 queries <http://www.h2database.com/jcr/grammar.html>`_ against the repository

.. code-block:: bash

    app/console doctrine:phpcr:query "SELECT title FROM [nt:unstructured] WHERE NAME() = 'home'"


Dumping nodes under /cms/simple including their properties

.. code-block:: bash

    app/console doctrine:phpcr:dump /cms/simple --props=yes


