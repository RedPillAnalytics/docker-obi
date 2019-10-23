# Install the JDK on oracle linux
FROM oraclelinux:7-slim as slim

ENV JDK=jdk-8u221-linux-x64.tar.gz \
    JDK_DIR=/jdk1.8.0_221

ENV PATH=$JDK_DIR/bin:$PATH

RUN yum -y install wget tar gzip

FROM slim as jdk

RUN wget -q -O $JDK -c --content-disposition \
    "https://javadl.oracle.com/webapps/download/AutoDL?BundleId=239835_230deb18db3e4014bb8e3e8324f81b43" && \
    tar xvzf $JDK

# just need the java executables, nothing else
FROM slim as base 
COPY --chown=root:root --from=jdk $JDK_DIR $JDK_DIR

ENV ORACLE_BASE=/opt/oracle \
    ORACLE_HOME=/opt/oracle/product/18c/dbhomeXE \
    ORACLE_SID=XE \
    ORACLE_PWD=Admin123 \
    INSTALL_FILE_DB=oracle-database-xe-18c-1.0-1.x86_64.rpm \
    INSTALL_FILE_WLS=fmw_12.2.1.3.0_infrastructure.jar \
    INSTALL_FILE_BI_1=bi_platform-12.2.1.4.0_linux64.bin \
    INSTALL_FILE_BI_2=bi_platform-12.2.1.4.0_linux64-2.zip \
    OBIEE_VERSION=12.2.1.4.0 \
    INSTALL_DIR=$HOME/install \
    INSTALL_FILE_RSP_WLS=weblogic.rsp \
    INSTALL_FILE_RSP_BI=obiee.rsp \
    INSTALL_FILE_RSP_CONFIG=bi_config.rsp \
    WAIT_DB_CLASS=OracleJDBC.class \
    RUN_DB=runOracle.sh \
    PWD_FILE=setPassword.sh \
    CONF_FILE=oracle-xe-18c.conf \
    CHECK_DB_FILE=checkDBStatus.sh \
    ORACLE_DOCKER_INSTALL=true \
    NLS_LANG=AMERICAN_AMERICA.AL32UTF8 \
    CUSTOM_SQL=custom.zip \
    ADD_GCLOUD=add-gcloud-repo.sh \
    WAIT_DB_SCRIPT=wait-for-database.sh \
    RUN_OBIEE=runOBIEE.sh

# Separate ENV for substituion
ENV OBI_HOME=$ORACLE_BASE/product/$OBIEE_VERSION \
    DOMAIN_HOME=$ORACLE_BASE/config/domains \
    PATH=$ORACLE_HOME/bin:$PATH

# Copy scripts and configs
# -------------
COPY $INSTALL_FILE_RSP_WLS $INSTALL_FILE_RSP_BI $INSTALL_DIR/
COPY $CONF_FILE $CHECK_DB_FILE $PWD_FILE $INSTALL_FILE_RSP_CONFIG $RUN_DB $RUN_OBIEE _configureOBIEE.sh _dropRCU.sh _validateRCU.sh $WAIT_DB_CLASS $WAIT_DB_SCRIPT $ORACLE_BASE/

RUN cd $INSTALL_DIR && \
    yum -y install which unzip.x86_64 file openssl oracle-database-preinstall-18c

# FROM base as gcloud

# # Install gcloud requirements
# RUN cd $INSTALL_DIR && \
#     chmod u+x $ADD_GCLOUD && \
#     ./$ADD_GCLOUD && \
#     yum -y install gcc python-devel python-setuptools redhat-rpm-config python-pip google-cloud-sdk && \
#     pip install --upgrade pip && \
#     pip install --no-cache-dir -U crcmod && \
#     rm -rf /var/cache/yum

# # New builder stage
# FROM gcloud as builder

# # Download installation media
# RUN gsutil -q cp gs://software.redpillanalytics.io/database/$INSTALL_FILE_DB $INSTALL_DIR
# RUN gsutil -q cp gs://software.redpillanalytics.io/fmw/$INSTALL_FILE_WLS $INSTALL_DIR
# RUN gsutil -q cp gs://software.redpillanalytics.io/obi/$INSTALL_FILE_BI_1 $INSTALL_DIR
# RUN gsutil -q cp gs://software.redpillanalytics.io/obi/$INSTALL_FILE_BI_2 $INSTALL_DIR

FROM base AS builder

# Copy installation media
COPY $INSTALL_FILE_DB $INSTALL_FILE_WLS $INSTALL_FILE_BI_1 $INSTALL_FILE_BI_2 $INSTALL_DIR

# Install DB
RUN chown oracle:oinstall $ORACLE_BASE && \
    cd $INSTALL_DIR && \
    yum -y install $INSTALL_FILE_DB && \
    rm -rf /var/cache/yum && \
    rm -f $INSTALL_FILE_1 && \
    rm -rf $ORACLE_HOME/demo && \
    rm -rf $ORACLE_HOME/dmu && \
    rm -rf $ORACLE_HOME/javavm && \
    rm -rf $ORACLE_HOME/md && \
    rm -rf $ORACLE_HOME/nls/demo && \
    rm -rf $ORACLE_HOME/odbc && \
    rm -rf $ORACLE_HOME/rdbms/demo && \
    rm -rf $ORACLE_HOME/R && \
    rm -rf $ORACLE_HOME/instantclient && \
    rm -rf $ORACLE_HOME/inventory && \
    rm -rf $ORACLE_HOME/deinstall && \
    rm -r  $ORACLE_HOME/lib/ra_aix_ppc64.zip && \
    rm -r  $ORACLE_HOME/lib/ra_hpux_ia64.zip && \
    rm -r  $ORACLE_HOME/lib/ra_solaris*.zip && \
    rm -f  $ORACLE_HOME/lib/ra_windows64.zip && \
    rm -f  $ORACLE_HOME/lib/ra_zlinux64.zip && \
    rm -rf $ORACLE_HOME/crs && \
    rm -f  $ORACLE_HOME/bin/asmcmd && \
    rm -f  $ORACLE_HOME/bin/asmcmdcore && \
    rm -f  $ORACLE_HOME/bin/bdschecksw && \
    rm -f  £ORACLE_HOME/bin/dbv && \
    rm -f  $ORACLE_HOME/bin/ldap* && \
    rm -f  $ORACLE_HOME/bin/dbfs_client && \
    rm -f  $ORACLE_HOME/bin/afdboot && \
    rm -f  $ORACLE_HOME/bin/exp && \
    rm -f  $ORACLE_HOME/bin/imp && \
    rm -f  $ORACLE_HOME/bin/*.exe && \
    rm -f  $ORACLE_HOME/bin/lcsscan && \
    rm -f  $ORACLE_HOME/bin/dgmgrl && \
    rm -f  $ORACLE_HOME/bin/nid && \
    rm -f  $ORACLE_HOME/bin/orion && \
    rm -f  $ORACLE_HOME/bin/procob && \
    rm -f  $ORACLE_HOME/bin/setasmgid && \
    rm -f  $ORACLE_HOME/bin/wrap && \
    rm -f  $ORACLE_HOME/bin/*0 && \
    rm -f  $ORACLE_HOME/bin/tnsping && \
    rm -f  $ORACLE_HOME/bin/tkprof && \
    rm -f  $ORACLE_HOME/bin/srvctl && \
    rm -f  $ORCALE_HOME/bin/wrc && \
    rm -rf $ORACLE_HOME/sdk && \
    strip --remove-section=.comment $ORACLE_HOME/bin/oracle && \
    strip --remove-section=.comment $ORACLE_HOME/bin/rman && \
    strip --remove-section=.comment $ORACLE_HOME/bin/tnslsnr

# Replace place holders
# Force /dev/urandom for java
RUN sed -i -e "s|###OBI_HOME###|$OBI_HOME|g" $INSTALL_DIR/$INSTALL_FILE_RSP_WLS && \
    sed -i -e "s|###OBI_HOME###|$OBI_HOME|g" $INSTALL_DIR/$INSTALL_FILE_RSP_BI && \
    sed -i -e "s|###OBI_HOME###|$OBI_HOME|g" $ORACLE_BASE/$INSTALL_FILE_RSP_CONFIG  && \
    sed -i -e "s|###DOMAIN_HOME###|$DOMAIN_HOME|g" $ORACLE_BASE/$INSTALL_FILE_RSP_CONFIG  && \
    sed -i -e "s|###OBI_HOME###|$OBI_HOME|g" $ORACLE_BASE/$RUN_OBIEE && \
    sed -i -e "s|###ORACLE_BASE###|$ORACLE_BASE|g" $ORACLE_BASE/$RUN_OBIEE && \
    sed -i -e "s|###OBI_HOME###|$OBI_HOME|g" $ORACLE_BASE/$INSTALL_FILE_RSP_CONFIG && \
    sed -i -e "s|###DOMAIN_HOME###|$DOMAIN_HOME|g" $ORACLE_BASE/$INSTALL_FILE_RSP_CONFIG && \
    sed -i -e "s|source=file:/dev/random|source=file:/dev/urandom|g" $JDK_DIR/jre/lib/security/java.security && \
    sed -i -e "s|source=file:/dev/urandom|source=file:/dev/./urandom|g" $JDK_DIR/jre/lib/security/java.security

# Set permissions
RUN chmod a+x $INSTALL_DIR/$INSTALL_FILE_BI_1 && \
    chmod a+x $ORACLE_BASE/_configureOBIEE.sh && \
    chmod a+x $ORACLE_BASE/_dropRCU.sh && \
    chmod a+x $ORACLE_BASE/_validateRCU.sh && \
    chmod a+x $ORACLE_BASE/$RUN_OBIEE && \
    chmod a+x $ORACLE_BASE/$RUN_DB

# Install FMW Infrastructure
USER oracle
RUN touch $ORACLE_BASE/oraInst.loc && \
    echo inventory_loc=$ORACLE_BASE/oraInventory > $ORACLE_BASE/oraInst.loc && \
    echo inst_group= >> $ORACLE_BASE/oraInst.loc && \
    java -jar $INSTALL_DIR/$INSTALL_FILE_WLS -silent -ignoreSysPrereqs -novalidation -responseFile $INSTALL_DIR/$INSTALL_FILE_RSP_WLS -invPtrLoc $ORACLE_BASE/oraInst.loc

# Install OBI
RUN $INSTALL_DIR/$INSTALL_FILE_BI_1 -silent -ignoreSysPrereqs -novalidation -responseFile $INSTALL_DIR/$INSTALL_FILE_RSP_BI -invPtrLoc $ORACLE_BASE/oraInst.loc

FROM base as runtime

COPY --chown=root:root --from=builder /etc /etc
COPY --chown=oracle:oinstall --from=builder $ORACLE_BASE $ORACLE_BASE

CMD bash
