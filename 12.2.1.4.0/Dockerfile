# Install the JDK on oracle linux
FROM oraclelinux:7-slim as jdk

ENV JDK=jdk-8u221-linux-x64.tar.gz \
    JDK_DIR=jdk1.8.0_221

RUN yum -y install wget which unzip.x86_64 file openssl oracle-database-preinstall-18c && \
    wget -q -O $JDK -c --content-disposition \
    "https://javadl.oracle.com/webapps/download/AutoDL?BundleId=239835_230deb18db3e4014bb8e3e8324f81b43" && \
    tar xvzf $JDK

ENV PATH=/$JDK_DIR/bin:$PATH

# just need the java executables, nothing else
FROM jdk as base 
COPY --chown=root:root --from=jdk $JDK_DIR $JDK_DIR

ENV ORACLE_BASE=/opt/oracle \
    ORACLE_HOME=/opt/oracle/product/18c/dbhomeXE \
    ORACLE_SID=XE \
    INSTALL_FILE_DB=oracle-database-xe-18c-1.0-1.x86_64.rpm \
    INSTALL_FILE_WLS=fmw_12.2.1.3.0_infrastructure.jar \
    INSTALL_FILE_BI_1=bi_platform-12.2.1.4.0_linux64.bin \
    INSTALL_FILE_BI_2=bi_platform-12.2.1.4.0_linux64-2.zip \
    OBIEE_VERSION=12.2.1.4.0 \
    INSTALL_FILE_RSP_WLS=weblogic.rsp \
    INSTALL_FILE_RSP_BI=obiee.rsp \
    INSTALL_FILE_RSP_CONFIG=bi_config.rsp \
    WAIT_DB_CLASS=OracleJDBC.class \
    RUN_DB=runOracle.sh \
    PWD_FILE=setPassword.sh \
    CONF_FILE=oracle-xe-18c.conf \
    CHECK_DB_FILE=checkDBStatus.sh \
    INSTALL_DIR=$HOME/install \
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
COPY $PWD_FILE $CHECK_DB_FILE $CONF_FILE $ADD_GCLOUD $INSTALL_FILE_RSP_WLS $INSTALL_FILE_RSP_BI $INSTALL_DIR/
COPY $INSTALL_FILE_RSP_CONFIG $RUN_DB $RUN_OBIEE _configureOBIEE.sh _dropRCU.sh _validateRCU.sh $WAIT_DB_CLASS $WAIT_DB_SCRIPT $ORACLE_BASE/


FROM base as gcloud

# Install gcloud requirements
RUN cd $INSTALL_DIR && \
    chmod u+x $ADD_GCLOUD && \
    ./$ADD_GCLOUD && \
    yum -y install gcc python-devel python-setuptools redhat-rpm-config python-pip google-cloud-sdk && \
    pip install --upgrade pip && \
    pip install --no-cache-dir -U crcmod && \
    rm -rf /var/cache/yum

# New builder stage
FROM gcloud as builder

# Download installation media
RUN gsutil -q cp gs://software.redpillanalytics.io/database/$INSTALL_FILE_DB $INSTALL_DIR
RUN gsutil -q cp gs://software.redpillanalytics.io/fmw/$INSTALL_FILE_WLS $INSTALL_DIR
RUN gsutil -q cp gs://software.redpillanalytics.io/obi/$INSTALL_FILE_BI_1 $INSTALL_DIR
RUN gsutil -q cp gs://software.redpillanalytics.io/obi/$INSTALL_FILE_BI_2 $INSTALL_DIR

# Install DB
RUN chown oracle:oinstall $ORACLE_BASE && \
    cd $INSTALL_DIR && \
    yum -y install $INSTALL_FILE_DB

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
    java -jar $INSTALL_DIR/$INSTALL_FILE_WLS -silent -responseFile $INSTALL_DIR/$INSTALL_FILE_RSP_WLS -invPtrLoc $ORACLE_BASE/oraInst.loc

# Install OBI
RUN $INSTALL_DIR/$INSTALL_FILE_BI_1 -silent -responseFile $INSTALL_DIR/$INSTALL_FILE_RSP_BI -invPtrLoc $ORACLE_BASE/oraInst.loc

CMD bash
