FROM mcr.microsoft.com/devcontainers/base:ubuntu26.04

ARG USERNAME=vscode
ENV PATH=/usr/local/bin:${PATH}

USER root

RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends \
      build-essential gcc cmake meson ninja-build pkg-config \
      autoconf automake libtool m4 \
      zlib1g-dev libbz2-dev libedit-dev libreadline-dev \
      llvm clang clang-tools clang-format-22 gdb lldb valgrind lcov \
      ruby ruby-dev hugo \
      python3 python3-venv perl \
      ca-certificates curl xz-utils \
      language-pack-fr jq ripgrep shellcheck \
      aptitude cmake-curses-gui git-lfs \
    && gem install asciidoctor asciidoctor-html5s rouge \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

RUN arch="$(dpkg --print-architecture)" \
    && case "${arch}" in \
      amd64) zig_arch=x86_64 ;; \
      arm64) zig_arch=aarch64 ;; \
      *) echo "Unsupported target architecture: ${arch}" >&2; exit 1 ;; \
    esac \
    && zig_index="$(curl --fail --location --silent --show-error https://ziglang.org/download/index.json)" \
    && zig_download="$(printf '%s' "${zig_index}" | jq -ec --arg arch "${zig_arch}-linux" \
      'del(.master) | to_entries | max_by(.key | split(".") | map(tonumber)) | .value[$arch]')" \
    && zig_url="$(printf '%s' "${zig_download}" | jq -er '.tarball')" \
    && zig_sha256="$(printf '%s' "${zig_download}" | jq -er '.shasum')" \
    && curl --fail --location --silent --show-error \
      --output /tmp/zig.tar.xz \
      "${zig_url}" \
    && echo "${zig_sha256}  /tmp/zig.tar.xz" | sha256sum --check --status \
    && mkdir /opt/zig \
    && tar --extract --xz --file /tmp/zig.tar.xz --directory /opt/zig --strip-components=1 \
    && ln --symbolic /opt/zig/zig /usr/local/bin/zig \
    && rm /tmp/zig.tar.xz \
    && curl --fail --location --silent --show-error \
      --output /usr/local/bin/bazelisk \
      "https://github.com/bazelbuild/bazelisk/releases/latest/download/bazelisk-linux-${arch}" \
    && curl --fail --location --silent --show-error \
      --output /usr/local/bin/buildifier \
      "https://github.com/bazel-contrib/buildtools/releases/latest/download/buildifier-linux-${arch}" \
    && chmod 0755 /usr/local/bin/bazelisk /usr/local/bin/buildifier \
    && ln --symbolic /usr/local/bin/bazelisk /usr/local/bin/bazel

USER ${USERNAME}