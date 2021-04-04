# ARDEDUP - image (and document) ARchival and DEDUPlication

ARDEDUP will scan media such as external hard drives and make archival
copies of all images (and optionally other file types) it can find. It
stores these in a unified way based on a hash value of the file
contents. This removes duplicate files. 

At the same time, a database of all copied files is created that
retains all the important meta information such as the original file name,
name of the source disk it came from (optional), file
creation/modification times, etc.

For images a separate _difference hash_ is computed that can be used
to identify and match low-resolution copies of larger images, and to
group similar images. 

Apart from the scanning (which includes the archival) of media, this
command line program can list and search the file database in various
ways.

The database is stored in JSON format, an ubiquitous human-readable
data format, which should aid further processing in your programming
language of choice.

## Installation

ARDEDUP was developed using Python 3.9.1 on MacOS. It should work on
any UNIX-type system with Python >= 3.9.1, but nothing has been
tested.

To install this software, simply create a virtual environment
```bash
    python -m venv venv
    . venv/bin/activate
```
and install the library dependencies with
```bash
    python -m pip install -r requirements.txt
```

Now you should be all set.

## Usage

The command line interface is well documented, simply run
```bash
    ./ardedup --help
```
for a list of available commands. The main ones are as follows.

### `ardedup scan`

This is used to scan a directory (e.g., the mounting point of an
external hard drive) and to collect all relevant files in a sub folder
`./data` along with the corresponding meta information in the JSON
database `./data.json`.

Assuming a 2.5" hard drive labeled "Photos 2015-2018" is attached to
my Mac and mounted at the locations `/Volumes/2015`, `/Volumes/2016`,
`/Volumes/2017`, and `/Volumes/2018`, I would issue the command
```bash
    ./ardedup scan --diskname "Photos 2015-2018" /Volumes/201?
```

ARDEDUP will now scan the four directories for images (as well as a
few document file formats, this can be configured using a command line
option, see `./ardedup scan --help`).

All images files will be stored under `./data` with filenames such as
`./data/e4/394c307b4df83b3203f6ffc4aa4e37c6b357fc`, corresponding to
the SHA1 hash of the content of the file (yes, this is a content
addressable file system).

At the same time, a JSON database entry similar to the following one
is created:
```json
        "22858": {
            "path": "/Volumes/2017/DCIM1234.jpg",
            "sha1": "e4394c307b4df83b3203f6ffc4aa4e37c6b357fc",
            "diskname": "Photos 2015-2018",
            "dev": 16777226,
            "size": 1234567,
            "atime": 1282813761796875000,
            "ctime": 1272809616687500000,
            "mtime": 1264134092000000000,
            "imgtype": "png",
            "dhash128": "4f03479b3a34363410364f9f1a640a74",
            "im_mode": "P",
            "im_size": [
                3000,
                2000
            ]
        },
```

### `ardedup ls`

Since this is not very humanly readable, ARDEDUP has a `ls` command
that can list all the disk names as well as files, filtered by
appropriate constraints on size or name (supporting regular
expressions), which produces a nice, colorized, tabular output.

For example, to list all disks, you can
```bash
    ./ardedup ls disks
```
and to list all image files that contain the word `DCIM` and have a
resolution of 3000x2000 or more, you could
```bash
    ./ardedup ls files --min-size 3000 2000 --file-regex DCIM
```

### `ardedup stat`

To just get an overview of the number of files and disks known to the
database './data.json', use this command. The output looks something
like this:

```
Statistic                     #
----------------------------  -----
Total db entries              28390
Unique SHA1s                  9575
Image files (non-unique)      13295
Image files (unique)          5267
Image dhash128 (unique)       5035
Combined file sizes          10G
Combined image file sizes    8G
Total storage used            5G
Total storage used by images  5G
```

## Identification of identical and similar images

Images may look the same, even though they are stored in different
formats or at different resolutions. To identify such duplicates, a
SHA1 has is useless, as it changes even the the most minute variation
of a file. The 128 bit _difference hash_ used here aims to provide 
(bit-wise) similar outputs for similar _looking_ inputs. 

To this end, the Hamming distance (=number of bits differing between
the respective difference hashes) can be used to identify how close
two images are to each other. This is easy to compute by converting
the hexadecimal hashes into binary strings and counting of
element-wise differences.

No functionality to identify similar images has been implemented in
ARDEDUP yet, but the difference hashes are already provided. 

## License

```
Copyright (c) 2021 Björn Rüffer

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU Affero General Public License as
published by the Free Software Foundation, either version 3 of the
License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but
WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
Affero General Public License for more details.

A copy of the GNU Affero General Public License is available from
<https://www.gnu.org/licenses/>.
```
