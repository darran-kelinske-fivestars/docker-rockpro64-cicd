Note: This was forked from a project that is used to build LineageOS for MicroG.


The actual usage differs and it is intended that you mount the src directory of your RockPRO source to /srv/src and then connect to the terminal of the running instance to perform the necessary build steps.

Big thanks to the original: https://github.com/lineageos4microg/docker-lineage-cicd




## Volumes

You also have to provide Docker some volumes, where it'll store the source, the
resulting builds, the cache and so on. The volumes are:

 * `/srv/src`, for the LineageOS sources
 * `/srv/zips`, for the output builds
 * `/srv/logs`, for the output logs
 * `/srv/ccache`, for the ccache
 * `/srv/local_manifests`, for custom manifests (optional)
 * `/srv/userscripts`, for the user scripts (optional)

When `SIGN_BUILDS` is `true`

 * `/srv/keys`, for the signing keys

When `BUILD_OVERLAY` is `true`

 * `/srv/tmp`, for temporary files

When `LOCAL_MIRROR` is `true`:

 * `/srv/mirror`, for the LineageOS mirror

## Examples

### Build for thea (cm-14.1, officially supported), test keys, no patches

```
docker run \
    -e "BRANCH_NAME=cm-14.1" \
    -e "DEVICE_LIST=thea" \
    -v "/home/user/lineage:/srv/src" \
    -v "/home/user/zips:/srv/zips" \
    -v "/home/user/logs:/srv/logs" \
    -v "/home/user/cache:/srv/ccache" \
    darrank/docker-rockpro64-cicd
```

### Build for dumpling (lineage-15.1, officially supported), custom keys, restricted signature spoofing with integrated microG and FDroid

```
docker run \
    -e "BRANCH_NAME=lineage-15.1" \
    -e "DEVICE_LIST=dumpling" \
    -e "SIGN_BUILDS=true" \
    -e "SIGNATURE_SPOOFING=restricted" \
    -e "CUSTOM_PACKAGES=GmsCore GsfProxy FakeStore MozillaNlpBackend NominatimNlpBackend com.google.android.maps.jar FDroid FDroidPrivilegedExtension " \
    -v "/home/user/lineage:/srv/src" \
    -v "/home/user/zips:/srv/zips" \
    -v "/home/user/logs:/srv/logs" \
    -v "/home/user/cache:/srv/ccache" \
    -v "/home/user/keys:/srv/keys" \
    -v "/home/user/manifests:/srv/local_manifests" \
    darrank/docker-rockpro64-cicd
```

If there are already keys in `/home/user/keys` they will be used, otherwise a
new set will be generated before starting the build (and will be used for every
subsequent build).

The microG and FDroid packages are not present in the LineageOS repositories,
and must be provided through an XML in the `/home/user/manifests`.
[This][prebuiltapks] repo contains some of the most common packages for these
kind of builds: to include it create an XML (the name is irrelevant, as long as
it ends with `.xml`) in the `/home/user/manifests` folder with this content:

```
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <project name="lineageos4microg/android_prebuilts_prebuiltapks" path="prebuilts/prebuiltapks" remote="github" revision="master" />
</manifest>
```

### Build for four devices on cm-14.1 and lineage-15.1 (officially supported), custom keys, restricted signature spoofing with integrated microG and FDroid, custom OTA server

```
docker run \
    -e "BRANCH_NAME=cm-14.1,lineage-15.1" \
    -e "DEVICE_LIST_CM_14_1=onyx,thea" \
    -e "DEVICE_LIST_LINEAGE_15_1=cheeseburger,dumpling" \
    -e "SIGN_BUILDS=true" \
    -e "SIGNATURE_SPOOFING=restricted" \
    -e "CUSTOM_PACKAGES=GmsCore GsfProxy FakeStore MozillaNlpBackend NominatimNlpBackend com.google.android.maps.jar FDroid FDroidPrivilegedExtension " \
    -e "OTA_URL=https://api.myserver.com/" \
    -v "/home/user/lineage:/srv/src" \
    -v "/home/user/zips:/srv/zips" \
    -v "/home/user/logs:/srv/logs" \
    -v "/home/user/cache:/srv/ccache" \
    -v "/home/user/keys:/srv/keys" \
    -v "/home/user/manifests:/srv/local_manifests" \
    darrank/docker-rockpro64-cicd
```

### Build for a6000 (not officially supported), custom keys, restricted signature spoofing with integrated microG and FDroid

As there is no official support for this device, we first have to include the
sources in the source tree through an XML in the `/home/user/manifests` folder;
from [this][a6000-xda] thread we get the links of:

 * Device tree: https://github.com/dev-harsh1998/android_device_lenovo_a6000
 * Common Tree: https://github.com/dev-harsh1998/android_device_lenovo_msm8916-common
 * Kernel: https://github.com/dev-harsh1998/kernel_lenovo_msm8916
 * Vendor blobs: https://github.com/dev-harsh1998/proprietary-vendor_lenovo

Then, with the help of lineage.dependencies from the
[device tree][a6000-device-tree-deps] and the
[common tree][a6000-common-tree-deps] we create an XML
`/home/user/manifests/a6000.xml` with this content:

```
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <project name="dev-harsh1998/android_device_lenovo_a6000" path="device/lenovo/a6000" remote="github" />
  <project name="dev-harsh1998/android_device_lenovo_msm8916-common" path="device/lenovo/msm8916-common" remote="github" />
  <project name="dev-harsh1998/kernel_lenovo_msm8916" path="kernel/lenovo/a6000" remote="github" />
  <project name="dev-harsh1998/proprietary-vendor_lenovo" path="vendor/lenovo" remote="github" />
  <project name="LineageOS/android_device_qcom_common" path="device/qcom/common" remote="github" />
</manifest>
```

We also want to include our custom packages so, like before, create an XML (for
example `/home/user/manifests/custom_packages.xml`) with this content:

```
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <project name="lineageos4microg/android_prebuilts_prebuiltapks" path="prebuilts/prebuiltapks" remote="github" revision="master" />
</manifest>
```

We also set `INCLUDE_PROPRIETARY=false`, as the proprietary blobs are already
provided by the repo
https://github.com/dev-harsh1998/prorietary_vendor_lenovo (so we
don't have to include the TheMuppets repo).

Now we can just run the build like it was officially supported:

```
docker run \
    -e "BRANCH_NAME=lineage-15.1" \
    -e "DEVICE_LIST=a6000" \
    -e "SIGN_BUILDS=true" \
    -e "SIGNATURE_SPOOFING=restricted" \
    -e "CUSTOM_PACKAGES=GmsCore GsfProxy FakeStore MozillaNlpBackend NominatimNlpBackend com.google.android.maps.jar FDroid FDroidPrivilegedExtension " \
    -e "INCLUDE_PROPRIETARY=false" \
    -v "/home/user/lineage:/srv/src" \
    -v "/home/user/zips:/srv/zips" \
    -v "/home/user/logs:/srv/logs" \
    -v "/home/user/cache:/srv/ccache" \
    -v "/home/user/keys:/srv/keys" \
    -v "/home/user/manifests:/srv/local_manifests" \
    darrank/docker-rockpro64-cicd
```


[docker-ubuntu]: https://docs.docker.com/install/linux/docker-ce/ubuntu/
[docker-debian]: https://docs.docker.com/install/linux/docker-ce/debian/
[docker-centos]: https://docs.docker.com/install/linux/docker-ce/centos/
[docker-fedora]: https://docs.docker.com/install/linux/docker-ce/fedora/
[docker-win]: https://docs.docker.com/docker-for-windows/install/
[docker-mac]: https://docs.docker.com/docker-for-mac/install/
[docker-toolbox]: https://docs.docker.com/toolbox/overview/
[docker-helloworld]: https://docs.docker.com/get-started/#test-docker-installation
[los-branches]: https://github.com/LineageOS/android/branches
[signature-spoofing]: https://github.com/microg/android_packages_apps_GmsCore/wiki/Signature-Spoofing
[microg]: https://microg.org/
[signature-spoofing-patches]: src/signature_spoofing_patches/
[blobs-pull]: https://wiki.lineageos.org/devices/bacon/build#extract-proprietary-blobs
[blobs-extract]: https://wiki.lineageos.org/extracting_blobs_from_zips.html
[blobs-themuppets]: https://github.com/TheMuppets/manifests
[lineageota]: https://github.com/julianxhokaxhiu/LineageOTA
[los-extras]: https://download.lineageos.org/extras
[dockerfile]: Dockerfile
[prebuiltapks]: https://github.com/lineageos4microg/android_prebuilts_prebuiltapks
[a6000-xda]: https://forum.xda-developers.com/lenovo-a6000/development/rom-lineageos-15-1-t3733747
[a6000-device-tree-deps]: https://github.com/dev-harsh1998/android_device_lenovo_a6000/blob/lineage-15.1/lineage.dependencies
[a6000-common-tree-deps]: https://github.com/dev-harsh1998/android_device_lenovo_msm8916-common/blob/lineage-15.1/lineage.dependencies

