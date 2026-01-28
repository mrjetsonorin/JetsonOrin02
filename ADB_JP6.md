DONT INSTALL FROM HERE:
https://developer.android.com/tools/releases/platform-tools

Is not arm compatible you dont believe me? Download it.
```bash
cd platform-tools
file adb
```
You want to see: `adb: ELF 64-bit LSB executable, ARM aarch64, dynamically linked, ...` but instead you see `ELF 64-bit LSB executable, x86-64
` And running it would fail instantly with: `cannot execute binary file: Exec format error`

Jeremy: oh on VM worked, cool, Your Linux VM is x86-64. Next.

Let's try this:
```bash
sudo apt update
sudo apt install -y adb
```
