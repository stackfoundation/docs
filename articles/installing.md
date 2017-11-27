# Installing Sandbox

Sandbox is released as a Zip file containing a set of cross-platform binaries and scripts. You can download the Zip from the [Downloads](/downloads) page.

Installing Sandbox is done by extracting this Zip file into your project's root directory. Sandbox is designed to live together with your project, and to be committed into your project's repo. That gives anyone that checks out your repo access to Sandbox, without needing to download it themselves. Sandbox is tiny: all the files contained in the Zip file are less than 200KB. The small size means you can commit Sandbox to your repo without being concerned about the impact on your repo size.

When you extract the Zip, you will see the following files:

- `sbox.exe`
- `sbox`
- `sbox-cli/`
  - Cross-platform binaries

The `sbox` file at the root is a Unix shell script, and is the way Sandbox is invoked on Linux and macOS. When the script is run, it detects the current platform, and launches one of the binaries in the `sbox-cli` folder, based on the current platform. The `sbox.exe` file is the binary for running Sandbox on Windows.
