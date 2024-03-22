psf-tuf-runbook
===============

This repository contains a runbook and supporting program for the JuliCA key generation and signing ceremonies.
The procedures documented here are designed to implement the security policies for offline keys for WebTrust CA Audit.

## Notation

This document is designed to be read as a *runbook* -- a collection of discrete instructions
with remediation steps that, if followed correctly, should result in the intended effects.

We use the following notation:

* **DO** *actions*: Perform the following actions.
* **IF** *condition* **THEN** *actions*: If *condition* is met, then perform the following *actions*.
* **GO TO** *heading*: Go to the referenced heading in the runbook and perform the stated actions
thereon.
* **END**: You've reached an end state.

In addition, this document uses [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119)
to describe optional and mandatory steps.

## Start

1. **DO GO TO** [Prepare the environment](#prepare-the-environment).

## Prepare the environment

1. **DO** perform the [pre-ceremony](PRE-CEREMONY.md).

1. **DO** Start streaming the ceremony using the communication computer.

1. **IF** you have a phone or other personal devices, **THEN** set them on airplane mode.

1. **DO** boot the trusted offline machine (the Raspberry Pi "ceremony computer"), and log into it using the credentials provided
during the pre-ceremony.

1. **DO** mount the flash storage stick:

    ```bash
    $ sudo mount -t vfat /dev/sda1 /media/ceremony-products -o umask=000
    ```

1. **DO** change directory to the runbook directory:

    ```bash
    $ cd ~/psf-tuf-runbook
    ```

1. **DO** take pictures of each HSM, in their tamper-evident bags.

1. **DO** remove `Nitrokey HSM-1` (keytype: P-384) from its tamper-evident bag and **GO TO**
[Provisioning the Nitrokey HSM](#provisioning-the-nitrokey-hsm)

1. **DO** remove `Nitrokey HSM-2` (keytype: P-256) from its tamper-evident bag and **GO TO**
[Provisioning the Nitrokey HSM](#provisioning-the-nitrokey-hsm)

1. **DO** remove `Nitrokey HSM-3` (keytype: P-384) from its tamper-evident bag and **GO TO**
[Provisioning the Nitrokey HSM](#provisioning-the-nitrokey-hsm)

1. **DO** copy the ceremony products to the flash storage stick:

    ```bash
    cp -R ./ceremony-products /media/ceremony-products
    ```

1. **DO** unmount the flash storage stick:

    ```bash
    $ sync
    $ sudo umount /media/ceremony-products
    ```

1. **DO** perform the [post-ceremony steps](#post-ceremony).

1. **END**

## Provisioning the Nitrokey HSM

*Time estimate: 10 minutes*.

1. **DO** determine the current Security Officer PIN ("SO-PIN"):

    1. **IF** the Nitrokey has not been provisioned before, **THEN** the SO-PIN is `3537363231383830`.

    1. **IF** the Nitrokey has been previously provisioned, **THEN** the SO-PIN should have been retained from the previous provisoning.

1. **DO** insert the Nitrokey HSM into the trusted offline computer.

1. **DO** ensure that exactly one (1) Nitrokey HSM is inserted into the trusted offline computer.

1. **DO** run the `nitrohsm-provision` script, using your SO-PIN:

    ```bash
    $ nitrohsm-provision --so-pin SO-PIN
    ```

1. **DO** wait for this prompt:

    ```
    !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    !!!                    DANGER!                    !!!
    !!!                                               !!!
    !!!   This program will reset and reprovision     !!!
    !!!   your Nitrokey HSM for TUF purposes.         !!!
    !!!                                               !!!
    !!!   Make sure to read the runbook before        !!!
    !!!   using this program. Failure to do so        !!!
    !!!   will cause PERMANENT key loss and MAY       !!!
    !!!   leave your HSM in an unusable state.        !!!
    !!!                                               !!!
    !!!   Hit "y" (case insensitive) to continue.     !!!
    !!!                                               !!!
    !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    ```

1. **DO** hit `y` once ready to continue.

1. **DO** wait for the following output and prompt:

    ```
    Successfully discovered a Nitrokey HSM with Slot #0
    Continue with factory reset? This step is IRREVERSIBLE! [y/N]
    ```

1. **DO** hit `y` once ready to continue.

1. **DO** wait for the following output and prompt:

    ```
    Success! Reinitialized the HSM.
    Enter your NEW Security Officer PIN:
    ```

1. **DO** enter the *new* Security Officer PIN generated for this Nitrokey during the pre-ceremony.

1. **DO** wait for the following prompt:

    ```
    Re-enter your NEW Security Officer PIN:
    ```

1. **DO** re-enter the *new* Security Officer PIN.

1. **DO** wait for the following prompt:

    ```
    Enter your NEW user PIN:
    ```

1. **DO** enter the *new* user PIN generated for this Nitrokey during the pre-ceremony.

1. **DO** wait for the following prompt:

    ```
    Re-enter your NEW user PIN:
    ```

1. **DO** re-enter the *new* user PIN.

1. **DO** wait for the following output:

    ```
    Success! We've reinitialized the Nitrokey with a new SO PIN and user PIN.
    Use this serial number when doing key generation: XXXXXXXXXXX
    ```

1. **DO** write down the serial number printed above on a *separate* piece of loose-leaf.

1. **DO** run the `generate-nitrohsm-keys` script, using your key type according to the following rules:

    * **IF** your keytype is "P-256", **THEN** pass `--type p256`
    * **IF** your keytype is "P-384", **THEN** pass `--type p384`

    ```bash
    $ generate-nitrohsm-keys --type KEY-TYPE --serial XXXXXXXXXXX
    ```

1. **DO** wait for the following prompt:

    ```
    Enter your user PIN:
    ```

1. **DO** enter the *new* user PIN.

1. **DO** check for the following files in the runbook directory:

    ```
    ceremony-products/XXXXXXXXXXX/XXXXXXXXXXX_root_pubkey.pub
    ceremony-products/XXXXXXXXXXX/XXXXXXXXXXX_root_pubkey.pem
    ceremony-products/XXXXXXXXXXX/XXXXXXXXXXX_targets_pubkey.pub
    ceremony-products/XXXXXXXXXXX/XXXXXXXXXXX_targets_pubkey.pem
    ```

1. **DO** remove the HSM.

1. **DO** label a tamper-evident bag with the HSM's signing body ID and serial number.

1. **DO** seal the provisioned HSM and folded Security Officer and user PINs in the tamper-evident bag.

1. **DO** hold the sealed tamper-evident bag up to the camera of the communication computer.

## Post-ceremony

1. **DO** insert the flash stick into the communication computer.

1. **DO** navigate to the runbook repository in a new terminal.

1. **DO** create a new branch:

    ```bash
    git checkout -b ceremony-YYYY-MM-DD
    ```

    Where `YYYY-MM-DD` is the current date.

1. **DO** create the following new subdirectories:

    ```bash
    mkdir -p ceremony/YYYY-MM-DD/ceremony-products
    mkdir -p ceremony/YYYY-MM-DD/images
    ```

    Where `YYYY-MM-DD` is the current date.

1. **DO** copy the contents of the ceremony flash stick into the `ceremony-products` subdirectory.

1. **DO** copy all images taken of the HSMs and tamper-evident bags into the `images` subdirectory.

1. **DO** commit the results, signing with a publicly announced PGP key:

    ```bash
    git add ceremony/YYYY-MM-DD
    git commit -S
    ```

    Where `YYYY-MM-DD` is the current date.

1. **DO** push the branch to [psf/psf-tuf-runbook](https://github.com/psf/psf-tuf-runbook) and open
a PR for review.

    ```bash
    git push origin ceremony-YYYY-MM-DD
    ```

    Where `YYYY-MM-DD` is the current date.

1. **DO** await for PR approval, and confirm that the branch is merged into the `main` branch.

    * You **MAY** delete the original ceremony branch once merged.

1. **DO** securely destroy the SD card used for the runbook image **OR** zero it:

    ```bash
    $ diskutil unmountDisk /dev/rdiskN
    $ sudo dd bs=4m if=/dev/zero of=/dev/rdiskN
    ```
