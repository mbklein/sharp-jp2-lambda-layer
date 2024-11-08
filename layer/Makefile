# Library Versions
WEBP_VERSION=1.4.0
LIBOPENJP2_VERSION=2.5.2
VIPS_VERSION=8.15.3
SHARP_VERSION=0.33.5

PREFIX_PATH=/usr/local
LIB_PATH=$(PREFIX_PATH)/lib
LIBRARY_PATH=$(LIB_PATH)
# EXAMPLE_IMAGE_PATH=$(CURDIR)/example-images

export PKG_CONFIG_PATH=$(LIB_PATH)/pkgconfig:$$PKG_CONFIG_PATH
export LD_LIBRARY_PATH=$(LIB_PATH):$$LD_LIBRARY_PATH

build-SharpJP2Layer: libvips
	mkdir -p "$(ARTIFACTS_DIR)/nodejs"
	mkdir -p "$(ARTIFACTS_DIR)/lib"

	# sharp uses several of the libs we installed or compiled. extract the full list and copy all of those into $(ARTIFACTS_DIR)/lib
	# extract list with ldd from sharp.node, manipulate a bit to only get the absolute paths, then copy

	# install
	cp /tmp/samcli/source/src/package.json "$(ARTIFACTS_DIR)/nodejs/"
	npm install --build-from-source --prefix "$(ARTIFACTS_DIR)/nodejs/" --verbose --foreground-scripts --platform=linux --arch=x64 
	npm prune --prefix "$(ARTIFACTS_DIR)/nodejs/" --omit=dev

	# debug logs
	# ls -la $(ARTIFACTS_DIR)/nodejs/node_modules/sharp/src/build/*/

	# copy the shared objects
	ldd $(ARTIFACTS_DIR)/nodejs/node_modules/sharp/src/build/Release/sharp-linux-x64.node | awk '{ print $$3 }' | grep '/' | xargs -I '{}' cp '{}' $(ARTIFACTS_DIR)/lib

	# debug logs
	# ls -la $(ARTIFACTS_DIR)/lib/

libwebp:
	# libwebp
	curl -L https://github.com/webmproject/libwebp/archive/v$(WEBP_VERSION).tar.gz | tar zx
	cd libwebp-$(WEBP_VERSION) && ./autogen.sh && ./configure --enable-libwebpmux --prefix=$(PREFIX_PATH) && make V=0 && make install

libopenjp2:
	# libopenjp2
	curl -L https://github.com/uclouvain/openjpeg/archive/refs/tags/v$(LIBOPENJP2_VERSION).tar.gz | tar zx
	echo "cd openjpeg-$(LIBOPENJP2_VERSION) && mkdir build && cd build && cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_LIBDIR=$(LIB_PATH) && make && make install"
	cd openjpeg-$(LIBOPENJP2_VERSION) && mkdir build && cd build && cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_LIBDIR=$(LIB_PATH) && make && make install

libvips: installLib
	# libvips
#	curl -L https://github.com/libvips/libvips/releases/download/v$(VIPS_VERSION)/vips-$(VIPS_VERSION).tar.xz | tar xJ
	git clone https://github.com/libvips/libvips.git --branch add-jp2kload-oneshot --single-branch vips
	cd vips && meson setup build --prefix ${PREFIX_PATH} --libdir ${LIB_PATH} -Dopenjpeg=enabled -Dwebp=enabled -Dlcms=enabled --buildtype release && cd build && meson compile && meson install

installLib:
	dnf groupinstall -y "Development Tools" && \
	dnf install -y \
		glibc-langpack-en \
		glib2-devel \
		expat-devel \
		libjpeg-turbo-devel \
		libpng-devel \
		giflib-devel \
		libexif-devel \
		librsvg2-devel \
		libtiff-devel \
		libwebp-devel \
		lcms2-devel \
		openjpeg2-devel \
		gobject-introspection-devel \
		cmake \
		nasm \
		pkg-config \
		meson \
		ninja-build

