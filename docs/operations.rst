.. _service-zfs-operations:

ZFS Operations
##############
Common ZFS operations.

Mounting Existing ZFS Pool
**************************
.. code-block:: bash

  zpool import {POOL}
  zpool status {POOL}
  zpool scrub {POOL}
  zfs mount -l -a

.. note::
  * A pool name does not need to be used to identify a ZFS container, it
    will be detected automatically. ``zpool import -a``.
  * Search for the pool, then mount it, use ``-f`` if it wasn't exported.
    Scrub.

.. hint::
  Set mountpoint to immutable without the ZFS dataset mounted. This prevents
  writes when the dataset is not ready: ``chattr +i {MOUNTPOINT}``

.. _service-zfs-filesystem-options:

ZFS Filesystem Options
**********************
ZFS can be tweaked per dataset based on the data being used. ZFS only applies
new settings on newly written data; changing options for pre-existing data
requires export/re-import of that data to the dataset.

.. code-block:: bash
  :caption: Enable text compression and disable atime for Maildir datasets.

  zfs set atime=off {POOL}/{DATASET}
  zfs set compression=lz4 {POOL}/{DATASET}

.. code-block:: bash
  :caption: Enable high compression for backup datasets.

  zfs set compression=gzip {POOL}/{DATASET}

Add Caching (L2ARC) Disk
************************
Use a fast disk to cache reads. Disk may be added and removed at at point.
Prefer expanding RAM to adding cache disk.

.. code-block:: bash
  :caption: Enable read caching disk

  ls -l /dev/disk/by-id
  zpool add {POOL} cache /dev/disk/by-id/{ID}

Setup Monthly ZFS Scrub
***********************
Scrubbing verifies all blocks can be read, and marks then bad if not. This is
done while the filesystem is online, but may slightly impact performance.

Monthly and Weekly scrubs have been incorporated into systemd:

.. code-block:: bash
  :caption: Enable pre-defined ZFS scrub jobs

  systemctl enable zfs-scrub-monthly@{POOL}.timer --now
  systemctl enable zfs-scrub-weekly@{POOL}.timer --now

Consider cronjob setup deprecated.

.. code-block:: bash
  :caption: **0750 root root** ``/root/bin/scrub-zpool-monthly``

  #!/bin/bash
  #
  # Scrubs zpool monthly.
  /usr/bin/zpool scrub {POOL}

.. code-block:: bash
  :caption: Add to `root crontab <https://en.wikipedia.org/wiki/Cron>`_ to run monthly.

  @weekly /root/bin/scrub-zpool-monthly

`Reference <https://docs.oracle.com/cd/E23823_01/html/819-5461/gbbwa.html>`__