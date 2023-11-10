# Compiling Rust-for-Linux

Clone the repository and switch to the `rust-dev` branch:

```bash
git clone https://github.com/Rust-for-Linux/linux.git rust-for-linux
git checkout rust-dev
```

Install basic dependencies as per: <https://github.com/schwartx/rcoreos-rust-for-linux/blob/main/exercise1.md>

In the `rust-for-linux` directory, run:

```bash
make LLVM=1 rustavailable
```

This automatically detects any missing compile dependencies. For any errors encountered, solutions are mostly found in Documentation/rust/quick-start.rst. Before compiling, ensure you get:

```txt
Rust is available!
```

Reset compile configuration to default:

```bash
make ARCH=arm64 LLVM=1 O=build defconfig
```

Enter the compile configuration menu and select `Rust support`:

```bash
make ARCH=arm64 LLVM=1 O=build menuconfig
```

Navigate to the `./build` directory and start compiling:

```bash
cd ./build
make ARCH=arm64 LLVM=1 -j$(nproc)
```

After a lengthy wait...

Once compilation is complete, download a QEMU-compatible OS image from <https://gitlab.com/giomasce/dqib>: <https://gitlab.com/api/v4/projects/giomasce%2Fdqib/jobs/artifacts/master/download?job=convert_arm64-virt>

Unzipping the file yields:

```txt
 .
├──  image.qcow2
├──  initrd
├──  kernel
├──  readme.txt
├──  ssh_user_ecdsa_key
├──  ssh_user_ed25519_key
└──  ssh_user_rsa_key
```

Carefully read `./readme.txt` for usage instructions.

Run the following command in `./build`:

```bash
qemu-system-aarch64 -machine 'virt' -cpu 'cortex-a57' -m 1G -device virtio-blk-device,drive=hd -drive file=$HOME/dqib_arm64-virt/image.qcow2,if=none,id=hd -device virtio-net-device,netdev=net -netdev user,id=net,hostfwd=tcp::2222-:22 -kernel arch/arm64/boot/Image.gz -initrd $HOME/dqib_arm64-virt/initrd -nographic -append "root=LABEL=rootfs console=ttyAMA0"
```

Note: `-drive file=` points to the virtual disk file in the dqib directory, and `-initrd` also from the dqib directory, is a temporary root filesystem used during the Linux boot process.

After running the command, the system starts up, leading to the login interface.
