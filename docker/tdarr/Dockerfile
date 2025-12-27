ARG BUILD_DATE

FROM rapidmark/tdarr_base:noble_${BUILD_DATE}

ARG VERSION
ARG MODULE

ENV WEB_UI_PORT="8265" SERVER_PORT="8266" NODE_PORT="8267" PUID="1000" PGID="1000" UMASK="002" TZ="Etc/UTC" HOME="/home/Tdarr"
ENV VERSION="${VERSION}"
ENV MODULE="${MODULE}"
ENV MODULE_LINK=""
ENV NODE_LINK=""
ENV BASE_LINK="https://storage.tdarr.io/versions/$VERSION"
ENV inContainer="true"

COPY docker/root/ /
RUN chmod +x /etc/services.d/tdarr_server/run && \
    chmod +x /etc/services.d/tdarr_node/run && \
    chmod +x /etc/cont-init.d/01-setup-perms && \
    chmod +x /etc/cont-init.d/02-hw-transcoding

# Handle Tdarr binaries, pnpm, and buildDate patching
RUN echo $MODULE && echo $VERSION && echo $BUILD_DATE && \
    if [ "$MODULE" = "Tdarr_Node" ]; then \
      echo removing /tdarr_server && \
      rm -rdf /etc/services.d/tdarr_server ; \
    fi && \
    if uname -m | grep -q x86; then \
      MODULE_LINK="${BASE_LINK}/linux_x64/${MODULE}.zip" && \
      NODE_LINK="${BASE_LINK}/linux_x64/Tdarr_Node.zip" ; \
    fi && \
    if uname -m | grep -q aarch64; then \
      MODULE_LINK="${BASE_LINK}/linux_arm64/${MODULE}.zip" && \
      NODE_LINK="${BASE_LINK}/linux_arm64/Tdarr_Node.zip" ; \
    fi && \
    npm install -g pnpm && \
    which pnpm && \
    ln -sf $(which pnpm) /usr/local/bin/pnpm && \
    echo MODULE_LINK=$MODULE_LINK && \
    echo NODE_LINK=$NODE_LINK && \
    curl --connect-timeout 120 --retry 5 -o /tmp/${MODULE}.zip -L "$MODULE_LINK"  && \
    unzip -q /tmp/${MODULE}.zip -d /app/${MODULE} -x *.exe && \
    rm -rdf /app/${MODULE}/${MODULE} && \
    rm -rdf /app/${MODULE}/${MODULE}_Tray && \
    rm -rdf /app/${MODULE}/runtime && \
    cp /usr/bin/ccextractor /app/${MODULE}/assets/app/ccextractor/linux_x64/ccextractor && \
    if [ "$MODULE" = "Tdarr_Server" ]; then \
      curl --connect-timeout 120 --retry 5 -o /tmp/Tdarr_Node.zip -L "$NODE_LINK" && \
      unzip -q /tmp/Tdarr_Node.zip -d /app/Tdarr_Node -x *.exe && \
      rm -rdf /app/Tdarr_Node/Tdarr_Node && \
      rm -rdf /app/Tdarr_Node/Tdarr_Node_Tray && \
      rm -rdf /app/Tdarr_Node/runtime && \
      cp /usr/bin/ccextractor /app/Tdarr_Node/assets/app/ccextractor/linux_x64/ccextractor ; \
    fi && \
    rm -rdf /tmp/${MODULE}.zip && \
    rm -rdf /tmp/Tdarr_Node.zip && \
    \
    # Patch buildDate safely (ONLY the buildDate module; do NOT mass-rewrite JS bundles)
    for f in \
      "/app/${MODULE}/srcug/commonModules/buildDate.js" \
      "/app/Tdarr_Node/srcug/commonModules/buildDate.js" \
      "/app/Tdarr_Server/srcug/commonModules/buildDate.js" \
    ; do \
      if [ -f "$f" ]; then \
        echo "Patching buildDate in $f => ${BUILD_DATE}" && \
        cat > "$f" <<EOF
'use strict';
Object.defineProperty(exports, "__esModule", { value: true });
exports.default = "${BUILD_DATE}";
EOF 
      ; fi; \
    done && \
    \
    # Optional: fail build fast if key bundles are syntactically invalid
    if [ -f "/app/Tdarr_Node/srcug/config/config.js" ]; then node -c "/app/Tdarr_Node/srcug/config/config.js"; fi && \
    if [ -f "/app/Tdarr_Server/srcug/commonModules/configPath.js" ]; then node -c "/app/Tdarr_Server/srcug/commonModules/configPath.js"; fi && \
    \
    chmod -R 755 /app && \
    groupmod -o -g ${PGID} abc && \
    usermod -o -u ${PUID} abc && \
    chown -R abc:abc /app && \
    cp -r /app /app_test && \
    if [ "$MODULE" = "Tdarr_Server" ]; then \
      runTests=true runDockerTests=false node /app_test/Tdarr_Node/srcug/main.js ; \
    fi && \
    rm -rdf /app_test


EXPOSE ${NODE_PORT}
EXPOSE ${WEB_UI_PORT}
EXPOSE ${SERVER_PORT}
ENTRYPOINT ["/init"]
