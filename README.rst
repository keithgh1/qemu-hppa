=============================
Keith's Fork and what it does
=============================

This fork carries a rewrite of QEMU's NCR 53C710 SCSI controller model
(``hw/scsi/ncr53c710.c``) and its LASI glue. The existing upstream 53C710
model does not fully support the ``-M 715`` machine type, particularly when
running HP-UX 10.20. This includes HP-UX Ignite (16700A) install media, which
drive the controller through the LASI 710 in ways the upstream model does not
handle.

The 53C710 executes SCSI SCRIPTS, a small program the guest driver loads into
host memory and the controller runs. HP-UX 10.20 exercises paths the upstream
model does not implement completely: disconnect and reselect for overlapped
tagged commands, asynchronous completion delivery matching how the real chip
behaves rather than reentrant delivery inside the interpreter, and the LASI
interrupt behavior the HP-UX driver depends on. The rewrite derives its engine
from QEMU's actively maintained LSI53C895A model and adapts it to the 53C710:
its register map, single-byte interrupt model, big-endian SCRIPTS and
table-indirect fetch (the part sits on the big-endian PA-RISC LASI bus), and
24-bit DMA counts. Behavior follows the NCR 53C710 Data Manual and
Programmer's Guide.

**What has been tested**

* HP-UX 10.20 installs and reboots into the installed system, on both the
  Ignite (16700A) media and a software install that overlaps tagged commands.
* Linux (lasi700 / 53c700) installs and boots Debian off the 710.
* NetBSD/hppa 9.4 and 10.1 (osiop) boot and enumerate disk and CD-ROM.

The companion LASI interrupt controller fix has already been merged upstream.
The device rewrite is the remaining piece, and lives on the ``ncr710-rewrite``
branch of this fork.

=============
Upstream Push
=============

I have worked to upstream this rewrite into QEMU, and I have the support of
Helge Deller, the hppa maintainer. Some reviewers on the mailing list consider
the rewrite too large to review comfortably as a single change, which is a
reasonable concern for a change of this size.

I proposed decomposing it into a series of ten or more smaller patches.
Because that decomposition has not yet been agreed as an acceptable approach,
I have held off reworking the code to fit it. The model is complete and
tested, and restructuring working code into a bisectable multi-patch series is
a substantial effort. I would rather settle the review approach first and then
do that work once than rework it speculatively. I am glad to proceed as soon
as there is agreement on the shape of the series.

===========
QEMU README
===========

QEMU is a generic and open source machine & userspace emulator and
virtualizer.

QEMU is capable of emulating a complete machine in software without any
need for hardware virtualization support. By using dynamic translation,
it achieves very good performance. QEMU can also integrate with the Xen
and KVM hypervisors to provide emulated hardware while allowing the
hypervisor to manage the CPU. With hypervisor support, QEMU can achieve
near native performance for CPUs. When QEMU emulates CPUs directly it is
capable of running operating systems made for one machine (e.g. an ARMv7
board) on a different machine (e.g. an x86_64 PC board).

QEMU is also capable of providing userspace API virtualization for Linux
and BSD kernel interfaces. This allows binaries compiled against one
architecture ABI (e.g. the Linux PPC64 ABI) to be run on a host using a
different architecture ABI (e.g. the Linux x86_64 ABI). This does not
involve any hardware emulation, simply CPU and syscall emulation.

QEMU aims to fit into a variety of use cases. It can be invoked directly
by users wishing to have full control over its behaviour and settings.
It also aims to facilitate integration into higher level management
layers, by providing a stable command line interface and monitor API.
It is commonly invoked indirectly via the libvirt library when using
open source applications such as oVirt, OpenStack and virt-manager.

QEMU as a whole is released under the GNU General Public License,
version 2. For full licensing details, consult the LICENSE file.


Documentation
=============

Documentation can be found hosted online at
`<https://www.qemu.org/documentation/>`_. The documentation for the
current development version that is available at
`<https://www.qemu.org/docs/master/>`_ is generated from the ``docs/``
folder in the source tree, and is built by `Sphinx
<https://www.sphinx-doc.org/en/master/>`_.


Building
========

QEMU is multi-platform software intended to be buildable on all modern
Linux platforms, OS-X, Win32 (via the Mingw64 toolchain) and a variety
of other UNIX targets. The simple steps to build QEMU are:


.. code-block:: shell

  mkdir build
  cd build
  ../configure
  make

Additional information can also be found online via the QEMU website:

* `<https://wiki.qemu.org/Hosts/Linux>`_
* `<https://wiki.qemu.org/Hosts/Mac>`_
* `<https://wiki.qemu.org/Hosts/W32>`_


Submitting patches
==================

The QEMU source code is maintained under the GIT version control system.

.. code-block:: shell

   git clone https://gitlab.com/qemu-project/qemu.git

When submitting patches, one common approach is to use 'git
format-patch' and/or 'git send-email' to format & send the mail to the
qemu-devel@nongnu.org mailing list. All patches submitted must contain
a 'Signed-off-by' line from the author. Patches should follow the
guidelines set out in the `style section
<https://www.qemu.org/docs/master/devel/style.html>`_ of
the Developers Guide.

Additional information on submitting patches can be found online via
the QEMU website:

* `<https://wiki.qemu.org/Contribute/SubmitAPatch>`_
* `<https://wiki.qemu.org/Contribute/TrivialPatches>`_

The QEMU website is also maintained under source control.

.. code-block:: shell

  git clone https://gitlab.com/qemu-project/qemu-web.git

* `<https://www.qemu.org/2017/02/04/the-new-qemu-website-is-up/>`_

A 'git-publish' utility was created to make above process less
cumbersome, and is highly recommended for making regular contributions,
or even just for sending consecutive patch series revisions. It also
requires a working 'git send-email' setup, and by default doesn't
automate everything, so you may want to go through the above steps
manually for once.

For installation instructions, please go to:

*  `<https://github.com/stefanha/git-publish>`_

The workflow with 'git-publish' is:

.. code-block:: shell

  $ git checkout master -b my-feature
  $ # work on new commits, add your 'Signed-off-by' lines to each
  $ git publish

Your patch series will be sent and tagged as my-feature-v1 if you need to refer
back to it in the future.

Sending v2:

.. code-block:: shell

  $ git checkout my-feature # same topic branch
  $ # making changes to the commits (using 'git rebase', for example)
  $ git publish

Your patch series will be sent with 'v2' tag in the subject and the git tip
will be tagged as my-feature-v2.

Bug reporting
=============

The QEMU project uses GitLab issues to track bugs. Bugs
found when running code built from QEMU git or upstream released sources
should be reported via:

* `<https://gitlab.com/qemu-project/qemu/-/issues>`_

If using QEMU via an operating system vendor pre-built binary package, it
is preferable to report bugs to the vendor's own bug tracker first. If
the bug is also known to affect latest upstream code, it can also be
reported via GitLab.

For additional information on bug reporting consult:

* `<https://wiki.qemu.org/Contribute/ReportABug>`_


ChangeLog
=========

For version history and release notes, please visit
`<https://wiki.qemu.org/ChangeLog/>`_ or look at the git history for
more detailed information.


Contact
=======

The QEMU community can be contacted in a number of ways, with the two
main methods being email and IRC:

* `<mailto:qemu-devel@nongnu.org>`_
* `<https://lists.nongnu.org/mailman/listinfo/qemu-devel>`_
* #qemu on irc.oftc.net

Information on additional methods of contacting the community can be
found online via the QEMU website:

* `<https://wiki.qemu.org/Contribute/StartHere>`_
