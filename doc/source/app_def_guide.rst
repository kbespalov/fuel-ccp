.. app_def_guide:

=========================================
Application definition contribution guide
=========================================

This guide covers CCP specific DSL, which is used by the CCP CLI to populate
k8s objects.

Application definition files location
=====================================

All application definition files should be located in the ``service/``
directory, as a ``component_name.yaml`` file, for example:

::

    service/keystone.yaml

All templates, such as configs, scripts, etc, which will be used for this
service, should be located in ``service/<component_name>/files``, for example:

::

    service/files/keystone.conf.j2

All files inside this directory are Jinja2 templates. Default variables for
these templates should be located in
``service/component_name/files/defaults.yaml`` inside the ``configs`` key.

Understanding globals and defaults config
=========================================

There are three config locations, which the CCP CLI uses:

#. ``Global defaults`` - fuel_ccp/resources/defaults.yaml in ``fuel-ccp`` repo.
#. ``Component defaults`` - service/files/defaults.yaml in each component repo.
#. ``Global config`` - Optional. Set path to this config via
   "--deploy-config /path" CCP CLI arg.

Before deployment, CCP will merge all these files into one dict, using the
order above, so "component defaults" will override "global defaults" and
"global config" will override everything.

Global defaults
---------------

This is project wide defaults, CCP keeps it inside fuel-ccp repository in
``fuel_ccp/resources/defaults.yaml`` file. This file defines global variables,
that is variables that are not specific to any component, like eth interface
names.

Component defaults
------------------

Each component repository could contain a ``service/files/defaults.yaml`` file
with default config for this component only.

Global config
-------------

Optional config with global overrides for all services. Use it only if you need
to override some defaults.

Config keys types
=================

Each config could contain 3 keys:

- ``configs``

- ``versions``

- ``sources``

- ``nodes``

Each key has its own purpose and isolation, so you have to add your variable
to the right key to make it work.

configs key
-----------

Isolation:

- Used in service templates files (service/files/).

- Used in application definition file service/component_name.yaml.

Allowed content:

- Any types of variables allowed.

Example:

::

    configs:
      keystone_debug: false

So you could add "{{ keystone_debug }}" variable to you templates, which will
be rendered into "false" in this case.

versions key
------------

Isolation:

- Used in Dockerfile.j2 only.

Allowed content:

- Only versions of different software should be kept here.

For example:

::

    versions:
     influxdb_version: "0.13.0"

So you could add this to influxdb Dockerfile.j2:

::

    curl https://dl.influxdata.com/influxdb/releases/influxdb_{{ influxdb_version }}_amd64.deb

sources key
-----------

Isolation:

- Used in Dockerfile.j2 only.

Allowed content:

- This key has a restricted format, examples below.

Remote git repository example:

::

    sources:
      openstack/keystone:
        git_url: https://github.com/openstack/keystone.git
        git_ref: master

Local git repository exaple:

::

    sources:
      openstack/keystone:
        source_dir: /tmp/keystone

So you could add this to Dockerfile.j2:

::

    {{ copy_sources("openstack/keystone", "/keystone") }}

CCP will use the chosen configuration, to copy git repository into Docker
container, so you could use it latter.

network_topology key
--------------------

Isolation:

- Used in service templates files (service/files/).

Allowed content:

- This key is auto-created by entrypoint script and populated with container
  network topology, based on the following variables: ``private_interface`` and
  ``public_interface``.

You could use it to get the private and public eth IP address. For example:

::

    bind = network_topology["private"]["address"]
    listen = network_topology["public"]["address"]

nodes and roles key
-------------------

Isolation:

- Not used in any template file, only used by the CCP CLI to create a cluster
  topology.

Allowed content:

- This key has a restricted format, example of this format can be found in
  ``fuel-ccp`` git repository in ``etc/topology-example.yaml`` file.

"CCP_*" env variables
---------------------

Isolation:

- Used in service templates files (service/files/).

Allowed content:

- This variables are created from the application definition ``env`` key.
  Only env keys which start with "CCP\_" will be passed to config hash.

This is mainly used to pass some k8s related information to container, for
example, you could use it to pass k8s node hostname to container via this
variable:

Create env key:

::

      env:
        - name: CCP_NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName

Use this variable in some config:

::

    {{ CCP_NODE_NAME }}

Application definition language
===============================

Please refer to :doc:`dsl` for detailed description of CCP DSL syntax.

