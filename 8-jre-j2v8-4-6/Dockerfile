FROM openjdk:8-jre-alpine

ENV CCFLAGS="${CCFLAGS} -fPIC" CXXFLAGS="${CXXFLAGS} -fPIC" CPPFLAGS="${CPPFLAGS} -fPIC"
RUN apk add --update --virtual build-dependencies wget git g++ python linux-headers alpine-sdk binutils-gold openjdk8-jre="$JAVA_ALPINE_VERSION" --no-cache && \
    mkdir data && cd data && \

    # Download and build the matching nodejs version for J2V8 4.6.0
    # Source: https://github.com/nodejs/node/blob/master/BUILDING.md
    wget https://nodejs.org/download/release/v6.1.0/node-v6.1.0.tar.gz -O - | tar -xz && mv node-v6.1.0 node && \
    cd node && ./configure --enable-static && make -j4 && \

    # Download and build J2V8 4.6.0 (with tests patched for V8 5.x via node 6.1.0)
    # Source: https://github.com/eclipsesource/J2V8/issues/151
    cd /data/ && git clone https://github.com/eclipsesource/J2V8.git && cd /data/J2V8/ && git checkout -b 7dedc23b42465d08cc75b0d32c23eda7e6a6e9d1 && \
    cd /data/J2V8/jni && \
    g++ -I ../../node -I ../../node/deps/v8 -I ../../node/deps/v8/include \
        -I ../../node/src -I /usr/lib/jvm/java-1.8-openjdk -I /usr/lib/jvm/java-1.8-openjdk/linux  \
        com_eclipsesource_v8_V8Impl.cpp -std=c++11 -fPIC -shared -o /usr/lib/libj2v8_linux_x86_64.so \
        -Wl,--whole-archive ../../node/out/Release/libnode.a -Wl,--no-whole-archive \
        -Wl,--start-group \
                          ../../node/out/Release/libv8_libbase.a \
                          ../../node/out/Release/libv8_libplatform.a \
                          ../../node/out/Release/libv8_base.a \
                          ../../node/out/Release/libv8_nosnapshot.a \
                          ../../node/out/Release/libuv.a \
                          ../../node/out/Release/libopenssl.a \
                          ../../node/out/Release/libhttp_parser.a \
                          ../../node/out/Release/libgtest.a \
                          ../../node/out/Release/libzlib.a \
                          ../../node/out/Release/libcares.a \
        -Wl,--end-group \
        -lrt -z noexecstack && \

    # Cleanup
    apk del build-dependencies && \
    rm -rf /var/cache/apk/* && \
    rm -rf /data/