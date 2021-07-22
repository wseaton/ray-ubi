ARG PY_MAJOR
ARG PY_MINOR

# this is my mirror of thoth nb image builds, named consistently for easy automation
# will fail for any image with a python version I haven't mirrored
# currently, the only odh notebook python that ray-pipeline works with is py38
# python < 3.7 is known to NOT work due to ray-pipeline dep on SimpleQueue
FROM quay.io/erikerlandson/odh-jupyter:py${PY_MAJOR}${PY_MINOR}

# other python deps
COPY ./requirements.txt /tmp/

USER root
ENV JAVA_VERSION=${JAVA_VERSION:-1.8.0}

# Install java + code-server
RUN yum -y install java-$JAVA_VERSION-openjdk maven

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

### Clear any existing PySpark install that may exist
### Omit if you know the environment does not have PySpark
RUN pip uninstall pyspark &>/dev/null

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


RUN cd $SPARK_HOME/python && python setup.py install

# Emulate an anonymous uid
USER 9999:0


RUN /opt/app-root/bin/pip3 install -r /tmp/requirements.txt
