EDRN Directory Service
======================

The EDRN Directory Service is a set of interoperating components that provide
directory lookup and authentication for the Early Detection Research Network
(EDRN_).  It includes the following:

• An LDAP server running Apache DS_.
• A special Apache DS interceptor_ that handles authentication of EDRN
  Secure Site (SS_) usernames.
• A program to synchronize users from the DMCC_ RDF_ feed of people registered
  with EDRN.
• A program to synchronize groups of users based on the RDF feeds of people,
  EDRN member sites, and EDRN committees.
• A process supervisor to monitor Apache DS.
• Cron scripts to run the synchronization programs.
• Automatically generated configuration files to link it all together.

The EDRN Directory Service was developed by the EDRN Informatics Center (IC_),
at the Jet Propulsion Laboratory (JPL_), operated by the California Institute
of Technology (Caltech_).

This is copyrighted and licensed software.  Please see LICENSE.rst for more
information.


Requirements
------------

Installation assumes you're deploying to cancer.jpl.nasa.gov.  But more
concretely, we assume the following:

• Java JDK 1.6 or later
• OpenLDAP 2.3 or later
• Python 2.6 or 2.7


Deploying the Service
---------------------

The EDRN Directory Service must be "primed" with a dump of the current
directory data stored in its incarnation about to be replaced.  To do so, run
the following::

    ldapsearch -x -W -H ldaps://edrn.jpl.nasa.gov -D uid=admin,ou=system \
        -b dc=edrn,dc=jpl,dc=nasa,dc=gov -s sub '(objectClass=*)' > dump.ldif

When prompted, provide the password to ``uid=admin,ou=system``.  Save the
``dump.ldif`` file to a convenient location.  *Important:* if the entry for
the dn "dc=edrn,dc=jpl,dc=nasa,dc=gov" isn't at the top of the file, find it
and move it up top!

Next, after extracting the software archive or exporting it from revision
control, and changing the current working directory to the newly extract tree,
do the following:

1.  Edit the ops.cfg file.  The [passwords] section *must* be updated with
    better passwords for the system-dn (``uid=admin,ou=system``) and for the
    process supervisor.  The password for the EDRN keystore file,
    ``cancer.ks``, must be the correct password to decrypt and open the file.

    You can edit the other sections, if necessary, however the defaults should
    be just fine for cancer.jpl.nasa.gov.

2.  Bootstrap by running: ``python2.6 bootstrap.py -dc ops.cfg``

3.  Build out by running: ``bin/buildout -c ops.cfg``

4.  Shut down the incarnation of the service about to be replaced.

5.  Run the following commands (as root)::

        install -o root -g root -m 755 bin/edrndir /etc/init.d
        install -o root -g root -m 755 bin/cron.daily /etc/cron.daily/edrndir
        chkconfig --add edrndir

6.  Start the service by running: ``/etc/init.d/edrndir start``

7.  Wait a few moments for the server to start up, then run the following
    commands::

        bin/buildout -c ops.cfg install apacheds-admin
        bin/buildout -c ops.cfg install schema

8.  Prime the server with data dumped from the previous incarnation::

        ldapadd -x -W -D uid=admin,ou=system -H ldap://localhost -f dump.ldif
    
    Replace ``dump.ldif`` with the path to the dump file you made.  When
    prompted, enter the password for the system-dn (set in the ``ops.cfg``
    file).

9.  Protect the ops.cfg file: ``chmod 600 ops.cfg``.

That's it!  Have a refreshing beverage.


Development
-----------

Since the EDRN Directory Service is deployed as a Buildout_, you can use it to
develop and test the Python_ components that go into it.  To do so, run::

    python2.6 bootstrap.py -c dev.cfg
    bin/buildout -c dev.cfg

This will put your buildout into "development mode", which gives you Apache DS
configured with a simple password "``password``", listening for non-SSL LDAP
requests on port 10389.  You can then start the server by running::

    bin/supervisord

And then load the simple admin password and schema::

    bin/buildout -Noc dev.cfg install apacheds-admin
    bin/buildout -Noc dev.cfg install schema

Finally, you can use the ``bin/develop`` command to check out and work on the
Python components.  See http://pypi.python.org/pypi/mr.developer/ for more
details.



.. References:
.. _Buildout: http://www.buildout.org/
.. _Caltech: http://www.caltech.edu/
.. _DMCC: http://edrn.nci.nih.gov/about-edrn/scicomponents/dmcc
.. _DS: http://directory.apache.org/apacheds/1.5/
.. _EDRN: http://edrn.nci.nih.gov/
.. _IC: http://cancer.jpl.nasa.gov
.. _interceptor: http://directory.apache.org/apacheds/1.5/12-interceptors.html
.. _JPL: http://www.jpl.nasa.gov/
.. _Python: http://python.org/
.. _RDF: http://www.w3.org/RDF/
.. _SS: http://www.compass.fhcrc.org/enterEDRN/
