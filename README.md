# GPU Driver Update (Adreno 819.x) – Integration Guide for Xiaomi Garnet

This document summarizes the complete steps for integrating the **Adreno 819.x GPU driver** into the Xiaomi *garnet* device tree and vendor proprietary tree.

## 1. Prepare the GPU Driver Source

Clone the GPU driver repository:

```bash
git clone https://github.com/zylhdrXP/gpu-driver -b 819
```

The repository provides:

```
vendor/
  lib/
  lib64/
  etc/
  firmware/
```

These blobs will replace or extend the existing GPU drivers in the garnet vendor tree.

## 2. Update the Vendor Proprietary Tree

Navigate to the vendor directory:

```bash
cd vendor/xiaomi/garnet/proprietary
```

Backup old GPU blobs (recommended):

```bash
mv vendor/lib vendor/lib_bak
mv vendor/lib64 vendor/lib64_bak
mv vendor/etc vendor/etc_bak
mv vendor/firmware vendor/firmware_bak

mkdir -p vendor/lib vendor/lib64 vendor/etc vendor/firmware
```

Copy the new GPU driver blobs:

```bash
cp -a ../../../../gpu-driver/vendor/lib/*      vendor/lib/
cp -a ../../../../gpu-driver/vendor/lib64/*    vendor/lib64/
cp -a ../../../../gpu-driver/vendor/etc/*      vendor/etc/
cp -a ../../../../gpu-driver/vendor/firmware/* vendor/firmware/
```

## 3. Update `Android.bp` (Vendor Folder)

Modify GPU-related prebuilt modules to match dependencies required by Adreno 819.x.

### Add to EGL/GLES/Vulkan modules:

```bp
"liblog",
"libnativewindow",
```

### Add to modules using `libadreno_utils` + `libgsl`:

```bp
"libbase",
```

This matches the upstream changes used by crDroid for the 819 driver stack.

## 4. Update `garnet-vendor.mk`

Add new GPU files to `PRODUCT_COPY_FILES` (only if they are not already present):

```makefile
PRODUCT_COPY_FILES += \
    vendor/xiaomi/garnet/proprietary/vendor/lib/libadreno_compiler_cl.so:$(TARGET_COPY_OUT_VENDOR)/lib/libadreno_compiler_cl.so \
    vendor/xiaomi/garnet/proprietary/vendor/lib/libdmabufheap.so:$(TARGET_COPY_OUT_VENDOR)/lib/libdmabufheap.so \
    vendor/xiaomi/garnet/proprietary/vendor/lib/hw/vulkan.adreno.so:$(TARGET_COPY_OUT_VENDOR)/lib/hw/vulkan.adreno.so \
    vendor/xiaomi/garnet/proprietary/vendor/lib64/gpu++.so:$(TARGET_COPY_OUT_VENDOR)/lib64/gpu++.so \
    vendor/xiaomi/garnet/proprietary/vendor/lib64/libdmabufheap.so:$(TARGET_COPY_OUT_VENDOR)/lib64/libdmabufheap.so \
    vendor/xiaomi/garnet/proprietary/vendor/lib64/libgpumemtracer.so:$(TARGET_COPY_OUT_VENDOR)/lib64/libgpumemtracer.so \
    vendor/xiaomi/garnet/proprietary/vendor/lib64/libgpuservice.so:$(TARGET_COPY_OUT_VENDOR)/lib64/libgpuservice.so \
    vendor/xiaomi/garnet/proprietary/vendor/lib64/vendor.qti.hardware.display.mapper@3.0.so:$(TARGET_COPY_OUT_VENDOR)/lib64/vendor.qti.hardware.display.mapper@3.0.so \
    vendor/xiaomi/garnet/proprietary/vendor/lib64/vendor.qti.hardware.display.mapper@4.0.so:$(TARGET_COPY_OUT_VENDOR)/lib64/vendor.qti.hardware.display.mapper@4.0.so \
    vendor/xiaomi/garnet/proprietary/vendor/lib64/vendor.qti.hardware.display.mapperextensions@1.1.so:$(TARGET_COPY_OUT_VENDOR)/lib64/vendor.qti.hardware.display.mapperextensions@1.1.so \
    vendor/xiaomi/garnet/proprietary/vendor/lib64/vendor.qti.hardware.display.mapperextensions@1.2.so:$(TARGET_COPY_OUT_VENDOR)/lib64/vendor.qti.hardware.display.mapperextensions@1.2.so \
    vendor/xiaomi/garnet/proprietary/vendor/lib64/vendor.qti.hardware.display.mapperextensions@1.3.so:$(TARGET_COPY_OUT_VENDOR)/lib64/vendor.qti.hardware.display.mapperextensions@1.3.so \
    vendor/xiaomi/garnet/proprietary/vendor/lib64/egl/libPipeline_plugin.so:$(TARGET_COPY_OUT_VENDOR)/lib64/egl/libPipeline_plugin.so \
    vendor/xiaomi/garnet/proprietary/vendor/etc/permissions/android.hardware.vulkan.version-1_1.xml:$(TARGET_COPY_OUT_VENDOR)/etc/permissions/android.hardware.vulkan.version-1_1.xml \
    vendor/xiaomi/garnet/proprietary/vendor/etc/permissions/android.hardware.vulkan.version-1_3.xml:$(TARGET_COPY_OUT_VENDOR)/etc/permissions/android.hardware.vulkan.version-1_3.xml \
    vendor/xiaomi/garnet/proprietary/vendor/etc/permissions/android.hardware.vulkan.version-1_4.xml:$(TARGET_COPY_OUT_VENDOR)/etc/permissions/android.hardware.vulkan.version-1_4.xml \
    vendor/xiaomi/garnet/proprietary/vendor/etc/permissions/android.software.opengles.deqp.level.xml:$(TARGET_COPY_OUT_VENDOR)/etc/permissions/android.software.opengles.deqp.level.xml \
    vendor/xiaomi/garnet/proprietary/vendor/etc/permissions/android.software.vulkan.deqp.level.xml:$(TARGET_COPY_OUT_VENDOR)/etc/permissions/android.software.vulkan.deqp.level.xml
```

## 5. Allow ELF Files in `PRODUCT_COPY_FILES`

Add the following line to `device/xiaomi/garnet/BoardConfig.mk`:

```makefile
BUILD_BROKEN_ELF_PREBUILT_PRODUCT_COPY_FILES := true
```

This bypasses Soong’s ELF prebuilt restriction for vendor blobs.

## 6. Update `proprietary-files.txt`

To keep extraction consistent, add the new GPU blobs:

```
vendor/lib/libadreno_compiler_cl.so
vendor/lib/libdmabufheap.so
vendor/lib/hw/vulkan.adreno.so
vendor/lib64/gpu++.so
vendor/lib64/libdmabufheap.so
vendor/lib64/libgpumemtracer.so
vendor/lib64/libgpuservice.so
vendor/lib64/vendor.qti.hardware.display.mapper@3.0.so
vendor/lib64/vendor.qti.hardware.display.mapper@4.0.so
vendor/lib64/vendor.qti.hardware.display.mapperextensions@1.1.so
vendor/lib64/vendor.qti.hardware.display.mapperextensions@1.2.so
vendor/lib64/vendor.qti.hardware.display.mapperextensions@1.3.so
vendor/lib64/egl/libPipeline_plugin.so
vendor/etc/permissions/android.hardware.vulkan.version-1_1.xml
vendor/etc/permissions/android.hardware.vulkan.version-1_3.xml
vendor/etc/permissions/android.hardware.vulkan.version-1_4.xml
vendor/etc/permissions/android.software.opengles.deqp.level.xml
vendor/etc/permissions/android.software.vulkan.deqp.level.xml
```

## 7. Build the ROM

```bash
source build/envsetup.sh
lunch <target>
mka bacon -j$(nproc)
```

If runtime issues occur:

```bash
logcat | grep -Ei "EGL|Adreno|Vulkan|hwcomposer"
```

## Status

After completing these steps:

- Adreno 819.x GPU blobs are integrated.
- Soong resolves all dependencies.
- ELF prebuilt checks are allowed.
- Vendor partition includes all GPU components.

## Acknowledgements

This work includes references and assets provided by:

- **Adarsh Grewal** - Source for the vendor tree used as the base reference.
  https://github.com/adarshgrewal
- **My Selly** – Documentation and reference material.
  https://github.com/MySelly
- **zurauciha** – GPU driver source (Adreno 819), originally extracted from a Magisk module.
  https://t.me/zurauciha
