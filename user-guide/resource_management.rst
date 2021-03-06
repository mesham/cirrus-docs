File and Resource Management
============================

This section covers some of the tools and technical knowledge that will
be key to maximising the usage of the Cirrus system, such as the online
administration tool SAFE and calculating the CPU-time available.

The default file permissions are then outlined, along with a description
of changing these permissions to the desired setting. This leads on to
the sharing of data between users and systems often a vital tool for
project groups and collaboration.

Finally we cover some guidelines for I/O and data archiving on Cirrus.

The Cirrus Administration Web Site (SAFE)
-----------------------------------------

All users have a login and password on the Cirrus Administration Web
Site (also know as the 'SAFE'):
`SAFE <https://safe.epcc.ed.ac.uk/safadmin/>`__. Once logged into this
web site, users can find out much about their usage of the Cirrus
system, including:

-  Account details - password reset, change contact details
-  Project details - project code, start and end dates
-  kAU balance - how much time is left in each project you are a member
   of
-  Filesystem details - current usage and quotas
-  Reports - generate reports on your usage over a specified period,
   including individual job records
-  Helpdesk - raise queries and track progress of open queries

Checking your CPU-time (kAU) balance
------------------------------------

You can view these details by logging into the SAFE
(https://safe.epcc.ed.ac.uk/safadmin/).

Use the *Login accounts* menu to select the user account that you wish
to query. The page for the login account will summarise the resources
available to account.

You can also generate reports on your usage over a particular period and
examine the details of how many kAU individual jobs on the system cost.
To do this use the *Service information* menu and selet *Report generator*.

For more information on using the SAFE see the :doc:`../safe-guide/introduction`.

Disk quotas
-----------

Disk quotas on Cirrus are managed via
`SAFE <https://safe.epcc.ed.ac.uk/safadmin/>`__

File permissions and security
-----------------------------

By default, each user is a member of the group with the same name as
[group\_code] in the ``/lustre/home`` and ``/lustre/work`` directory paths, e.g.
``x01-a``. This allows the user to share files with only members of that
group by setting the appropriate group file access permissions. As on
other UNIX or Linux systems, a user may also be a member of other
groups. The list of groups that a user is part of can be determined by
running the ``groups`` command.

It is, of course, possible to make files accessible to all users on
Cirrus, by setting the "other" file access permissions appropriately.
You should not allow "other" write access to any of your directories,
and for security reasons it is recommended that you do not allow anyone
else read or write access to important account information (e.g.
``$HOME/.ssh``).

Default Unix file permissions can be specified by the ``umask`` command.
The default umask value on Cirrus is 22, which provides "group" and
"other" read permissions for all files created, and "group" and "other"
read and execute permissions for all directories created. This is highly
undesirable, as it allows everyone else on the system to access (but at
least not modify or delete) every file you create. Thus it is strongly
recommended that users change this default umask behaviour, by adding
the command ``umask 077`` to their ``$HOME/.profile`` file. This umask
setting only allows the user access to any file or directory created.
The user can then selectively enable "group" and/or "other" access to
particular files or directories if required.

ASCII (or formatted) files
~~~~~~~~~~~~~~~~~~~~~~~~~~

These are the most portable, but can be extremely inefficient to read
and write. There is also the problem that if the formatting is not done
correctly, the data may not be output to full precision (or to the
subsequently required precision), resulting in inaccurate results when
the data is used. Another common problem with formatted files is FORMAT
statements that fail to provide an adequate range to accommodate future
requirements, e.g. if we wish to output the total number of processors,
NPROC, used by the application, the statement:

::

    WRITE (*,'I3') NPROC

will not work correctly if NPROC is greater than 999.

Binary (or unformatted) files
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

These are much faster to read and write, especially if an entire array
is read or written with a single READ or WRITE statement. However the
files produced may not be readable on other systems.

GNU compiler ``-fconvert=swap`` compiler option.
    This compiler option often needs to be used together with a second
    option ``-frecord-marker``, which specifies the length of record
    marker (extra bytes inserted before or after the actual data in the
    binary file) for unformatted files generated on a particular system.
    To read a binary file generated by a big-endian system on Cirrus,
    use
    ``-fconvert=swap -frecord-marker=4``.
    Please note that due to the same 'length of record marker' reason,
    the unformatted files generated by GNU and other compilers on Cirrus
    are not compatible. In fact, the same WRITE statements would result
    in slightly larger files with GNU compiler. Therefore it is
    recommended to use the same compiler for your simulations and
    related pre- and post-processing jobs.

Other options for file formats include:

Direct access files
    Fortran unformatted files with specified record lengths. These may
    be more portable between different systems than ordinary (i.e.
    sequential IO) unformatted files, with significantly better
    performance than formatted (or ASCII) files. The "endian" issue
    will, however, still be a potential problem.
Portable data formats
    These machine-independent formats for representing scientific data
    are specifically designed to enable the same data files to be used
    on a wide variety of different hardware and operating systems. The
    most common formats are:

    -  netCDF: http://www.unidata.ucar.edu/software/netcdf/
    -  HDF: http://www.hdfgroup.org/HDF5/

    It is important to note that these portable data formats are
    evolving standards, so make sure you are aware of which version of
    the standard/software you are using, and keep up-to-date with any
    backward-compatibility implications of each new release.

File IO Performance Guidelines
------------------------------

Here are some general guidelines

-  Whichever data formats you choose, it is vital that you test that you
   can access your data correctly on all the different systems where it
   is required. This testing should be done as early as possible in the
   software development or porting process (i.e. before you generate
   lots of data from expensive production runs), and should be repeated
   with every major software upgrade.
-  Document the file formats and metadata of your important data files
   very carefully. The best documentation will include a copy of the
   relevant I/O subroutines from your code. Of course, this
   documentation must be kept up-to-date with any code modifications.
-  Use binary (or unformatted) format for files that will only be used
   on the Intel system, e.g. for checkpointing files. This will give the
   best performance. Binary files may also be suitable for larger output
   data files, if they can be read correctly on other systems.
-  Most codes will produce some human-readable (i.e. ASCII) files to
   provide some information on the progress and correctness of the
   calculation. Plan ahead when choosing format statements to allow for
   future code usage, e.g. larger problem sizes and processor counts.
-  If the data you generate is widely shared within a large community,
   or if it must be archived for future reference, invest the time and
   effort to standardise on a suitable portable data format, such as
   netCDF or HDF.

Backup policies
---------------

There are currently no backups of data on Cirrus as backing up the whole 
Lustre file system would adversly affect the performance of write
access for simulations. The nature of the Lustre parallel file system
means that there is data resiliance in the case of failures of individual
hardware components. However, we strongly advise that you keep copies of
any critical data on different  systems.

We are currently investigating options for providing backups of critical data.
