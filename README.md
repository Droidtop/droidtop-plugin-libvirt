# droidtop-plugin-libvirt

libvirt and QEMU on Android. Part of "droidtop makes an Android device into a full computer":
virtual machines give the broad OS compatibility libvirt and QEMU are known for (any Linux distro,
Windows, BSD, other architectures). This is a compatibility feature, not a performance one.

Two halves in one repo:

- **Root module** (`module/`). A standard root-manager module (the common module zip layout that
  Magisk, KernelSU and APatch all install). It owns libvirt: ships `libvirtd`/`virtqemud` and
  `qemu-system-*`, starts them at boot, and exposes libvirt **the way libvirt is normally exposed**:
  its usual UNIX sockets (and optional TCP/TLS), so `virsh`, `virt-manager` over the network, and any
  other libvirt client work unchanged. Its own settings control access and configuration.
- **droidtop plugin** (`droidtop-plugin/`). A droidtop plugin that hooks the module's libvirt
  directly: lists and manages VMs, shows them in droidtop (Desktop mode, library), and offers VM
  management to other plugins through droidtop's plugin API. It requires the module; it never
  reimplements it.

Acceleration: where the device exposes `/dev/kvm` (pKVM or KVM-capable kernels), QEMU uses it.
Everywhere else it falls back to QEMU's software emulation (TCG): slow, but it still runs.

Status: design only. See [docs/DESIGN.md](docs/DESIGN.md).

Licence: our code is Apache-2.0. Bundled libvirt (LGPL-2.1-or-later) and QEMU (GPL-2.0) keep their
own licences, with their sources published alongside every release.
