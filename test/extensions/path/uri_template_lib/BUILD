load(
    "//bazel:envoy_build_system.bzl",
    "envoy_cc_test",
    "envoy_package",
)

licenses(["notice"])  # Apache 2

envoy_package()

# Wildcard & Uri Pattern Matching

envoy_cc_test(
    name = "uri_template_test",
    srcs = ["uri_template_test.cc"],
    deps = [
        "//source/extensions/path/uri_template_lib",
        "//test/test_common:status_utility_lib",
        "@com_google_absl//absl/status",
        "@com_google_absl//absl/status:statusor",
        "@com_google_absl//absl/strings",
    ],
)

envoy_cc_test(
    name = "uri_template_internal_test",
    srcs = ["uri_template_internal_test.cc"],
    deps = [
        "//source/extensions/path/uri_template_lib:uri_template_internal_cc",
        "//test/test_common:status_utility_lib",
        "@com_google_absl//absl/status",
        "@com_google_absl//absl/status:statusor",
        "@com_google_absl//absl/strings",
        "@com_googlesource_code_re2//:re2",
    ],
)
