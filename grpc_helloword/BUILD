load("@rules_proto_grpc//cpp:defs.bzl", "cpp_grpc_library")

proto_library(
    name = "helloworld_proto",
    srcs = ["src/proto/helloworld.proto"],
)

cc_proto_library(
    name = "helloworld_cc_proto",
    deps = [":helloworld_proto"],
)

cpp_grpc_library(
    name = "helloworld_cc_grpc",
    protos = [":helloworld_proto"],
)

cc_binary(
    name = "greeter_client",
    srcs = ["src/greeter_client.cc"],
    deps = [
        "@com_github_grpc_grpc//:grpc++",
        ":helloworld_cc_grpc",
    ],
)

cc_binary(
    name = "greeter_server",
    srcs = ["src/greeter_server.cc"],
    deps = [
        "@com_github_grpc_grpc//:grpc++",
        "@com_github_grpc_grpc//:grpc++_reflection",
        ":helloworld_cc_grpc",
    ],
)