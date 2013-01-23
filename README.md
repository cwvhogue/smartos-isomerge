# smartos-isomerge

Unpack, Add Directories, Files, Symlinks, and Repack a SmartOS ISO

## Introduction

If you want to add (or replace) a few files in a SmartOS ISO, this utility will
unpack the ISO, merge files and repack the ISO automatically.  It should work
on any illumos system, provided you have `root` access in a zone that enables
the use of `lofiadm(1M)`, `ufs(7FS)` and `hsfs(7FS)`.

## Zone Setup

Example configuration for a zone used for `smartos-isomerge`.

For existing zones, ```vmadm update ID fs_allowed=hsfs,lofs,ufs``` should work.

```javascript
{
  "alias": "isomerge",
  "autoboot": true,
  "brand": "joyent",
  "fs_allowed": "hsfs,lofs,ufs",
  "image_uuid": "fdea06b0-3f24-11e2-ac50-0b645575ce9d",
  "max_physical_memory": 1024,
  "nics": [
    {
      "nic_tag": "admin",
      "ip": "dhcp"
    }
  ],
  "resolvers": [
    "8.8.8.8",
    "8.8.4.4"
  ]
}
```

Use `pkgsrc` to install the `cdrtools` inside the zone for access to the `mkisofs` utility:

```bash
# pkgin in cdrtools
```

## Usage

First, create a JSON file to provide a working directory, and ISO and a list of
files to the utility.  e.g.

```javascript
{
  "workdir": "/var/tmp/isomerge-quagga",
  "inputiso": "/sysmgr/apps/smartos/releases/smartos-20130111T010112Z.iso",
  "mergedirs": {
    "/usr/sbin": {
      "owner": "root",
      "group": "bin",
      "perms": "0755"
    }
  },
  "mergefiles": {
    "/usr/sbin/quaggaadm": {
      "owner": "root",
      "group": "bin",
      "perms": "0555",
      "src": "/content/quagga/usr/sbin/quaggaadm"
    }
  },
  "mergelinks": {
    "/usr/sbin/zebraadm": {
      "target": "quaggaadm"
    }
  }
}
```

Then, invoke the utility:

```bash
# ./merge.js < input.json
 * mkdir /var/tmp/isomerge-quagga
 * lofi add /sysmgr/apps/smartos/releases/smartos-20130111T010112Z.iso
    - device: /dev/lofi/2
 * fstyp /dev/lofi/2
    - type: hsfs
 * mkdir /var/tmp/isomerge-quagga/iso
 * mount hsfs /dev/lofi/2
       on /var/tmp/isomerge-quagga/iso
     opts ro
 * cp from /var/tmp/isomerge-quagga/iso
        to /var/tmp/isomerge-quagga/isounpack
 * umount /var/tmp/isomerge-quagga/iso
 * lofi remove /dev/lofi/2
 * lofi add /var/tmp/isomerge-quagga/isounpack/platform/i86pc/amd64/boot_archive
    - device: /dev/lofi/2
 * fstyp /dev/lofi/2
    - type: ufs
 * mkdir /var/tmp/isomerge-quagga/root
 * fsck /dev/lofi/2
 * mount ufs /dev/lofi/2
       on /var/tmp/isomerge-quagga/root
     opts rw nologging
 * mv from /var/tmp/isomerge-quagga/root/usr.lgz
        to /var/tmp/isomerge-quagga/tmpusr.lgz
 * lofi uncompress /var/tmp/isomerge-quagga/tmpusr.lgz
 * lofi add /var/tmp/isomerge-quagga/tmpusr.lgz
    - device: /dev/lofi/3
 * fstyp /dev/lofi/3
    - type: ufs
 * fsck /dev/lofi/3
 * mount ufs /dev/lofi/3
       on /var/tmp/isomerge-quagga/root/usr
     opts rw nologging
 * merging dirs...
 * mkdir /var/tmp/isomerge-quagga/root/usr/sbin
 * chown root:bin /var/tmp/isomerge-quagga/root/usr/sbin
 * chmod 0755 /var/tmp/isomerge-quagga/root/usr/sbin
 * ...done merging dirs
 * merging files...
 * cp from /content/quagga/usr/sbin/quaggaadm
        to /var/tmp/isomerge-quagga/root/usr/sbin/quaggaadm
 * chown root:bin /var/tmp/isomerge-quagga/root/usr/sbin/quaggaadm
 * chmod 0555 /var/tmp/isomerge-quagga/root/usr/sbin/quaggaadm
 * ...done merging files
 * merging links...
 * ln target quaggaadm
        dest /var/tmp/isomerge-quagga/root/usr/sbin/zebraadm
 * ...done merging links
 * umount /var/tmp/isomerge-quagga/root/usr
 * fsck /dev/lofi/3
 * lofi remove /dev/lofi/3
 * lofi compress /var/tmp/isomerge-quagga/tmpusr.lgz
 * mv from /var/tmp/isomerge-quagga/tmpusr.lgz
        to /var/tmp/isomerge-quagga/root/usr.lgz
 * umount /var/tmp/isomerge-quagga/root
 * fsck /dev/lofi/2
 * lofi remove /dev/lofi/2
 * mkisofs /var/tmp/isomerge-quagga/output.iso
 * done
```
