
# GPU Passthrough to LXC Container (Intel GPU)

This guide will help you pass through an Intel-based GPU to an LXC container.

## Steps:

### 1. Create a Linux container
Follow the normal procedure to create a new LXC container on your system.

### 2. Change Device Permissions
Allow other users to access the GPU devices:
```bash
chmod 666 /dev/dri/card0
chmod 666 /dev/dri/renderD128
```

### 3. Add Udev Rules for Persistent Permissions
Create a new file with udev rules to persistently grant permissions across reboots:
```bash
cat > /etc/udev/rules.d/99-intel-chmod666.rules << 'EOF'
KERNEL=="renderD128", MODE="0666"
KERNEL=="card0", MODE="0666"
EOF
```

### 4. Update LXC Configuration
Modify your container's configuration file found in `/etc/pve/lxc/xxx.conf` (replace `xxx` with your container ID) by adding the following lines:

```bash
lxc.cgroup2.devices.allow: c 226:0 rwm
lxc.cgroup2.devices.allow: c 226:128 rwm
lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file 0, 0
lxc.mount.entry: /dev/dri dev/dri none bind,optional,create=dir
```

## Verification

After applying these changes, restart your LXC container. You can verify the successful passthrough inside the host or container by running `intel_gpu_top`, which requires installing the package `intel-gpu-tools`.

## Note:
- Ensure your GPU and hardware setup is compatible with virtualization and container-based GPU sharing.
- The device numbers `226:0` and `226:128` may vary based on your hardware. Please verify the correct device numbers for your system.
```

Please replace the `xxx` in the configuration file path with your actual container ID and ensure that the device numbers correspond to your specific hardware setup. This guide assumes familiarity with Linux containers and command-line operations.
