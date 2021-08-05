
# Thoth's extension to OpenShift's S2I build
FROM registry.access.redhat.com/ubi8/s2i-base:latest

EXPOSE 8080

ENV SUMMARY="Thoth's Source-to-Image for Python ${PYTHON_VERSION} applications" \
    DESCRIPTION="Thoth's Source-to-Image for Python ${PYTHON_VERSION} applications. This toolchain is based on Red Hat UBI8. It includes pipenv." \
    PYTHON_VERSION=3.8 \
    PATH=$HOME/.local/bin/:$PATH \
    PYTHONUNBUFFERED=1 \
    PYTHONIOENCODING=UTF-8 \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    PIP_NO_CACHE_DIR=off \
    THOTH_S2I_NAME=quay.io/thoth-station/s2i-thoth-ubi8-py38 \
    THOTH_S2I_VERSION=0.31.1 \
    THAMOS_NO_PROGRESSBAR=1 \
    THAMOS_NO_EMOJI=1 \
    MICROPIPENV_NO_LOCKFILE_PRINT=0 \
    MICROPIPENV_NO_LOCKFILE_WRITE=0

# RHEL7 base images automatically set these envvars to run scl_enable. RHEl8
# images, however, don't as most images don't need SCLs any more. But we want
# to run it even on RHEL8, because we set the virtualenv environment as part of
# that script
ENV BASH_ENV=${APP_ROOT}/etc/scl_enable \
    ENV=${APP_ROOT}/etc/scl_enable \
    PROMPT_COMMAND=". ${APP_ROOT}/etc/scl_enable"

LABEL summary="$SUMMARY" \
    description="$DESCRIPTION" \
    io.k8s.description="$DESCRIPTION" \
    io.k8s.display-name="Thoth Python 3.8-ubi8 S2I" \
    io.openshift.expose-services="8080:http" \
    io.openshift.tags="python,python38" \
    name="$THOTH_S2I_NAME:v$THOTH_S2I_VERSION" \
    vendor="AICoE at the Office of the CTO, Red Hat Inc." \
    authoritative-source-url="https://quay.io/thoth-station/s2i-thoth" \
    version="$THOTH_S2I_VERSION" \
    release="0" \
    ninja.thoth-station.version="0.7.0-dev" \
    maintainer="Thoth Station <aicoe-thoth@redhat.com>"

RUN INSTALL_PKGS="python38 python38-devel python38-setuptools python38-pip nss_wrapper \
        httpd httpd-devel mod_ssl mod_auth_gssapi mod_ldap \
        mod_session atlas-devel gcc-gfortran libffi-devel libtool-ltdl enchant" && \
    yum -y module enable python38:3.8 httpd:2.4 && \
    yum -y --setopt=tsflags=nodocs install $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    # Remove redhat-logos-httpd (httpd dependency) to keep image size smaller.
    rpm -e --nodeps redhat-logos-httpd && \
    yum -y clean all --enablerepo='*' && \
    rm -rf /var/cache/yum

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH.
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

# Copy extra files to the image.
COPY ./root/ /

COPY ./s2i_assemble.patch /tmp/s2i_assemble.patch
COPY ./requirements.txt $HOME/requirements.txt
RUN python3.8 -m venv ${APP_ROOT} && \
    TMPFILE=$(mktemp) && \
    TMPFILE_ASSEMBLE=$(mktemp) && \
    pushd "${STI_SCRIPTS_PATH}" && patch -p 1 </tmp/s2i_assemble.patch && popd && \
    pip3 install -U "pip==20.3.3" && \
    /usr/bin/pip3 install -U "pip==20.3.3" && \
    curl https://raw.githubusercontent.com/thoth-station/micropipenv/master/micropipenv.py | MICROPIPENV_NO_LOCKFILE_WRITE=1 MICROPIPENV_PIP_BIN=/usr/bin/pip3 /usr/bin/python3 - install -- && \
    curl "https://raw.githubusercontent.com/thoth-station/s2i-thoth/master/assemble" -o "${TMPFILE_ASSEMBLE}" && \
    cp "${STI_SCRIPTS_PATH}/assemble" "${TMPFILE}" && \
    head -n1 "${TMPFILE}" >"${STI_SCRIPTS_PATH}/assemble" && \
    cat "${TMPFILE_ASSEMBLE}" >>"${STI_SCRIPTS_PATH}/assemble" && \
    tail -n+2 "${TMPFILE}" >>"${STI_SCRIPTS_PATH}/assemble" && \
    rm "${TMPFILE}" "${TMPFILE_ASSEMBLE}" /tmp/s2i_assemble.patch requirements.txt && \
    sed -i '/  echo "---> Running application from .*/d' "${STI_SCRIPTS_PATH}/run" && \
    chown -R 1001:0 ${APP_ROOT} && \
    fix-permissions ${APP_ROOT} -P && \
    rpm-file-permissions

USER 1001

# Set the default CMD to print the usage of the language image.
CMD $STI_SCRIPTS_PATH/usage
