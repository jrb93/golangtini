FROM jrb93/alpinetini:3.6

LABEL org.label-schema.schema-version="1.0" \
      org.label-schema.name="golangtini" \
      org.label-schema.version="1.1.0" \
      org.label-schema.vcs-url="https://github.com/jrb93/golangtini"

ENV GOLANG_VERSION="1.8.5" \
		GOPATH="/go" \
		PATH="$GOPATH/bin:/usr/local/go/bin:$PATH"

# https://golang.org/issue/14851 (Go 1.8 & 1.7)
# https://golang.org/issue/17847 (Go 1.7)
COPY ["*.patch", "go-wrapper", "/tmp/"]

RUN set -eux; \
# Move patch and go-wrapper to correct directory
	mkdir /go-alpine-patches/; \
	mv /tmp/*.patch /go-alpine-patches/; \
	mv /tmp/go-wrapper /usr/local/bin/; \
# Install packages required after build 
	apk add --no-cache ca-certificates git; \
# Install temporary packages for compiling golang from source
	apk add --no-cache --virtual .build-deps \
		bash \
		gcc \
		musl-dev \
		openssl \
		go \
	; \
	export \
# set GOROOT_BOOTSTRAP such that we can actually build Go
		GOROOT_BOOTSTRAP="$(go env GOROOT)" \
# ... and set "cross-building" related vars to the installed system's values so that we create a build targeting the proper arch
# (for example, if our build host is GOARCH=amd64, but our build env/image is GOARCH=386, our build needs GOARCH=386)
		GOOS="$(go env GOOS)" \
		GOARCH="$(go env GOARCH)" \
		GO386="$(go env GO386)" \
		GOARM="$(go env GOARM)" \
		GOHOSTOS="$(go env GOHOSTOS)" \
		GOHOSTARCH="$(go env GOHOSTARCH)" \
	; \
	\
# Download golang, check hash and untar
	wget -O go.tgz "https://golang.org/dl/go$GOLANG_VERSION.src.tar.gz"; \
	echo '4949fd1a5a4954eb54dd208f2f412e720e23f32c91203116bed0387cf5d0ff2d *go.tgz' | sha256sum -c -; \
	tar -C /usr/local -xzf go.tgz; \
	rm go.tgz; \
	\
	cd /usr/local/go/src; \
# Apply patches
	for p in /go-alpine-patches/*.patch; do \
		[ -f "$p" ] || continue; \
		patch -p2 -i "$p"; \
	done; \
# Compile golang
	./make.bash; \
	\
# Finish golang setup
	export PATH="/usr/local/go/bin:$PATH"; \
	mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 777 "$GOPATH"; \
	go version; \
	\
# Cleanup image
	rm -rf /go-alpine-patches; \
	apk del --no-cache .build-deps

WORKDIR $GOPATH