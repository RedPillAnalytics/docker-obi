# Start with Oracle XE image
FROM redpillanalytics/docker-oxe as base

ENV INSTALL_FILE_JDK="jdk-8u221-linux-x64.rpm" \
    INSTALL_FILE_WLS="fmw_12.2.1.3.0_infrastructure.jar" \
    INSTALL_FILE_BI_1="bi_platform-12.2.1.4.0_linux64.bin" \
    INSTALL_FILE_BI_2="bi_platform-12.2.1.4.0_linux64-2.zip" \
    OBIEE_VERSION="12.2.1.4.0" \
    INSTALL_FILE_RSP_WLS="weblogic.rsp" \
    INSTALL_FILE_RSP_BI="obiee.rsp" \
    INSTALL_FILE_RSP_CONFIG="bi_config.rsp" \
    WAIT_DB_CLASS="OracleJDBC.class" \
    WAIT_DB_SCRIPT="wait-for-database.sh" \
    RUN_FILE="runOBIEE.sh" \
    RUN_DB="runOracle.sh" \
    ORACLE_BASE=/opt/oracle \
    BI_CONFIG_ADMIN_PWD=Admin123 \
    BI_CONFIG_RCU_PWD=Admin123 \
    ORACLE_PWD=Admin123

# Use second ENV so that variable get substituted
ENV INSTALL_DIR=$ORACLE_BASE/install \
    OBI_HOME=$ORACLE_BASE/product/$OBIEE_VERSION \
    ORACLE_HOME=$ORACLE_BASE/product/18c/dbhomeXE \
    DOMAIN_HOME=$ORACLE_BASE/config/domains
    
COPY $RUN_FILE $RUN_OBIEE $WAIT_DB_SCRIPT $ORACLE_BASE/

# Use second ENV so that variable get substituted
ENV PATH=$ORACLE_HOME/bin:$PATH

# -------------------------------------------
# Start new stage for installing OBI
# -------------------------------------------

FROM base AS builder

# Copy Install files
COPY $INSTALL_FILE_JDK $INSTALL_FILE_WLS $INSTALL_FILE_BI_1 $INSTALL_FILE_BI_2 $INSTALL_DIR/
COPY $INSTALL_FILE_RSP_WLS $INSTALL_FILE_RSP_BI $INSTALL_FILE_RSP_CONFIG $RUN_FILE $RUN_DB _configureOBIEE.sh _dropRCU.sh _validateRCU.sh $WAIT_DB_CLASS $WAIT_DB_SCRIPT $ORACLE_BASE/

# Install JDK
RUN cd $INSTALL_DIR && \
    yum -y install which $INSTALL_FILE_JDK && \
    rm -rf /var/cache/yum

# Replace place holders
# Force /dev/urandom for java
RUN sed -i -e "s|###OBI_HOME###|$OBI_HOME|g" $ORACLE_BASE/$INSTALL_FILE_RSP_WLS && \
    sed -i -e "s|###OBI_HOME###|$OBI_HOME|g" $ORACLE_BASE/$INSTALL_FILE_RSP_BI && \
    sed -i -e "s|###OBI_HOME###|$OBI_HOME|g" $ORACLE_BASE/$INSTALL_FILE_RSP_CONFIG  && \
    sed -i -e "s|###DOMAIN_HOME###|$DOMAIN_HOME|g" $ORACLE_BASE/$INSTALL_FILE_RSP_CONFIG  && \
    sed -i -e "s|###OBI_HOME###|$OBI_HOME|g" $ORACLE_BASE/$RUN_FILE && \
    sed -i -e "s|###ORACLE_BASE###|$ORACLE_BASE|g" $ORACLE_BASE/$RUN_FILE && \
    sed -i -e "s|###OBI_HOME###|$OBI_HOME|g" $ORACLE_BASE/$INSTALL_FILE_RSP_CONFIG && \
    sed -i -e "s|###DOMAIN_HOME###|$DOMAIN_HOME|g" $ORACLE_BASE/$INSTALL_FILE_RSP_CONFIG && \
    sed -i -e "s|source=file:/dev/random|source=file:/dev/urandom|g" /usr/java/default/jre/lib/security/java.security && \
    sed -i -e "s|source=file:/dev/urandom|source=file:/dev/./urandom|g" /usr/java/default/jre/lib/security/java.security

# Set permissions
RUN chmod a+x $INSTALL_DIR/$INSTALL_FILE_BI_1 && \
    chmod a+x $ORACLE_BASE/_configureOBIEE.sh && \
    chmod a+x $ORACLE_BASE/_dropRCU.sh && \
    chmod a+x $ORACLE_BASE/_validateRCU.sh && \
    chmod a+x $ORACLE_BASE/$RUN_OBIEE && \
    chmod a+x $ORACLE_BASE/$RUN_FILE

# cat response file
RUN cat $ORACLE_BASE/$INSTALL_FILE_RSP_WLS

# Install FMW Infrastructure
USER oracle
RUN touch $ORACLE_BASE/oraInst.loc && \
    echo inventory_loc=$ORACLE_BASE/oraInventory > $ORACLE_BASE/oraInst.loc && \
    echo inst_group= >> $ORACLE_BASE/oraInst.loc && \
    java -jar $INSTALL_DIR/$INSTALL_FILE_WLS -silent -responseFile $ORACLE_BASE/$INSTALL_FILE_RSP_WLS -invPtrLoc $ORACLE_BASE/oraInst.loc

# Install OBI
RUN $INSTALL_DIR/$INSTALL_FILE_BI_1 -silent -responseFile $ORACLE_BASE/$INSTALL_FILE_RSP_BI -invPtrLoc $ORACLE_BASE/oraInst.loc

# Startup DB
USER root
RUN $ORACLE_BASE/$RUN_DB

# Configure domain
USER oracle
RUN $ORACLE_BASE/$RUN_FILE

# -------------------------------------------
# Final stage for copying directories
# -------------------------------------------

FROM base

COPY --chown=root:root --from=builder /usr /usr
COPY --chown=root:root --from=builder /etc /etc
COPY --chown=oracle:oinstall --from=builder $OBI_HOME $OBI_HOME
COPY --chown=oracle:oinstall --from=builder $DOMAIN_HOME $DOMAIN_HOME

CMD bash
# CMD exec $ORACLE_BASE/$RUN_FILE
