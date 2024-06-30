FROM alpine:3.16 AS builder
LABEL maintainer="tindy.it@gmail.com"
ARG THREADS="4"
ARG SHA=""

# 设置 Git 配置
RUN git config --global url."https://".insteadOf git://

# 安装构建工具和依赖
RUN apk add --no-cache --virtual .build-tools git g++ build-base linux-headers cmake python3 && \
    apk add --no-cache --virtual .build-deps curl-dev rapidjson-dev pcre2-dev yaml-cpp-dev

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

# 构建 subconverter
RUN git clone https://github.com/tindy2013/subconverter --depth=1 && \
    cd subconverter && \
    [ -n "$SHA" ] && sed -i 's/\(v[0-9]\.[0-9]\.[0-9]\)/\1-'"$SHA"'/' src/version.h && \
    python3 -m ensurepip && \
    python3 -m pip install gitpython && \
    python3 scripts/update_rules.py -c scripts/rules_config.conf && \
    cmake -DCMAKE_BUILD_TYPE=Release . && \
    make -j $THREADS

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
