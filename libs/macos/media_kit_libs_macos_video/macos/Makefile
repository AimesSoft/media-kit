all: Frameworks/*.xcframework Frameworks/.symlinks

MPV_XCFRAMEWORKS_VERSION ?= v0.6.0
MPV_XCFRAMEWORKS_SHA256SUM ?= 20c5add312f4cdb2a9cbf589fcf4495973575db3d90a140747c331690156364e
LIBMPV_DARWIN_XCFRAMEWORKS_ARCHIVE ?=
LIBMPV_DARWIN_XCFRAMEWORKS_DIR ?=

LOCAL_XCFRAMEWORKS_ARCHIVE := $(strip ${LIBMPV_DARWIN_XCFRAMEWORKS_ARCHIVE})
LOCAL_XCFRAMEWORKS_DIR := $(strip ${LIBMPV_DARWIN_XCFRAMEWORKS_DIR})
XCFRAMEWORKS_ARCHIVE_PATH := .cache/xcframeworks/libmpv-xcframeworks-${MPV_XCFRAMEWORKS_VERSION}-macos-universal.tar.gz
XCFRAMEWORKS_ARCHIVE_LINK := .cache/xcframeworks/libmpv-xcframeworks-macos-universal.tar.gz
XCFRAMEWORKS_REMOTE_URL := https://github.com/media-kit/libmpv-darwin-build/releases/download/${MPV_XCFRAMEWORKS_VERSION}/libmpv-xcframeworks_${MPV_XCFRAMEWORKS_VERSION}_macos-universal-video-full.tar.gz

TAR=tar
TAR_WILDCARDS_FLAG := $(shell ${TAR} --version 2>&1 | grep -q 'GNU' && echo "--wildcards" || echo "")

SED=sed
SED_INPLACE_FLAG := $(shell ${SED} --version 2>&1 | grep -q 'GNU' && echo "" || echo "''")

${XCFRAMEWORKS_ARCHIVE_PATH}:
	mkdir -p .cache/xcframeworks
	rm -f .cache/xcframeworks/*.tmp .cache/xcframeworks/*-macos-universal.tar.gz
	if [ -n "${LOCAL_XCFRAMEWORKS_DIR}" ]; then \
		test -d "${LOCAL_XCFRAMEWORKS_DIR}"; \
		find "${LOCAL_XCFRAMEWORKS_DIR}" -maxdepth 1 -type d -name '*.xcframework' | grep -q .; \
		touch ${XCFRAMEWORKS_ARCHIVE_PATH}; \
	elif [ -n "${LOCAL_XCFRAMEWORKS_ARCHIVE}" ]; then \
		test -f "${LOCAL_XCFRAMEWORKS_ARCHIVE}"; \
		cp "${LOCAL_XCFRAMEWORKS_ARCHIVE}" .cache/xcframeworks/libmpv.tar.gz.tmp; \
		mv .cache/xcframeworks/libmpv.tar.gz.tmp ${XCFRAMEWORKS_ARCHIVE_PATH}; \
	else \
		curl -L ${XCFRAMEWORKS_REMOTE_URL} -o .cache/xcframeworks/libmpv.tar.gz.tmp; \
		shasum -a 256 -c <<< '${MPV_XCFRAMEWORKS_SHA256SUM}  .cache/xcframeworks/libmpv.tar.gz.tmp'; \
		mv .cache/xcframeworks/libmpv.tar.gz.tmp ${XCFRAMEWORKS_ARCHIVE_PATH}; \
	fi
	touch ${XCFRAMEWORKS_ARCHIVE_PATH}

${XCFRAMEWORKS_ARCHIVE_LINK}: ${XCFRAMEWORKS_ARCHIVE_PATH}
	rm -f ${XCFRAMEWORKS_ARCHIVE_LINK}
	ln -s libmpv-xcframeworks-${MPV_XCFRAMEWORKS_VERSION}-macos-universal.tar.gz ${XCFRAMEWORKS_ARCHIVE_LINK}

Frameworks/*.xcframework: ${XCFRAMEWORKS_ARCHIVE_LINK}
	mkdir -p Frameworks
	rm -rf Frameworks/*.xcframework
	if [ -n "${LOCAL_XCFRAMEWORKS_DIR}" ]; then \
		cp -R "${LOCAL_XCFRAMEWORKS_DIR}"/*.xcframework Frameworks/; \
	else \
		${TAR} ${TAR_WILDCARDS_FLAG} -xvf ${XCFRAMEWORKS_ARCHIVE_LINK} --strip-components 1 -C Frameworks; \
	fi
	touch Frameworks/*.xcframework

Frameworks/.symlinks: Frameworks/*.xcframework
	rm -rf Frameworks/.symlinks
	mkdir -p Frameworks/.symlinks/mpv
	${SED} -i ${SED_INPLACE_FLAG} 's/\r$$//g' create_framework_symlinks.sh # remove CRLF line terminator added by Flutter packaging (https://github.com/media-kit/media-kit/issues/338)
	sh create_framework_symlinks.sh Frameworks/Mpv.xcframework Frameworks/.symlinks/mpv

.PHONY: clean
clean:
	rm -rf .cache
	rm -f ./Headers/mpv/*.h
	rm -rf ./Frameworks/*.xcframework
