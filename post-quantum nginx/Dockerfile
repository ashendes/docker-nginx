# Base image
FROM nginx:latest

# Define arguments
ARG OPENSSL_INSTALL_DIR=/usr/local/ssl
ARG OPENSSL_VERSION=3.2.1
ARG LIBOQS_VERSION=0.9.2
ARG OQS_PROVIDER_VERSION=0.5.2

# Install build dependencies for OpenSSL
RUN apt-get update && apt-get install -y \
    wget \
    build-essential \
    checkinstall \
    zlib1g-dev

# Download and unpack OpenSSL
WORKDIR /usr/src
RUN wget https://www.openssl.org/source/openssl-${OPENSSL_VERSION}.tar.gz \
    && tar -xf openssl-${OPENSSL_VERSION}.tar.gz 

# Configure, compile, and install OpenSSL
RUN cd /usr/src/openssl-${OPENSSL_VERSION} \
    && ./config --prefix=${OPENSSL_INSTALL_DIR} --openssldir=${OPENSSL_INSTALL_DIR} shared zlib \
    && make \
    && make test \
    && make install

# Configure dynamic linker run-time bindings
RUN echo "${OPENSSL_INSTALL_DIR}/lib" > /etc/ld.so.conf.d/openssl-${OPENSSL_VERSION}.conf \
    && ldconfig -v

# Update PATH to include the new OpenSSL binaries and set the LD_LIBRARY_PATH
ENV PATH="${OPENSSL_INSTALL_DIR}/bin:${PATH}"
ENV LD_LIBRARY_PATH="${OPENSSL_INSTALL_DIR}/lib"

# Download and unpack OQS Provider
WORKDIR /usr/src
RUN wget https://github.com/open-quantum-safe/oqs-provider/archive/refs/tags/${OQS_PROVIDER_VERSION}.tar.gz \
    && tar -xf ${OQS_PROVIDER_VERSION}.tar.gz 

# Download and unpack LibOQS
RUN cd oqs-provider-${OQS_PROVIDER_VERSION} \
    && wget https://github.com/open-quantum-safe/liboqs/archive/refs/tags/${LIBOQS_VERSION}.tar.gz \
    && tar -xf ${LIBOQS_VERSION}.tar.gz \
    && mv liboqs-${LIBOQS_VERSION} liboqs

# Install build dependencies for LibOQS
RUN apt-get update && apt-get install -y \
    cmake \
    gcc \
    ninja-build

# Install LibOQS and OQS Provider
RUN cd oqs-provider-${OQS_PROVIDER_VERSION} \
    && OPENSSL_INSTALL=${OPENSSL_INSTALL_DIR} ./scripts/fullbuild.sh \
    && cmake --install _build \
    && cp _build/lib/oqsprovider.so ${OPENSSL_INSTALL_DIR}/lib/ossl-modules/

# Configure OpenSSL to use OQSProvider
RUN cd ${OPENSSL_INSTALL_DIR} \ 
    && echo "[provider_sect]" >> openssl.cnf \
    && echo "oqsprovider = oqsprovider_sect" >> openssl.cnf \
    && echo "[oqsprovider_sect]" >> openssl.cnf \
    && echo "activate = 1" >> openssl.cnf \
    && echo "[default_sect]" >> openssl.cnf \
    && echo "activate = 1" >> openssl.cnf \
    && echo "[openssl_init]" >> openssl.cnf \
    && echo "ssl_conf = ssl_sect" >> openssl.cnf \
    && echo "[ssl_sect]" >> openssl.cnf \
    && echo "system_default = system_default_sect" >> openssl.cnf \
    && echo "[system_default_sect]" >> openssl.cnf \
    && echo "Groups = x25519_kyber768:x25519" >> openssl.cnf

# Cleanup build dependencies and directories
RUN apt-get remove --purge -y build-essential checkinstall zlib1g-dev wget cmake gcc ninja-build \
    && apt-get autoremove -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /usr/src/openssl-* /usr/src/oqs-provider-* \
    && rm /usr/src/*.tar.gz  
