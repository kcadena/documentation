=============================
SSL Configuration
=============================
.. _`using-ssl`:

When you finish generating the SSL certificates for your cluster, you need to enable it for each node.  If you have not yet generated the SSL certificates, see :doc:`sslcert` for a guide on how to do so.

.. note:: For Gelera Cluster, SSL configurations are not dynamic.  Since they must be set on every node in the cluster, if you are enabling this feature with a running cluster you need to restart the entire cluster.


-----------------------
Enabling SSL
-----------------------

There are three vectors that you can secure with :abbr:`SSL (Secure Socket Layer)`: the database server and client, replication traffic from Galera Cluster, and state snapshot transfers through ``stunnel``.


^^^^^^^^^^^^^^^^^^^^^^
Securing the Database
^^^^^^^^^^^^^^^^^^^^^^
.. _`securing-database`:

For securing database server and client connections, you can use the internal MySQL :abbr:`SSL (Secure Socket Layer)` support.  In the event that you use logical transfer methods for state snapshot transfer, such as ``mysqldump``, this is the only step you need to take in securing your state snapshot transfers.

In the configuration file, (``my.cnf``), add the follow parameters to each unit:

.. code-block:: ini

   # MySQL Server
   [mysqld]
   ssl-ca = /path/to/ca-cert.pem
   ssl-key = /path/to/server-key.pem
   ssl-cert = /path/to/server-cert.pem

   # MySQL Client Configuration
   [mysql]
   ssl-ca = /path/to/ca-cert.pem
   ssl-key = /path/to/client-key.pem
   ssl-cert = /path/to/client-cert.pem

These parameters tell the database server and client which files to use in encrypting and decrypting their interactions through :abbr:`SSL (Secure Socket Layer)`.  The node will begin to use them once it restarts.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Securing Replication Traffic
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. _`securing-replication-traffic`:

In order to enable SSL on the internal node processes, you need to define the paths to the key, certificate and certificate authority files that you want the node to use in encrypting replication traffic.

- :ref:`socket.ssl_key <socket.ssl_key>` The key file.

- :ref:`socket.ssl_cert <socket.ssl_cert>` The certificate file.

- :ref:`socket.ssl_ca <socket.ssl_ca>` The certificate authority file.

You can configure these options through the :ref:`wsrep_provider_options <wsrep_provider_options>` parameter in the configuration file, (that is, ``my.cnf``).
  
.. code-block:: ini
		
   wsrep_provider_options="socket.ssl_ca=/path/to/server-key.pem,
                           socket.ssl_cert=/path/to/server-cert.pem,
                           socket.ssl_ca=/path/to/cacert.pem"
			   
This tells Galera Cluster which files to use in encrypting and decrypting replication traffic through SSL.  The node will begin to use them once it restarts.
			  
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Securing Physical State Snapshot Transfers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. _`securing-state-transfers`:

In the event that you use a physical transfer method for your state snapshot transfers, such as ``rsync``, you need to configure the node to use ``stunnel`` to secure these transfers.

Using your preferred text editor, edit ``/etc/stunnel/stunnel.conf`` to add the following lines:

.. code-block:: ini

   ;; stunnel Configuration
   CAfile = /path/to/ca-cert.pem
   cert = /path/to/client-cert.pem
   key = /path/to/client-key.pem

   ;; ssync Server Configuration
   [ssync]
   accept = 4444
   connect = 4444

   ;; rsync Client Configuration
   [rsync]
   accept = 4444
   connect = 4444





-------------------------
Configuring SSL
-------------------------
.. _`configuring-ssl`:

In the event that you want or need to further configure how the node uses :abbr:`SSL (Secure Sockets Layer)`, Galera Cluster provides some additional parameters, including defining the cyclic redundancy check and setting the cryptographic cipher algorithm you want to use.

.. seealso:: For a complete list of available configurations available for :abbr:`SSL (Secure Sockets Layer)`, see the options with the ``socket.`` prefix at :doc:`galeraparameters`.



^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Configuring the Socket Checksum
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. _`configuring-socket-checksum`:

Using the :ref:`socket.checksum<socket.checksum>` parameter, you can define whether or which cyclic redundancy check the node uses in detecting errors.  There are three available settings for this parameter, which are defined by an integer:

- ``0`` Disables the checksum.

- ``1`` Enables the CRC-32 checksum.

- ``2`` Enables the CRC-32C checksum.

The default configuration for this parameter is ``1`` or ``2`` depending upon your version.  CRC-32C is optimized for and potentially hardware accelerated on Intel CPU's.


.. code-block:: ini

   wsrep_provider_options = "socket.checksum=2"



^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Configuring the Encryption Cipher
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. _`configuring-cipher`:

Using the :ref:`socket.ssl_cipher<socket.ssl_cipher>` parameter, you define which cipher the node uses in encrypting replication traffic.  Galera Cluster uses whatever ciphers are available to the :abbr:`SSL (Secure Sockets Layer)` implementation installed on the nodes. For instance, if you install OpenSSL on your node, Galera Cluster can use any cryptographic algorithms OpenSSL uses in ciphers.

The :abbr:`SSL (Secure Sockets Layer)` configuration for Galera Cluster defaults to AES128-SHA, as this setting is considerably faster and no less secure than AES256.

.. code-block:: ini

   wsrep_provider_options = "socket.ssl_cipher=AES128-SHA"




