# Corpus structure

ceph.git/ceph-object-corpus is a submodule.:

    bin/   # misc scripts
    archive/$version/objects/$type/$hash  # a sample of encoded objects from a specific version

You can also mark known or deliberate incompatibilities between versions
with:

    archive/$version/forward_incompat/$type

The presence of a file indicates that new versions of code cannot decode
old objects across that `$version` (this is normally the case).

## How to generate an object corpus

We can generate an object corpus for a particular version of ceph using
the script of `script/gen-corpus.sh`, or by following the instructions
below:

1.  Checkout a clean repo (best not to do this where you normally work):

    ``` shell
    git clone ceph.git
    cd ceph
    git submodule update --init --recursive --progress
    ```

2.  Build with flag to dump objects to `/tmp/foo`:

    ``` shell
    rm -rf /tmp/foo ; mkdir /tmp/foo
    do_cmake.sh -DCMAKE_CXX_FLAGS="-DENCODE_DUMP_PATH=/tmp/foo"
    cd build
    make
    ```

3.  Start via vstart:

    ``` shell
    cd build
    MON=3 MGR=2 OSD=3 MDS=3 RGW=1 ../src/vstart.sh -n -x
    ```

4.  Use as much functionality of the cluster as you can, to exercise as
    many object encoder methods as possible:

    ``` shell
    bin/ceph osd pool create mypool
    bin/rados -p mypool bench 10 write -b 123
    bin/ceph osd out 0
    bin/ceph osd in 0
    bin/init-ceph restart osd.1
    for f in ../qa/workunits/cls/*.sh ; do PATH="bin:$PATH" $f ; done
    PATH="bin:$PATH" ../qa/workunits/rados/test.sh
    bin/ceph_test_librbd
    bin/ceph_test_libcephfs
    bin/init-ceph restart mds.a
    ../qa/workunits/rgw/run-s3tests.sh
    ```

5.  Stop:

    ``` shell
    ../src/stop.sh
    ```

6.  Import the corpus (this will take a few minutes):

    ``` shell
    ../src/test/encoding/import.sh /tmp/foo `bin/ceph-dencoder version` ../ceph-object-corpus/archive
    ../src/test/encoding/import-generated.sh ../ceph-object-corpus/archive
    ```

7.  Prune it! There will be a bazillion copies of various objects, and
    we only want a representative sample.:

    ``` shell
    pushd ../ceph-object-corpus
    bin/prune-archive.sh
    popd
    ```

8.  Verify the tests pass:

    ``` shell
    ctest -R readable.sh
    ```

9.  Commit it to the corpus repo and push:

    ``` shell
    pushd ../ceph-object-corpus
    git checkout -b wip-new
    git add archive/`../build/bin/ceph-dencoder version`
    git commit -m `../build/bin/ceph-dencoder version`
    git remote add cc git@github.com:ceph/ceph-object-corpus.git
    git push cc wip-new
    popd
    ```

10. Go test it out:

    ``` shell
    cd my/regular/tree
    cd ceph-object-corpus
    git fetch origin
    git checkout wip-new
    cd ../build
    ctest -R readable.sh
    ```

11. If everything looks good, update the submodule master branch, and
    commit the submodule in ceph.git.
