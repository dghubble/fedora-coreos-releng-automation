FROM registry.fedoraproject.org/fedora:31

# set PYTHONUNBUFFERED env var to non-empty string so that our
# output immediately comes to the console
ENV PYTHONUNBUFFERED=true

# Get any latest updates since last container spin
RUN dnf update -y

# Install needed rpms
# Grab ostree from koji for now for https://github.com/ostreedev/ostree/pull/1984
RUN dnf -y install \
        fedora-messaging \
        python-requests  \
        strace           \
        https://kojipkgs.fedoraproject.org/packages/ostree/2020.3/3.fc31/x86_64/ostree-2020.3-3.fc31.x86_64.rpm \
        https://kojipkgs.fedoraproject.org/packages/ostree/2020.3/3.fc31/x86_64/ostree-libs-2020.3-3.fc31.x86_64.rpm

# Configure a umask of 0002 which will allow for the group permissions
# to include write for newly created files. We need this because we'd
# like to access the OSTree repos from two different Kubernetes pods,
# which will have different UIDs but the same GID.
# See: https://pagure.io/releng/issue/8811#comment-616490
#
# This is also done within the coreos-ostree-importer-wrapper script,
# but we also want it done in the container because admins may run some
# ostree operations by hand from a bash prompt and we want those
# operations to not put the repo in a bad state.
RUN echo 'umask 0002' > /etc/profile.d/umask-for-openshift.sh

# Put the file into a location that can be imported
ADD coreos_ostree_importer.py /usr/lib/python3.7/site-packages/

# Copy in the fedora messaging config into the
# default location
ADD fedora-messaging-config.toml /etc/fedora-messaging/config.toml

# Copy in the wrapper that starts the importer
ADD coreos-ostree-importer-wrapper /usr/local/bin/

# Call the wrapper
CMD ["/usr/local/bin/coreos-ostree-importer-wrapper"]
