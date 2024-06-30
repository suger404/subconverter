FROM alpine:3.16 AS builder
LABEL maintainer="tindy.it@gmail.com"
ARG THREADS="4"
ARG SHA=""

# 安装所有必要的工具和依赖
RUN apk add --no-cache git g++ build-base linux-headers cmake python3 py3-pip \
    curl-dev rapidjson-dev pcre2-dev yaml-cpp-dev

# 设置 Git 配置
RUN git config --global url."https://".insteadOf git://

# 构建 quickjspp
WORKDIR /tmp
RUN git clone https://github.com/ftk/quickjspp --depth=1 && \
    cd quickjspp && \
    git submodule update --init && \
    cmake -DCMAKE_BUILD_TYPE=Release . && \
    make quickjs -j $THREADS && \
    make install

# 构建 libcron
RUN git clone https://github.com/PerMalmberg/libcron --depth=1 && \
    cd libcron && \
    git submodule update --init && \
    cmake -DCMAKE_BUILD_TYPE=Release . && \
    make libcron -j $THREADS && \
    make install

# 构建 toml11
RUN git clone https://github.com/ToruNiina/toml11 --branch="v3.7.1" --depth=1 && \
    cd toml11 && \
    cmake -DCMAKE_CXX_STANDARD=11 . && \
    make install -j $THREADS

# 克隆 subconverter
WORKDIR /tmp
RUN git clone https://github.com/tindy2013/subconverter --depth=1 || (echo "Failed to clone subconverter" && exit 1)

# 进入 subconverter 目录并更新版本（如果需要）
WORKDIR /tmp/subconverter
RUN if [ -n "$SHA" ]; then \
        sed -i 's/\(v[0-9]\.[0-9]\.[0-9]\)/\1-'"$SHA"'/' src/version.h || (echo "Failed to update version" && exit 1); \
    fi

# 安装 Python 依赖
RUN pip install gitpython || (echo "Failed to install gitpython" && exit 1)

# 更新规则
RUN python3 scripts/update_rules.py -c scripts/rules_config.conf || (echo "Failed to update rules" && exit 1)

# 构建 subconverter
RUN cmake -DCMAKE_BUILD_TYPE=Release . || (echo "CMake configuration failed" && exit 1)
RUN make -j $THREADS || (echo "Make failed" && exit 1)

# 构建最终镜像
FROM alpine:3.16
LABEL maintainer="tindy.it@gmail.com"

RUN apk add --no-cache pcre2 libcurl yaml-cpp

COPY --from=builder /usr/local/lib/ /usr/local/lib/
COPY --from=builder /usr/local/include/ /usr/local/include/
COPY --from=builder /tmp/subconverter/subconverter /usr/bin/
COPY --from=builder /tmp/subconverter/base /base/

WORKDIR /base
CMD ["subconverter"]

EXPOSE 25500/tcp
