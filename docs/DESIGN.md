# Design

## Goal
Bring libvirt to Android so a droidtop device can run full virtual machines, using KVM where the
device has it. Compatibility, not speed: the fast paths for running software stay droidtop's
containers, Box64 and Wine.

## Root module: owns libvirt
- **Format:** the standard root-manager module layout (`module.prop`, `post-fs-data.sh`,
  `service.sh`, `customize.sh`, optional `webroot/`), so it installs in Magisk, KernelSU and APatch.
  Nothing Magisk-specific.
- **Payload:** `libvirtd` (or the modular `virtqemud` + `virtlogd`), `virsh`, and
  `qemu-system-aarch64` / `qemu-system-x86_64` plus firmware (EDK2/OVMF, SeaBIOS), built for Android
  arm64-v8a AND x86_64.
- **Runtime layout:** state under `/data/adb/libvirt/` (config, domain XML, images, logs). A
  `/run`-style tmpfs for sockets and pid files, since Android has no `/run`.
- **Exposure, as libvirt normally is:** `virtqemud-sock` / `libvirt-sock` (read-write) and
  `libvirt-sock-ro` UNIX sockets, plus optional TCP+TLS listening, off by default.
- **Access control (module settings):**
  - which Android apps (by uid/package) may use the read-write and read-only sockets;
  - whether network listening is on, and its TLS certificates;
  - storage pool locations;
  - network mode (user-mode NAT by default; tap/bridge via `/dev/tun` when chosen).

  Android has no polkit, so access is enforced by socket ownership and ACLs set by the module.
- **Settings UI:** the module's own WebUI (KernelSU/APatch WebUI, and MMRL, which renders module
  WebUIs), editing the same config files the daemons read. One config, two editors: the WebUI, and
  the droidtop plugin below.
- **Acceleration:** at start, detect `/dev/kvm` and advertise `kvm` domains in libvirt's
  capabilities; otherwise only `qemu` (TCG) domains.

## droidtop plugin: hooks libvirt
- Talks to the module's socket with the ordinary libvirt RPC protocol. No private side channel.
- **In droidtop:**
  - VMs appear as items in Desktop mode and in the library (as a source);
  - start, stop, snapshot and create;
  - a status tile in the Quick Menu;
  - a settings page that edits the module's config (the same access and config options as the
    WebUI).
- **For other plugins:** exposes a VM management API through droidtop's plugin broker
  (docs/plugin-api.md: plugin-provided APIs), gated by droidtop permissions.
- Needs root, so it declares a requirement on a root provider plugin. It is an enhancement: nothing
  in droidtop depends on it.

## Open questions (owner decisions)
1. **VM display path.** QEMU's SPICE/VNC is a remote-display protocol. droidtop's rule is that
   streaming lives in windowcast, so should the display go through windowcast's VNC/SPICE backend
   (tracker: windowcast RDP/VNC backends)?
2. **Build source.** Cross-compile libvirt/QEMU with the Android NDK ourselves, or start from an
   existing Android port's build recipes (e.g. Termux's QEMU packages) and adapt them?
3. **Daemon model.** Monolithic `libvirtd` or modular `virtqemud`? Modular is upstream's direction.
