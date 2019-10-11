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
    ORACLE_BASE=/opt/oracle

# Use second ENV so that variable get substituted
ENV INSTALL_DIR=$ORACLE_BASE/install \
    ORACLE_HOME=$ORACLE_BASE/product/$OBIEE_VERSION \
    DOMAIN_HOME=$ORACLE_BASE/config/domains
    
COPY $RUN_FILE $WAIT_DB_SCRIPT $ORACLE_BASE/

# Use second ENV so that variable get substituted
ENV PATH=$ORACLE_HOME/bin:$PATH

# -------------------------------------------
# Start new stage for installing OBI
# -------------------------------------------

FROM base AS builder

# Copy Install files
COPY $INSTALL_FILE_JDK $INSTALL_FILE_WLS $INSTALL_FILE_BI_1 $INSTALL_FILE_BI_2 $INSTALL_DIR/
COPY $INSTALL_FILE_RSP_WLS $INSTALL_FILE_RSP_BI $INSTALL_FILE_RSP_CONFIG $RUN_FILE _configureOBIEE.sh _dropRCU.sh _validateRCU.sh $WAIT_DB_CLASS $WAIT_DB_SCRIPT $ORACLE_BASE/


# Install JDK
RUN cd $INSTALL_DIR && \
    yum -y install which $INSTALL_FILE_JDK && \
    rm -rf /var/cache/yum

# Replace place holders
# Force /dev/urandom for java
RUN sed -i -e "s|###ORACLE_HOME###|$ORACLE_HOME|g"   $ORACLE_BASE/$INSTALL_FILE_RSP_WLS     && \
    sed -i -e "s|###ORACLE_HOME###|$ORACLE_HOME|g"   $ORACLE_BASE/$INSTALL_FILE_RSP_BI      && \
    sed -i -e "s|###ORACLE_HOME###|$ORACLE_HOME|g"   $ORACLE_BASE/$INSTALL_FILE_RSP_CONFIG  && \
    sed -i -e "s|###DOMAIN_HOME###|$DOMAIN_HOME|g"   $ORACLE_BASE/$INSTALL_FILE_RSP_CONFIG  && \
    sed -i -e "s|###ORACLE_HOME###|$ORACLE_HOME|g"   $ORACLE_BASE/$RUN_FILE                 && \
    sed -i -e "s|###ORACLE_BASE###|$ORACLE_BASE|g"   $ORACLE_BASE/$RUN_FILE                 && \
    sed -i -e "s|source=file:/dev/random|source=file:/dev/urandom|g"     /usr/java/default/jre/lib/security/java.security  && \
    sed -i -e "s|source=file:/dev/urandom|source=file:/dev/./urandom|g"  /usr/java/default/jre/lib/security/java.security

# Set permissions
RUN chmod a+x $INSTALL_DIR/$INSTALL_FILE_BI_1

# Install FMW Infrastructure
USER oracle
RUN touch $ORACLE_BASE/oraInst.loc && \
    echo inventory_loc=$ORACLE_BASE/oraInventory > $ORACLE_BASE/oraInst.loc && \
    echo inst_group= >> $ORACLE_BASE/oraInst.loc && \
    java -jar $INSTALL_DIR/$INSTALL_FILE_WLS -silent -responseFile $ORACLE_BASE/$INSTALL_FILE_RSP_WLS -invPtrLoc $ORACLE_BASE/oraInst.loc

# Install OBI
USER oracle
RUN $INSTALL_DIR/$INSTALL_FILE_BI_1 -silent -responseFile $ORACLE_BASE/$INSTALL_FILE_RSP_BI -invPtrLoc $ORACLE_BASE/oraInst.loc

# -------------------------------------------
# Final stage for copying directories
# -------------------------------------------

FROM base

COPY --chown=root:root --from=builder /usr /usr
COPY --chown=root:root --from=builder /etc /etc
COPY --chown=oracle:oinstall --from=builder $ORACLE_HOME $ORACLE_HOME
#COPY --chown=oracle:oinstall --from=builder $DOMAIN_HOME $DOMAIN_HOME

# CMD exec $ORACLE_BASE/$RUN_FILE
CMD bash
