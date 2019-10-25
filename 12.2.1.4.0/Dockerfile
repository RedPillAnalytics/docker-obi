FROM gcr.io/rpa-devops/oxe

WORKDIR $INSTALL_DIR

ENV JDK=jdk-8u231-linux-x64.rpm \
    JDK_DIR=/usr/java/jdk1.8.0_231-amd64 \
    INSTALL_FILE_WLS=fmw_12.2.1.3.0_infrastructure.jar \
    INSTALL_FILE_BI_1=bi_platform-12.2.1.4.0_linux64.bin \
    INSTALL_FILE_BI_2=bi_platform-12.2.1.4.0_linux64-2.zip \
    OBIEE_VERSION=12.2.1.4.0 \
    INSTALL_FILE_RSP_WLS=weblogic.rsp \
    INSTALL_FILE_RSP_BI=obiee.rsp \
    CONFIG_FILE_BI_RSP=bi_config.rsp \
    WAIT_DB_CLASS=OracleJDBC.class \
    WAIT_DB_SCRIPT=wait-for-database.sh \
    RUN_OBIEE=runOBIEE.sh

# Separate ENV for substituion
ENV OBI_HOME=$ORACLE_BASE/product/$OBIEE_VERSION \
    DOMAIN_HOME=$ORACLE_BASE/config/domains \
    PATH=$DOMAIN_HOME/bi/bitools/bin:$PATH

# # Copy scripts and configs
# # -------------
COPY $RUN_OBIEE \
     _configureOBIEE.sh \
     _dropRCU.sh \
     _validateRCU.sh \
     $WAIT_DB_CLASS \
     $WAIT_DB_SCRIPT \
     $ORACLE_BASE/

# Copy respons files
COPY $INSTALL_FILE_RSP_WLS $INSTALL_FILE_RSP_BI $INSTALL_DIR/
COPY $CONFIG_FILE_BI_RSP $ORACLE_BASE/

# Download installation media
RUN curl -o $INSTALL_DIR/$JDK https://s3.amazonaws.com/software.redpillanalytics.io/oracle/java/$JDK \
    && curl -o $INSTALL_DIR/$INSTALL_FILE_WLS https://s3.amazonaws.com/software.redpillanalytics.io/oracle/obi/12.2.1.4.0/$INSTALL_FILE_WLS \
    && curl -o $INSTALL_DIR/$INSTALL_FILE_BI_1 https://s3.amazonaws.com/software.redpillanalytics.io/oracle/obi/12.2.1.4.0/$INSTALL_FILE_BI_1 \
    && curl -o $INSTALL_DIR/$INSTALL_FILE_BI_2 https://s3.amazonaws.com/software.redpillanalytics.io/oracle/obi/12.2.1.4.0/$INSTALL_FILE_BI_2

# Install JDK
RUN yum -y localinstall $INSTALL_DIR/$JDK \
    && yum -y install which \
    && rm -rf /var/cache/yum \
    && rm -f $INSTALL_DIR/$JDK

# Replace place holders
# Force /dev/urandom for java
RUN sed -i -e "s|###OBI_HOME###|$OBI_HOME|g" $INSTALL_DIR/$INSTALL_FILE_RSP_WLS \
    && sed -i -e "s|###OBI_HOME###|$OBI_HOME|g" $INSTALL_DIR/$INSTALL_FILE_RSP_BI \
    && sed -i -e "s|###OBI_HOME###|$OBI_HOME|g" $ORACLE_BASE/$CONFIG_FILE_BI_RSP  \
    && sed -i -e "s|###DOMAIN_HOME###|$DOMAIN_HOME|g" $ORACLE_BASE/$CONFIG_FILE_BI_RSP \
    && sed -i -e "s|###OBI_HOME###|$OBI_HOME|g" $ORACLE_BASE/$RUN_OBIEE \
    && sed -i -e "s|###ORACLE_BASE###|$ORACLE_BASE|g" $ORACLE_BASE/$RUN_OBIEE \
    && sed -i -e "s|###OBI_HOME###|$OBI_HOME|g" $ORACLE_BASE/$CONFIG_FILE_BI_RSP \
    && sed -i -e "s|###DOMAIN_HOME###|$DOMAIN_HOME|g" $ORACLE_BASE/$CONFIG_FILE_BI_RSP \
    && sed -i -e "s|source=file:/dev/random|source=file:/dev/urandom|g" $JDK_DIR/jre/lib/security/java.security \
    && sed -i -e "s|source=file:/dev/urandom|source=file:/dev/./urandom|g" $JDK_DIR/jre/lib/security/java.security

# Set permissions
RUN chmod a+x $INSTALL_DIR/$INSTALL_FILE_BI_1 \
    && chmod a+x $ORACLE_BASE/_configureOBIEE.sh \
    && chmod a+x $ORACLE_BASE/_dropRCU.sh \
    && chmod a+x $ORACLE_BASE/_validateRCU.sh \
    && chmod a+x $ORACLE_BASE/$RUN_OBIEE

# Install FMW Infrastructure
USER oracle
RUN touch $ORACLE_BASE/oraInst.loc \
    && echo inventory_loc=$ORACLE_BASE/oraInventory > $ORACLE_BASE/oraInst.loc \
    && echo inst_group= >> $ORACLE_BASE/oraInst.loc \
    && java -jar $INSTALL_DIR/$INSTALL_FILE_WLS -silent -ignoreSysPrereqs -novalidation -responseFile $INSTALL_DIR/$INSTALL_FILE_RSP_WLS -invPtrLoc $ORACLE_BASE/oraInst.loc

# Install OBI
RUN $INSTALL_DIR/$INSTALL_FILE_BI_1 -silent -ignoreSysPrereqs -novalidation -responseFile $INSTALL_DIR/$INSTALL_FILE_RSP_BI -invPtrLoc $ORACLE_BASE/oraInst.loc \
    && rm -f $INSTALL_DIR/$INSTALL_FILE_BI_1

USER root
RUN rm -f $INSTALL_DIR/$INSTALL_FILE_WLS \
    && rm -f $INSTALL_DIR/$INSTALL_FILE_BI_2

# CMD exec $ORACLE_BASE/$RUN_DB
# CMD exec $ORACLE_BASE/$RUN_FILE
WORKDIR /home/oracle
CMD bash
