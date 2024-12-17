FROM openjdk:17-jdk-bullseye
ENV DEBIAN_FRONTEND noninteractive

# Install necessary packages
RUN apt-get update -y && apt-get upgrade -y && apt-get install -y git curl jq tidy unzip libxml2-utils

# Install git lfs on Debian bullseye
RUN curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | bash && apt-get install -y git-lfs && git lfs install \
    && rm -rf /var/lib/apt/lists/* \
    && curl -fsSL https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip -o awscliv2.zip \
    && unzip awscliv2.zip \
    && ./aws/install

# Install Jenkins Plugin Installation Manager Tool
RUN curl -fsSL https://github.com/jenkinsci/plugin-installation-manager-tool/releases/download/2.13.2/jenkins-plugin-manager-2.13.2.jar -o /usr/local/bin/jenkins-plugin-cli.jar
RUN echo '#!/bin/sh\njava -jar /usr/local/bin/jenkins-plugin-cli.jar "$@"' > /usr/local/bin/jenkins-plugin-cli
RUN chmod +x /usr/local/bin/jenkins-plugin-cli

ARG user=jenkins
ARG group=jenkins
ARG uid=1000
ARG gid=1000
ARG http_port=8080
ARG agent_port=50000
ARG JENKINS_HOME=/var/jenkins_home
ARG REF=/usr/share/jenkins/ref

ENV JENKINS_HOME $JENKINS_HOME
ENV JENKINS_SLAVE_AGENT_PORT ${agent_port}
ENV REF $REF
ENV KEY ""

RUN mkdir -p $JENKINS_HOME \
  && chown ${uid}:${gid} $JENKINS_HOME \
  && groupadd -g ${gid} ${group} \
  && useradd -d "$JENKINS_HOME" -u ${uid} -g ${gid} -m -s /bin/bash ${user}

VOLUME $JENKINS_HOME

RUN mkdir -p ${REF}/init.groovy.d

ARG TINI_VERSION=v0.16.1
COPY tini_pub.gpg ${JENKINS_HOME}/tini_pub.gpg
RUN curl -fsSL https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-static-$(dpkg --print-architecture) -o /sbin/tini \
  && curl -fsSL https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini-static-$(dpkg --print-architecture).asc -o /sbin/tini.asc \
  && gpg --no-tty --import ${JENKINS_HOME}/tini_pub.gpg \
  && gpg --verify /sbin/tini.asc \
  && rm -rf /sbin/tini.asc /root/.gnupg \
  && chmod +x /sbin/tini

ARG JENKINS_VERSION=2.479.2
ARG JENKINS_SHA=177c2c033f0d3ae4148e601d0fdada60112d83f250521f3a0a0fd97cbb138dbd
ENV JENKINS_VERSION ${JENKINS_VERSION}
ENV JENKINS_SHA ${JENKINS_SHA}

ARG JENKINS_URL=https://get.jenkins.io/war-stable/latest/jenkins.war
RUN curl -fsSL ${JENKINS_URL} -o /usr/share/jenkins/jenkins.war \
  && echo "${JENKINS_SHA}  /usr/share/jenkins/jenkins.war" | sha256sum -c -

RUN mkdir -p ${REF}/cli \
  && curl -fsSL https://platform.boomi.com/atom/atom_install64.sh -o ${REF}/cli/atom_install64.sh \
        && chmod +x "${REF}/cli/atom_install64.sh"

ENV JENKINS_UC https://updates.jenkins.io
ENV JENKINS_UC_EXPERIMENTAL=https://updates.jenkins.io/experimental
ENV JENKINS_INCREMENTALS_REPO_MIRROR=https://repo.jenkins-ci.org/incrementals
RUN chown -R ${user}:${group} "$JENKINS_HOME" "$REF"

EXPOSE ${http_port}
EXPOSE ${agent_port}

ENV COPY_REFERENCE_FILE_LOG $JENKINS_HOME/copy_reference_file.log
ENV UUID=${uuidgen}
USER ${user}

COPY cli $REF/cli
COPY secret $REF/secret
COPY jenkins_jobs/jobs ${REF}/jobs
COPY hudson.tasks.Shell.xml ${JENKINS_HOME}/hudson.tasks.Shell.xml
COPY jenkins-support /usr/local/bin/jenkins-support
COPY jenkins.sh /usr/local/bin/jenkins.sh
COPY tini-shim.sh /bin/tini
COPY jenkins-plugin-cli.sh /bin/jenkins-plugin-cli.sh

COPY entrypoint.sh /usr/local/bin/entrypoint.sh
COPY restart.sh ${REF}/restart/sh
COPY updateexecutors.sh ${REF}/updateexecutors.sh
ARG SONAR_HOME=${REF}/sonar-scanner-4.2.0.1873-linux
RUN mkdir -p ${SONAR_HOME}
COPY sonarqube/* ${REF}/sonar-scanner-4.2.0.1873-linux

ENTRYPOINT ["/bin/bash", "/usr/local/bin/entrypoint.sh"]

ENV JENKINS_USER admin
ENV JENKINS_PASS admin
COPY executors.groovy /usr/share/jenkins/ref/init.groovy.d/
COPY default-user.groovy /usr/share/jenkins/ref/init.groovy.d/
ENV JAVA_OPTS -Djenkins.install.runSetupWizard=false

# Use jenkins-plugin-cli to install plugins
RUN jenkins-plugin-cli --plugins pipeline-model-extensions build-timeout pipeline-milestone-step github-api workflow-job docker-workflow pipeline-stage-view workflow-multibranch apache-httpcomponents-client-4-api pipeline-model-api git-server resource-disposer jquery-detached workflow-scm-step credentials-binding ant momentjs pipeline-rest-api display-url-api gradle junit structs command-launcher trilead-api token-macro github cloudbees-folder ace-editor plain-credentials workflow-aggregator pipeline-model-definition credentials branch-api script-security workflow-step-api timestamper folder-properties git-client pipeline-github-lib mapdb-api workflow-cps pipeline-stage-tags-metadata bouncycastle-api pipeline-build-step antisamy-markup-formatter jsch pam-auth handlebars workflow-api authentication-tokens pipeline-stage-step lockable-resources workflow-cps-global-lib subversion matrix-project git jdk-tool ws-cleanup workflow-support jackson2-api ssh-credentials pipeline-input-step pipeline-graph-analysis durable-task ldap docker-commons ssh-slaves workflow-durable-task-step scm-api matrix-auth workflow-basic-steps authorize-project purge-job-history google-login uno-choice build-publisher htmlpublisher build-user-vars-plugin secure-requester-whitelist postbuildscript
