FROM openjdk:8-jre-alpine

RUN apk add --update --virtual build-dependencies wget git g++ python linux-headers alpine-sdk binutils-gold --no-cache && \

    # Download and build J2V8 4.8.0
    # Source: https://github.com/eclipsesource/J2V8/issues/151
    mkdir /data/ && cd /data/ && git clone https://github.com/eclipsesource/J2V8.git && cd /data/J2V8/ && git checkout 0828fcd5ac2a0f4281e7a5ef913da8cd5a993ac9 && \

    # Set compiler flags
    export CCFLAGS="${CCFLAGS} -fPIC" CXXFLAGS="${CXXFLAGS} -fPIC" CPPFLAGS="${CPPFLAGS} -fPIC" && \

    # Download and build the J2V8 specific node version
    cd /data/J2V8/ && sh ./build-node.sh && \

    cd /data/J2V8/jni && \
    g++ -I ../node -I ../node/deps/v8 -I ../node/deps/v8/include \
        -I ../node/src -I $JAVA_HOME -I $JAVA_HOME/linux  \
        com_eclipsesource_v8_V8Impl.cpp -std=c++11 -fPIC -shared -o /usr/lib/libj2v8_linux_x86_64.so \
        -Wl,--whole-archive ../node/out/Release/obj.target/libnode.a -Wl,--no-whole-archive \
        -Wl,--start-group \
                          ../node/out/Release/obj.target/deps/v8/src/libv8_libbase.a \
                          ../node/out/Release/obj.target/deps/v8/src/libv8_libplatform.a \
                          ../node/out/Release/obj.target/deps/v8/src/libv8_base.a \
                          ../node/out/Release/obj.target/deps/v8/src/libv8_nosnapshot.a \
                          ../node/out/Release/obj.target/deps/v8/src/libv8_libsampler.a \
                          ../node/out/Release/obj.target/deps/uv/libuv.a \
                          ../node/out/Release/obj.target/deps/openssl/libopenssl.a \
                          ../node/out/Release/obj.target/deps/http_parser/libhttp_parser.a \
                          ../node/out/Release/obj.target/deps/gtest/libgtest.a \
                          ../node/out/Release/obj.target/deps/zlib/libzlib.a \
                          ../node/out/Release/obj.target/deps/cares/libcares.a \
        -Wl,--end-group \
        -lrt -z noexecstack -D NODE_COMPATIBLE=1 && \

    # https://github.com/eclipsesource/J2V8/commit/67bfd09f24fece7a0eddcbb0388a572c895c4b8e
    strip --strip-unneeded -R .note -R .comment /usr/lib/libj2v8_linux_x86_64.so && \

    # Cleanup
    apk del build-dependencies && \
    rm -rf /var/cache/apk/* && \
    rm -rf /data/