# Fallback from TFRT to TF kernels and related utilities.
load(
    "//tensorflow:tensorflow.bzl",
    "if_oss",
    "tf_cc_test",
    "tf_cuda_library",
)
load("//tensorflow/core/platform:rules_cc.bzl", "cc_library")

package(
    default_visibility = ["//tensorflow/core/runtime_fallback:internal"],
    features = ["-layering_check"],
    licenses = ["notice"],
)

cc_library(
    name = "tfrt_op_kernel",
    srcs = ["tfrt_op_kernel.cc"],
    hdrs = ["tfrt_op_kernel.h"],
    visibility = ["//visibility:public"],
    deps = [
        ":attr_util",
        "@com_google_absl//absl/strings",
        "@llvm-project//llvm:Support",
        "//tensorflow/core/runtime_fallback/util:attr_util",
        "//tensorflow/core/tfrt/utils:error_util",
        "@tf_runtime//backends/common:eigencompat",
        "@tf_runtime//:core_runtime",
        "@tf_runtime//:hostcontext",
    ] + select({
        "//tensorflow:android": [
            "//tensorflow/core:portable_tensorflow_lib_lite",  # TODO(annarev): exclude runtime srcs
        ],
        "//conditions:default": [
            "//tensorflow/core/framework:tensor",
            "//tensorflow/core/framework:tensor_shape",
            "//tensorflow/core/platform:errors",
            "//tensorflow/core/platform:status",
            "//tensorflow/core/platform:stringpiece",
        ],
    }),
)

tf_cc_test(
    name = "tfrt_op_kernel_test",
    srcs = ["tfrt_op_kernel_test.cc"],
    tags = ["no_oss"],
    deps = [
        ":tfrt_op_kernel",
        "@tf_runtime//:core_runtime",
        "@tf_runtime//:hostcontext",
    ] + select({
        "//tensorflow:android": [
            "//tensorflow/core:portable_tensorflow_lib_lite",  # TODO(annarev): exclude runtime srcs
        ],
        "//conditions:default": [
            "//tensorflow/core:test",
            "//tensorflow/core:test_main",
            "//tensorflow/core/framework:types_proto_cc",
            "//tensorflow/core/lib/core:error_codes_proto_cc",
            "//tensorflow/core/util:padding",
        ],
    }),
)

cc_library(
    name = "attr_util",
    srcs = ["attr_util.cc"],
    hdrs = ["attr_util.h"],
    visibility = ["//visibility:public"],
    deps = [
        "@llvm-project//llvm:Support",
        "//tensorflow/core/runtime_fallback/util:attr_util",
        "@tf_runtime//:core_runtime",
        "@tf_runtime//:hostcontext",
    ] + select({
        "//tensorflow:android": [
            "//tensorflow/core:portable_tensorflow_lib_lite",  # TODO(annarev): exclude runtime srcs
        ],
        "//conditions:default": [
            "//tensorflow/core/framework:types_proto_cc",
            "//tensorflow/core/platform:errors",
            "//tensorflow/core/platform:status",
            "//tensorflow/core/platform:str_util",
            "//tensorflow/core/platform:stringpiece",
            "//tensorflow/core/util:padding",
        ],
    }),
)

tf_cc_test(
    name = "attr_util_test",
    srcs = ["attr_util_test.cc"],
    tags = ["no_oss"],
    deps = [
        ":attr_util",
        "@tf_runtime//:core_runtime",
        "@tf_runtime//:hostcontext",
        "@tf_runtime//:support",
    ] + select({
        "//tensorflow:android": [
            "//tensorflow/core:portable_tensorflow_lib_lite",  # TODO(annarev): exclude runtime srcs
        ],
        "//conditions:default": [
            "//tensorflow/core:test",
            "//tensorflow/core:test_main",
            "//tensorflow/core/platform:status",
            "//tensorflow/core/platform:types",
        ],
    }),
)

cc_library(
    name = "kernel_fallback_kernels_alwayslink",
    srcs = [
        "kernel_fallback_kernels.cc",
        "kernel_fallback_static_registration.cc",
    ],
    includes = [
        "third_party/tf_runtime/include",
    ],
    deps = [
        ":attr_util",
        ":kernel_fallback_execute",
        ":kernel_fallback_op_handler",
        ":kernel_fallback_tensor",
        "@llvm-project//llvm:Support",
        "@tf_runtime//:core_runtime",
        "@tf_runtime//:hostcontext",
        "@tf_runtime//:tensor",
        "@tf_runtime//backends/cpu:core_runtime",
    ] + select({
        "//tensorflow:android": [
            "//tensorflow/core:portable_tensorflow_lib_lite",  # TODO(annarev): exclude runtime srcs
        ],
        "//conditions:default": [
            "//tensorflow/core/framework:tensor",
        ],
    }),
    alwayslink = 1,
)

cc_library(
    name = "kernel_fallback_tensor",
    srcs = ["kernel_fallback_tensor.cc"],
    hdrs = ["kernel_fallback_tensor.h"],
    visibility = [
        "//tensorflow/core/runtime_fallback:internal",
        "//tensorflow/core/tfrt:__subpackages__",
    ],
    deps = [
        "@llvm-project//llvm:Support",
        "@tf_runtime//:hostcontext",
        "@tf_runtime//:support",
        "@tf_runtime//:dtype",
        "//tensorflow/core/runtime_fallback/util:tensor_util",
        "@tf_runtime//:tensor_alwayslink",
        "//tensorflow/core/runtime_fallback/util:type_util",
    ] + select({
        "//tensorflow:android": [
            "//tensorflow/core:portable_tensorflow_lib_lite",  # TODO(annarev): exclude runtime srcs
        ],
        "//conditions:default": [
            "//tensorflow/core/framework:tensor",
            "//tensorflow/core/framework:tensor_shape",
            "//tensorflow/core/framework:types_proto_cc",
            "//tensorflow/core/platform:status",
            "//tensorflow/c:tf_datatype",
            "//tensorflow/c:tf_tensor_internal",
        ],
    }),
)

cc_library(
    name = "conversion_lib",
    srcs = [
        "conversion/conversion.cc",
    ],
    hdrs = [
        "conversion/conversion.h",
    ],
    includes = [
        "third_party/tf_runtime/include",
    ],
    visibility = [
        "//tensorflow/core/runtime_fallback:internal",
        "//tensorflow/core/tfrt:__subpackages__",
    ],
    deps = [
        ":kernel_fallback_tensor",
        "//tensorflow/core/runtime_fallback/util:tensor_util",
        "//tensorflow/core/runtime_fallback/util:type_util",
        "@tf_runtime//:dtype",
        "@tf_runtime//:hostcontext",
        "@tf_runtime//:support",
        "@tf_runtime//:tensor_alwayslink",
    ] + select({
        "//tensorflow:android": [
            "//tensorflow/core:portable_tensorflow_lib_lite",  # TODO(annarev): exclude runtime srcs
        ],
        "//conditions:default": [
            "//tensorflow/core/framework:tensor",
        ],
    }),
)

cc_library(
    name = "kernel_fallback_tensor_conversion_alwayslink",
    srcs = [
        "conversion/static_registration.cc",
    ],
    visibility = ["//visibility:public"],
    deps = [
        ":conversion_lib",
        "@tf_runtime//:tensor",
    ],
    alwayslink = 1,
)

tf_cuda_library(
    name = "kernel_fallback_gpu_alwayslink",
    srcs = [
        "gpu/conversion_function.cc",
        "gpu/static_registration.cc",
    ],
    hdrs = [
        "gpu/conversion_function.h",
    ],
    compatible_with = [],
    # Only build this library with --config=cuda.
    tags = [
        "no_oss",
        "requires_cuda",
    ] + if_oss(["manual"]),
    visibility = ["//visibility:public"],
    deps = [
        ":kernel_fallback_execute_compat",
        ":kernel_fallback_tensor",
        "@com_google_absl//absl/strings",
        "//tensorflow/core/runtime_fallback/util:gpu_util",
        "//tensorflow/core/runtime_fallback/util:tensor_util",
        "//tensorflow/core/runtime_fallback/util:type_util",
        "@tf_runtime//:hostcontext",
        "@tf_runtime//:tensor",
        "@tf_runtime//backends/gpu:gpu_config",
        "@tf_runtime//backends/gpu:gpu_device",
        "@tf_runtime//backends/gpu:gpu_op_handler_alwayslink",
        "@tf_runtime//backends/gpu:gpu_tensor",
    ] + select({
        "//tensorflow:android": [
            "//tensorflow/core:portable_tensorflow_lib_lite",  # TODO(annarev): exclude runtime srcs
        ],
        "//conditions:default": [
            "//tensorflow/core:protos_all_cc",
            "//tensorflow/core/common_runtime/eager:tensor_handle",
        ],
    }),
    alwayslink = 1,
)

cc_library(
    name = "kernel_fallback_op_handler",
    srcs = ["kernel_fallback_op_handler.cc"],
    hdrs = ["kernel_fallback_op_handler.h"],
    visibility = ["//visibility:public"],
    deps = [
        ":kernel_fallback_execute_compat",
        ":kernel_fallback_tensor",
        ":kernel_fallback_tensor_conversion_alwayslink",
        "@tf_runtime//:core_runtime",
        "@tf_runtime//:hostcontext",
        "@tf_runtime//:support",
        "//tensorflow/core/runtime_fallback/runtime:kernel_utils",
        "@tf_runtime//:tensor_alwayslink",
    ] + select({
        "//tensorflow:android": [
            "//tensorflow/core:portable_tensorflow_lib_lite",  # TODO(annarev): exclude runtime srcs
        ],
        "//conditions:default": [
            "//tensorflow/core/platform:mutex",
            "//tensorflow/core/common_runtime/eager:context",
            "//tensorflow/core/framework:tensor",
        ],
    }),
    alwayslink = True,
)

cc_library(
    name = "kernel_fallback_execute",
    srcs = ["kernel_fallback_execute.cc"],
    hdrs = ["kernel_fallback_execute.h"],
    deps = [
        ":kernel_fallback_tensor",
        ":kernel_fallback_tensor_conversion_alwayslink",
        ":tfrt_op_kernel",
        "@llvm-project//llvm:Support",
        "@tf_runtime//:core_runtime",
        "@tf_runtime//:hostcontext",
        "@tf_runtime//:support",
        "@tf_runtime//:dtype",
        "@tf_runtime//:tensor_alwayslink",
        "@tf_runtime//backends/common:eigencompat",
    ] + select({
        "//tensorflow:android": [
            "//tensorflow/core:portable_tensorflow_lib_lite",  # TODO(annarev): exclude runtime srcs
        ],
        "//conditions:default": [
            "//tensorflow/core/framework:tensor",
        ],
    }),
)

cc_library(
    name = "kernel_fallback_execute_compat",
    srcs = ["kernel_fallback_execute_compat.cc"],
    hdrs = ["kernel_fallback_execute_compat.h"],
    visibility = [
        "//tensorflow/core/runtime_fallback:internal",
        "//tensorflow/core/tfrt/eager:__pkg__",
        "//tensorflow/core/tfrt/saved_model:__pkg__",
    ],
    deps = [
        ":kernel_fallback_compat_request_state",
        ":kernel_fallback_tensor_conversion_alwayslink",
        ":op_kernel_runner",
        "@llvm-project//llvm:Support",
        "//tensorflow/core/profiler/lib:traceme",
        "//tensorflow/core/runtime_fallback/runtime:kernel_utils",
        "//tensorflow/core/runtime_fallback/runtime:op_logger",
        "//tensorflow/core/runtime_fallback/util:attr_util",
        "//tensorflow/core/tfrt/utils:error_util",
        "//tensorflow/core/tfrt/utils:fallback_tensor",
        "//tensorflow/core/tfrt/utils:tensor_util",
        "//tensorflow/core/tfrt/utils:model_metadata",
        "@tf_runtime//:core_runtime",
        "@tf_runtime//:hostcontext",
        "@tf_runtime//:support",
        "@tf_runtime//:tracing",
        "@tf_runtime//:tensor_alwayslink",
    ] + select({
        "//tensorflow:android": [
            "//tensorflow/core:portable_tensorflow_lib_lite",  # TODO(annarev): exclude runtime srcs
        ],
        "//conditions:default": [
            "//tensorflow/core/framework:tensor",
            "//tensorflow/core:framework",
            "//tensorflow/core:lib",
            "//tensorflow/core/common_runtime/eager:context",
            "//tensorflow/core/common_runtime/eager:core",
        ],
    }),
    alwayslink = True,
)

cc_library(
    name = "op_kernel_runner",
    srcs = ["op_kernel_runner.cc"],
    hdrs = ["op_kernel_runner.h"],
    visibility = [
        "//tensorflow/core/runtime_fallback:internal",
        "//tensorflow/core/tfrt/eager:__pkg__",
    ],
    deps = [
        ":kernel_fallback_compat_request_state",
        ":kernel_fallback_tensor",
        ":kernel_fallback_tensor_conversion_alwayslink",
        "@com_google_absl//absl/container:flat_hash_map",
        "@com_google_absl//absl/container:inlined_vector",
        "@com_google_absl//absl/meta:type_traits",
        "@com_google_absl//absl/types:span",
        "@llvm-project//llvm:Support",
        "//tensorflow/core/runtime_fallback/util:attr_util",
        "//tensorflow/core/tfrt/utils:fallback_tensor",
        "//tensorflow/core/tfrt/utils:statusor",
        "@tf_runtime//:core_runtime",
        "@tf_runtime//:hostcontext",
        "@tf_runtime//:support",
        "@tf_runtime//:tensor_alwayslink",
    ] + select({
        "//tensorflow:android": [
            "//tensorflow/core:portable_tensorflow_lib_lite",  # TODO(annarev): exclude runtime srcs
        ],
        "//conditions:default": [
            "//tensorflow/core/common_runtime/eager:attr_builder",
            "//tensorflow/core/common_runtime/eager:core",
            "//tensorflow/core:framework",
            "//tensorflow/core:framework_lite",
            "//tensorflow/core/framework:node_def_proto_cc",
            "//tensorflow/core/framework:op_def_proto_cc",
            "//tensorflow/core/framework:tensor",
            "//tensorflow/core/platform:errors",
            "//tensorflow/core/platform:status",
            "//tensorflow/core:core_cpu_base",
        ],
    }),
)

cc_library(
    name = "kernel_fallback_compat_request_state",
    srcs = ["kernel_fallback_compat_request_state.cc"],
    hdrs = ["kernel_fallback_compat_request_state.h"],
    visibility = [
        "//tensorflow/core/runtime_fallback:internal",
        # JIT compilation kernels need access to fallback state to set up
        # Eigen thread pool as async runtime worker threads.
        "//tensorflow/compiler/mlir/tfrt:__pkg__",
        # TPU runtime needs to access fallback state to configure MLIR bridge
        # rollout state
        "//third_party/tf_runtime_google:__pkg__",
    ],
    deps = [
        ":kernel_fallback_tensor_conversion_alwayslink",
        "//tensorflow/core/tfrt/utils:fallback_tensor",
        "//tensorflow/core/tfrt/utils:model_metadata",
        "@tf_runtime//:support",
        "@tf_runtime//:tensor_alwayslink",
    ] + select({
        "//tensorflow:android": [
            "//tensorflow/core:portable_tensorflow_lib_lite",  # TODO(annarev): exclude runtime srcs
        ],
        "//conditions:default": [
            "//tensorflow/core/common_runtime:renamed_device",
            "//tensorflow/core/common_runtime/eager:context",
            "//tensorflow/core/common_runtime/eager:core",
            "//tensorflow/core:core_cpu_base",
            "//tensorflow/core:framework",
            "//tensorflow/core:framework_lite",
            "//tensorflow/core:lib",
            "//tensorflow/core/platform:refcount",
            "//tensorflow/core/framework:tensor",
        ],
    }),
)
