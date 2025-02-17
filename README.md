
# osm-tags-transform

This command line tool can transform the tags of OSM objects in the input file
according to rules defined in a Lua script and write them out again. Objects
can also be filtered.

It can be used, for instance, to canonicalize OSM tags, remove unwanted tags or
objects, or transliterate names.

## Usage

Call with `--help` to get usage information. Typical use:

```
osm-tags-transform -c some_config.lua input.osm.pbf -o output.osm.pbf
```

## Lua Config File

The Lua config file has to define three callback functions which are called
for each node, way, or relation, respectively. The functions are not called
for objects without tags, they are always copied "as is" to the output.

```
function ott.process_node(object)
    ...
end

function ott.process_way(object)
    ...
end

function ott.process_relation(object)
    ...
end
```

The `object` parameter of those functions has the field `id` with the OSM
object id and the field `tags`, a Lua table with all the tags.

The functions must return either `true` if the object should be copied to
the output "as is", `false` if the object should not appear in the output or
a Lua table with the tags it should have.

## Geometry Processing

By default there is no geometry processing: There are no node locations or way
geometries etc. available in Lua.

But in some cases you want to take the geometry into account when transforming
tags, for instance to transliterate names based on the country they are in or
to process `ref` tags for highway shields depening on the country. In this case
you can enable geometry processing with the `-g bbox` or `--geom-proc=bbox`
command line option.

If geometry processing is enabled, the `object` parameter to the Lua callback
function has a field `bbox` which contains the bounding box of the object
(as an array of four fields [left, bottom, right, top]).

Note the node locations and way bounding boxes need to be stored somewhere
for this to work which can need quite a lot of memory. The way bounding boxes
are always stored in memory. You can use the `-i` or `--index-type` command
line option to change the index type used for the node locations. Use the
`-I` or `--show-index-types` option to see what index types are available.
See the [osmium-index-types manpage](https://docs.osmcode.org/osmium/latest/osmium-index-types.html)
which applies for this as well and the [chapter on indexes](
https://osmcode.org/osmium-concepts/#indexes) in the Osmium Concepts Manual
for some more information.

## Prerequisites

osm-tags-transform needs the following libraries:

* [bzip2](http://www.bzip.org/)
* [expat](https://libexpat.github.io/)
* [libosmium](https://osmcode.org/libosmium)
* [protozero](https://github.com/mapbox/protozero)
* [zlib](https://www.zlib.net/)
* [Lua](https://www.lua.org/), optionally Lua JIT

On Debian/Ubuntu you can install these with:

```
apt install \
    libbz2-dev \
    libexpat1-dev \
    liblua5.3-dev \
    libosmium2-dev \
    libprotozero-dev \
    zlib1g-dev
```

## Compiling

You need a C++ compiler with C++14 support.

Compile as usual with CMake with something like:

```
mkdir build
cd build
cmake ..
make
```

You can set these options:

* Set `BUILD_TESTS=ON` if you want to build the tests.
* Set `WITH_LUAJIT=ON` if you want to build with LuaJIT support.

## License

Copyright (C) 2022  Jochen Topf (jochen@topf.org)

This program is available under the GNU GENERAL PUBLIC LICENSE Version 3.
See the file LICENSE.txt for the complete text of the license.

