# Dockerfile for custom Tdarr with AMD/Intel/NVIDIA, PgsToSrt, HandBrake, CCExtractor, .NET 9
# Uses official Tdarr image as base, adds custom hardware and tool support

FROM ghcr.io/haveagitgat/tdarr:latest

# Install dependencies and tools
USER root

ENV LIBVA_DRIVER_NAME="radeonsi" \
    LIBVA_DRIVERS_PATH="/usr/lib/x86_64-linux-gnu/dri" \
    LD_LIBRARY_PATH="/usr/lib/x86_64-linux-gnu" \
    NVIDIA_DRIVER_CAPABILITIES="compute,video,utility" \
    NVIDIA_VISIBLE_DEVICES="all" \
    HANDBRAKE="1.10.2" \
    DOTNET_ROLL_FORWARD="Major" \
    HOME="/home/Tdarr"


# Only install missing tools: HandBrakeCLI and dotnet-runtime-9.0 (for PgsToSrt)
RUN set -eux; \
    # Install HandBrakeCLI if not present
    if ! command -v HandBrakeCLI >/dev/null 2>&1; then \
        if uname -m | grep -q x86; then \
            apt-get update && \
            apt-get install -y --no-install-recommends \
                software-properties-common curl git trash-cli unzip wget comskip \
                vainfo mesa-va-drivers libva2 libdrm2 && \
            add-apt-repository -y ppa:kobuk-team/intel-graphics && \
            apt-get update && \
            apt-get install -y handbrake-cli; \
        elif uname -m | grep -q aarch64; then \
            apt-get update && apt-get install -y handbrake-cli; \
        elif uname -m | grep -q armv7l; then \
            apt-get update && apt-get install -y handbrake-cli; \
        fi; \
    fi; \
    # Install dotnet-runtime-9.0 if not present
    if ! command -v dotnet >/dev/null 2>&1; then \
        apt-get update && \
        apt-get install -y --no-install-recommends software-properties-common && \
        add-apt-repository -y ppa:dotnet/backports && \
        apt-get update && \
        apt-get install -y --no-install-recommends dotnet-runtime-9.0; \
    fi; \
    # Install pipx/apprise if not present
    if ! command -v apprise >/dev/null 2>&1; then \
        apt-get update && apt-get install -y pipx && \
        pipx install apprise && \
        ln -sf /root/.local/bin/apprise /usr/local/bin/apprise; \
    fi; \
    # Install PgsToSrt
    tmp=/tmp/pgs && mkdir -p "$tmp" && cd "$tmp" && \
    curl -L -o pgs.zip https://github.com/Tentacule/PgsToSrt/releases/download/v1.4.7/PgsToStr-1.4.7.zip && \
    mkdir -p /opt/pgstosrt && unzip -q pgs.zip -d /opt/pgstosrt && \
    printf '%s\n' '#!/usr/bin/env bash' 'set -euo pipefail' 'exec dotnet /opt/pgstosrt/PgsToSrt.dll "$@" 2> >(grep -v "Detected tesseract language data" >&2)' > /usr/local/bin/pgs2srt && chmod +x /usr/local/bin/pgs2srt && \
    rm -rf "$tmp" && \
    apt-get clean && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/*.deb /root/.cache/*

# Only copy HW transcoding script
COPY docker/root/etc/cont-init.d/01-hw-transcoding /etc/cont-init.d/01-hw-transcoding
RUN chmod +x /etc/cont-init.d/01-hw-transcoding

USER abc

# Entrypoint is inherited from official image
