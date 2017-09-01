FROM alpine

MAINTAINER Paulo Henrique Maia Soares <paulo.soares@jiva.com.br>

#Define o locale do container
ENV LANG pt_BR.ISO-8859-1

# Expoem as portas do container
EXPOSE 8080

ADD DPM /opt/DPM/

# Adiciona o usuario mgeweb no container
RUN apk add --update sudo && \
    adduser -D mgeweb && \
    chgrp -R mgeweb /usr/local && \
    find /usr/local -type d | xargs chmod g+w && \
    echo "mgeweb ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/mgeweb && \
    chmod 0440 /etc/sudoers.d/mgeweb

# Variaveis que serao utilizadas na inicializacao do container
# As variaveis que estao com a terminacao _DINAMICO% sera alterado pelo DCM (Docker Container Manager)
ENV JAVA_PACKAGE=jdk-7u65-linux-x64.tar.gz \
    APP_DIR=/opt/ \
    JAVA_FOLDER=jdk1.7.0_65 \
    JBOSS_PACKAGE=JBoss-4.0.5_JIVA_016.zip \
    JBOSS_FOLDER=jboss_producao \
    GLIBC=glibc-2.25-r0.apk \
    GLIBC_BIN=glibc-bin-2.25-r0.apk \
    GLIBC_I18N=glibc-i18n-2.25-r0.apk \
    IP_BD_JIVAW=172.16.3.146 \
    PORTA_BD_JIVAW=1521 \
    SID_BD_JIVAW=xe \
    USUARIO_BD_JIVAW=DESJAVA \
    SENHA_BD_JIVAW=TECSIS \
    PACOTE_JIVAW=jiva-w_3.19.11b32.pkg

# Altera o grupo de usuario da pasta ${APP_DIR}
RUN chown -R mgeweb ${APP_DIR}

# Adiciona a variavel no JAVA_HOME
ENV JAVA_HOME ${APP_DIR}${JAVA_FOLDER}
ENV PATH ${PATH}:${JAVA_HOME}/bin

# Instala o curl e o java
RUN echo 'Instalando o JDK' && \ 
    apk --update add curl ca-certificates tar && \
    curl -Ls https://downloads-sankhya-jdk.s3-sa-east-1.amazonaws.com/${JAVA_PACKAGE} > ${APP_DIR}${JAVA_PACKAGE} && \
    tar -xzf ${APP_DIR}${JAVA_PACKAGE} -C ${APP_DIR}&& \
    echo 'Removendo o pacote baixado' ${JAVA_PACKAGE} && \
    rm -rf ${APP_DIR}${JAVA_PACKAGE} && \
    chown -R mgeweb ${APP_DIR}/jdk1.7.0_65 &&\
    echo 'Limpando alguns arquivos do JDK' ${JAVA_HOME} && \
    rm -rf ${JAVA_HOME}/src.zip

# Instala o glibc, habilita o localedef e define o locale para o container
RUN echo 'Instalando o glibc' && \ 
    curl -Ls https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.25-r0/${GLIBC} > /tmp/${GLIBC} && \
    apk add --allow-untrusted /tmp/${GLIBC} && \
    curl -Ls https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.25-r0/${GLIBC_BIN} > /tmp/${GLIBC_BIN} && \
    apk add --allow-untrusted /tmp/${GLIBC_BIN} && \
    curl -Ls https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.25-r0/${GLIBC_I18N} > /tmp/${GLIBC_I18N} && \
    apk add --allow-untrusted /tmp/${GLIBC_I18N} && \
    /usr/glibc-compat/bin/localedef -i pt_BR -f ISO-8859-1 pt_BR.ISO-8859-1 && \
    rm -rf /tmp/*

# Adiciona o JBOSS no container e concede permissao para todas as pastas adicionadas no diretorio opt
RUN echo 'Instalando o JBOSS' && \
    curl -Ls https://downloads-jiva-tools.s3.amazonaws.com/${JBOSS_PACKAGE} > /tmp/${JBOSS_PACKAGE} && \
    unzip /tmp/${JBOSS_PACKAGE} -d ${APP_DIR} && \
    echo 'Removendo diretorios temporarios' && \
    sed -i '/#define the user under which jboss/i JBOSS_HOME=/opt/jboss_producao; export JBOSS_HOME' \
    ${APP_DIR}${JBOSS_FOLDER}/bin/jboss_init_redhat.sh && \
    chmod +x ${APP_DIR}${JBOSS_FOLDER}/bin/*.sh && \
    chown -R mgeweb ${APP_DIR}${JBOSS_FOLDER} && \
    rm -rf /tmp/* \
        ${APP_DIR}${JBOSS_FOLDER}/*.html \
        ${APP_DIR}${JBOSS_FOLDER}/docs

# Altera o usuario do container para mgeweb
USER mgeweb

# Instala o jivaW no container
RUN echo 'Instalando o Jiva-W no container' && \
    mkdir /home/mgeweb/sankhyaw-wpm /home/mgeweb/sankhyaw-wpm/pkgs && \
    cd /opt/DPM/ && \
    sh run.sh && \
    rm -rf /home/mgeweb/sankhyaw-wpm/pkgs/* && \
    mkdir ${APP_DIR}${JBOSS_FOLDER}/server/default/log/ && \
    touch ${APP_DIR}${JBOSS_FOLDER}/server/default/log/server.log
