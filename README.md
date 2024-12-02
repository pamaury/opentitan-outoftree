# opentitan-outoftree

This repository is an example of private, out-of-tree repository that uses OpenTitan to compile tests.

# opentitan submodule

This repository uses `opentitan` as a submodule. The first time you checkout this repository, you need
to initialize it with:
```console
git submodule init
```
Presently, this points to a particular version of the multitop work, together with a single patch to
the `WORKSPACE` file:
```diff
diff --git a/WORKSPACE b/WORKSPACE
index 14ec5fde85..fa19ef990d 100644
--- a/WORKSPACE
+++ b/WORKSPACE
@@ -188,3 +188,8 @@ hyperdebug_repos()
 register_toolchains(
     "//rules/opentitan:localtools",
 )
+
+local_repository(
+    name = "outoftree",
+    path = "..",
+)
```
If you change this to point to a fork of the opentitan repository, make sure to include this change
in the `WORKSPACE` file.

## Updating the submodule

If you want to change what the submodule points to, simply go into the `opentitan` directory
and use regular git commands to change to whatever you want. After that, make sure to
commit this change in the master repository.
```console
cd opentitan
git fetch
# Select a particular hash.
git reset--hard MY_HASH
# Or just rebase the branch
git pull --rebase
# Go back to private repo.
cd ..
# Commit change
git commit -va
```

# Compiling a test

To compile the example smoketest, go into the `opentitan` directory and compile it as usual
but use the `@outoftree` repository to refer to private tests:
```console
cd opentitan
./bazelisk.sh build --//hw/top=darjeeling @outoftree//tests:my_smoketest
```

# Gathering tests artefacts

Once you have compiled a tests and are happy with the code, you can get the paths to the artefacts as
follows:
```console
cd opentitan
./bazelisk.sh build --//hw/top=darjeeling @outoftree//tests:my_smoketest
./bazelisk.sh cquery --//hw/top=darjeeling --starlark:file=../get_artefacts.cquery --output=starlark @outoftree//tests:my_smoketest
```
This will for example print:
```
bazel-out/k8-fastbuild-ST-32b82a8f3fd7/bin/external/outoftree/tests/my_smoketest_sim_dv.bash
bazel-out/k8-fastbuild-ST-4b94735babb6/bin/sw/host/opentitantool/opentitantool
bazel-out/k8-fastbuild-ST-32b82a8f3fd7/bin/sw/device/lib/testing/test_rom/test_rom_sim_dv.elf
bazel-out/k8-fastbuild-ST-32b82a8f3fd7/bin/sw/device/lib/testing/test_rom/test_rom_sim_dv.bin
bazel-out/k8-fastbuild-ST-32b82a8f3fd7/bin/sw/device/lib/testing/test_rom/test_rom_sim_dv.39.scr.vmem
bazel-out/k8-fastbuild-ST-32b82a8f3fd7/bin/sw/device/lib/testing/test_rom/test_rom_sim_dv.dis
bazel-out/k8-fastbuild-ST-32b82a8f3fd7/bin/sw/device/lib/testing/test_rom/test_rom_sim_dv.logs.txt
bazel-out/k8-fastbuild-ST-32b82a8f3fd7/bin/sw/device/lib/testing/test_rom/test_rom_sim_dv.rodata.txt
bazel-out/k8-fastbuild-ST-32b82a8f3fd7/bin/sw/device/lib/testing/test_rom/test_rom_sim_dv.map
bazel-out/k8-fastbuild-ST-32b82a8f3fd7/bin/external/outoftree/tests/my_smoketest_sim_dv.elf
bazel-out/k8-fastbuild-ST-32b82a8f3fd7/bin/external/outoftree/tests/my_smoketest_sim_dv.bin
bazel-out/k8-fastbuild-ST-32b82a8f3fd7/bin/external/outoftree/tests/my_smoketest_sim_dv.64.scr.vmem
bazel-out/k8-fastbuild-ST-32b82a8f3fd7/bin/external/outoftree/tests/my_smoketest_sim_dv.dis
bazel-out/k8-fastbuild-ST-32b82a8f3fd7/bin/external/outoftree/tests/my_smoketest_sim_dv.logs.txt
bazel-out/k8-fastbuild-ST-32b82a8f3fd7/bin/external/outoftree/tests/my_smoketest_sim_dv.rodata.txt
bazel-out/k8-fastbuild-ST-32b82a8f3fd7/bin/external/outoftree/tests/my_smoketest_sim_dv.map
bazel-out/k8-fastbuild-ST-32b82a8f3fd7/bin/external/outoftree/tests/my_smoketest_sim_dv.64.vmem
```

# Creating a new test

In order to create a new test, follow the usual OpenTitan convention:
```python
opentitan_test(
    # Name of the tests.
    name = "my_smoketest",
    # Source code
    srcs = ["my_smoketest.c"],
    # Execution env: make sure to only use this one.
    exec_env = dicts.add(
        {
            "@@//hw:sim_dv": None,
        },
    ),
    # List of dependencies. If you refer to a dependency in the OT repo, prefix it by @@.
    # If you refer to a dependency in the private repo, use the "normal" syntax.
    deps = [
        "@@//hw/top:devicetables",
        "@@//sw/device/lib/arch:device",
        "@@//sw/device/lib/base:mmio",
        "@@//sw/device/lib/dif:uart",
        "@@//sw/device/lib/runtime:hart",
        "@@//sw/device/lib/testing/test_framework:ottf_main",
    ],
)
```
