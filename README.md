# smartos-isomerge

Unpack, Add Directories, Files, Symlinks, and Repack a SmartOS ISO

## Introduction

If you want to add (or replace) a few files in a SmartOS ISO, this utility will
unpack the ISO, merge files and repack the ISO automatically.  It should work
on any illumos system, provided you have `root` access in a zone that enables
the use of `lofiadm(1M)`, `ufs(7FS)` and `hsfs(7FS)`.

## Zone Setup

Example configuration for a zone used for `smartos-isomerge`.  For existing zones, ```vmadm update ID fs_allowed=hsfs,lofs,ufs``` should work.

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

```
# ./merge.js < input.json
 * mkdir /var/tmp/ISOMERG
 * lofi add /sysmgr/apps/smartos/releases/smartos-20120223T221136Z.iso
    - device: /dev/lofi/1
 * fstyp /dev/lofi/1
    - type: hsfs
 * mkdir /var/tmp/ISOMERG/iso
....
...
..
 * mkisofs /var/tmp/ISOMERG/output.iso
 * done
```
