# If you need these visible after the FROM, be sure to
# add duplicate ARG declarations _after_ the FROM
ARG PY_MAJOR
ARG PY_MINOR
ARG RAY_VERSION
ARG REGISTRY

FROM ${REGISTRY}/ray-ubi:py-${PY_MAJOR}.${PY_MINOR}-ray-${RAY_VERSION}

USER root:0

ENV JAVA_VERSION=${JAVA_VERSION:-1.8.0}
# Install java + code-server
RUN microdnf -y install java-$JAVA_VERSION-openjdk maven

COPY ./requirements.txt /tmp/
ENV HOROVOD_WITH_GLOO=1

### Modify the Hadoop and Spark versions below as needed.
ENV JAVA_HOME=/usr/lib/jvm/jre
ENV HADOOP_VERSION=3.2.1
ENV HADOOP_HOME=/opt/hadoop
ENV HADOOP_CONF_DIR=/opt/hadoop/etc/hadoop
ENV SPARK_VERSION=3.0.1
ENV SPARK_HOME=/opt/spark
ENV PATH="$PATH:$SPARK_HOME/bin:$HADOOP_HOME/bin"

### Enable this for access to ADLS Gen2 when using Hadoop 3.2+
ENV HADOOP_OPTIONAL_TOOLS=hadoop-aws


### Install the desired Hadoop-free Spark distribution
RUN rm -rf ${SPARK_HOME} && \
    wget -q https://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-without-hadoop.tgz && \
    tar -xf spark-${SPARK_VERSION}-bin-without-hadoop.tgz && \
    rm spark-${SPARK_VERSION}-bin-without-hadoop.tgz && \
    mv spark-${SPARK_VERSION}-bin-without-hadoop ${SPARK_HOME} && \
    chmod -R 777 ${SPARK_HOME}/conf

### Install the desired Hadoop libraries
RUN rm -rf ${HADOOP_HOME} && \
    wget -q http://archive.apache.org/dist/hadoop/common/hadoop-${HADOOP_VERSION}/hadoop-${HADOOP_VERSION}.tar.gz && \
    tar -xf hadoop-${HADOOP_VERSION}.tar.gz && \
    rm hadoop-${HADOOP_VERSION}.tar.gz && \
    mv hadoop-${HADOOP_VERSION} ${HADOOP_HOME}

RUN echo "export SPARK_DIST_CLASSPATH=\"\$(hadoop classpath):${HADOOP_HOME}/share/hadoop/tools/lib/*\"" >> ${SPARK_HOME}/conf/spark-env.sh

RUN mkdir -p /opt/jars && \
    wget -P /opt/jars https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk/1.7.4/aws-java-sdk-1.7.4.jar && \
    wget -P /opt/jars https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/3.2.0/hadoop-aws-3.2.0.jar


RUN cd $SPARK_HOME/python && /usr/bin/python3.8 setup.py install


RUN echo \
 && echo "installing ML python libraries" \
 && cd /opt/ray \
 && pipenv install -r /tmp/requirements.txt \
 && chown -R 9998:0 /opt/ray \
 && chmod -R g+rwX /opt/ray \
 && echo \
 && echo "clean up" \
 && rm -rf /tmp/* /root/.cache /root/.local \
 && echo

# emulate anonymous uid
USER 9999:0
