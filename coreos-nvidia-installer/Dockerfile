ARG COREOS_VERSION=1409.5.0
ARG DRIVER_VERSION=381.22

FROM bugroger/coreos-developer:${COREOS_VERSION} as BUILD
LABEL maintainer "Michael Schmidt <michael.j.schmidt@gmail.com>"

ARG COREOS_VERSION
ARG DRIVER_VERSION

RUN emerge-gitclone
RUN . /usr/share/coreos/release && \
  git -C /var/lib/portage/coreos-overlay checkout build-${COREOS_RELEASE_VERSION%%.*}
RUN emerge -gKv coreos-sources
RUN cp /usr/lib64/modules/$(ls /usr/lib64/modules)/build/.config /usr/src/linux/
RUN make -C /usr/src/linux modules_prepare

ENV DRIVER_ARCHIVE=NVIDIA-Linux-x86_64-${DRIVER_VERSION}
ENV SITE=us.download.nvidia.com/XFree86/Linux-x86_64
RUN curl -v -L http://${SITE}/${DRIVER_VERSION}/${DRIVER_ARCHIVE}.run -o ${DRIVER_ARCHIVE}.run
RUN chmod +x ${DRIVER_ARCHIVE}.run && ./${DRIVER_ARCHIVE}.run -x && mv ${DRIVER_ARCHIVE} /build
RUN mkdir /dest && build/nvidia-installer -s -n --kernel-source-path=/usr/src/linux \
  --no-check-for-alternate-installs --no-kernel-module-source --no-opengl-files --no-distro-scripts \
  --kernel-install-path=/dest --log-file-name=${PWD}/nvidia-installer.log || \
  echo "nspawn fails as expected. Because kernel modules can't install in the container"

RUN mkdir -p /opt/nvidia/.work
RUN mkdir -p /opt/nvidia/${DRIVER_VERSION}/${COREOS_VERSION}/bin
RUN mkdir -p /opt/nvidia/${DRIVER_VERSION}/${COREOS_VERSION}/lib64/modules/$(ls /usr/lib64/modules)/kernel/drivers/video/nvidia
RUN find /build        -maxdepth 1 -name "nvidia-*" -executable -exec cp {} /opt/nvidia/${DRIVER_VERSION}/${COREOS_VERSION}/bin \; 
RUN find /build        -maxdepth 1 -name "*.so.*"               -exec cp {} /opt/nvidia/${DRIVER_VERSION}/${COREOS_VERSION}/lib64 \; 
RUN find /build/kernel -maxdepth 1 -name "*.ko"                 -exec cp {} /opt/nvidia/${DRIVER_VERSION}/${COREOS_VERSION}/lib64/modules/$(ls /usr/lib64/modules)/kernel/drivers/video/ \; 

FROM alpine 
LABEL maintainer "Michael Schmidt <michael.j.schmidt@gmail.com>"

ARG COREOS_VERSION
ARG DRIVER_VERSION

COPY --from=BUILD /opt/nvidia /opt/nvidia
COPY run.sh /
COPY install.sh /

ENV COREOS_VERSION $COREOS_VERSION
ENV DRIVER_VERSION $DRIVER_VERSION

ENTRYPOINT ["/run.sh"]
