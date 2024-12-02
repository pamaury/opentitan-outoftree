# Copyright lowRISC contributors (OpenTitan project).
# Licensed under the Apache License, Version 2.0, see LICENSE for details.
# SPDX-License-Identifier: Apache-2.0

load("@bazel_skylib//lib:dicts.bzl", "dicts")
load(
    "@@//rules/opentitan:defs.bzl",
    "EARLGREY_CW340_TEST_ENVS",
    "EARLGREY_SILICON_OWNER_ROM_EXT_ENVS",
    "EARLGREY_TEST_ENVS",
    "OPENTITAN_CPU",
    "dv_params",
    "ecdsa_key_for_lc_state",
    "fpga_params",
    "opentitan_binary",
    "opentitan_test",
    "silicon_params",
    "verilator_params",
)

package(default_visibility = ["//visibility:public"])

opentitan_test(
    name = "my_smoketest",
    srcs = ["my_smoketest.c"],
    exec_env = dicts.add(
        {
            "@@//hw:sim_dv": None,
        },
    ),
    deps = [
        "@@//hw/top:devicetables",
        "@@//sw/device/lib/arch:device",
        "@@//sw/device/lib/base:mmio",
        "@@//sw/device/lib/dif:uart",
        "@@//sw/device/lib/runtime:hart",
        "@@//sw/device/lib/testing/test_framework:ottf_main",
    ],
)