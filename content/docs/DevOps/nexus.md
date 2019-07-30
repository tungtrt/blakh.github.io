---
title: Nexus Repository
type: docs
---

# Nexus Repository

## Contents

- [Nexus Repository](#nexus-repository)
  - [Contents](#contents)
  - [What is Nexus repository?](#what-is-nexus-repository)
  - [Configuration Nexus Repository on Linux (CentOS or Redhat)](#configuration-nexus-repository-on-linux-centos-or-redhat)
    - [1. Install packages](#1-install-packages)
    - [Change configuration of Nexus](#change-configuration-of-nexus)

## What is Nexus repository?


## Configuration Nexus Repository on Linux (CentOS or Redhat)

### 1. Install packages

* Install `Java 1.8`

```
$ sudo yum install java-1.8.0-openjdk.x86_64
```

* Download `Nexus` package from `SonarType`

```
$ cd /tmp/ && wget https://sonatype-download.global.ssl.fastly.net/repository/repositoryManager/3/nexus-3.14.0-04-unix.tar.gz
```

* Unzip and move to `/opt/nexus`

```
$ sudo tar -xvf nexus-3.14.0-04-unix.tar.gz && mv nexus-3.14.0-04 /opt/nexus
```

* Create Data folder for Nexus

```
$ mkdir /opt/sonatype-work
```

* Create `nexus` user for Nexus

```
$ sudo useradd nexus
$ sudo chown -R nexus:nexus /opt/nexus/ /opt/sonatype-work/
```

* Make Nexus run only with `nexus` user. Modify file `/opt/nexus/bin/nexus.rc` with this content below

```
run_as_user="nexus"
```
* Make `nexus` as a service and set `nexus` start when booting

```
$ sudo ln -s /opt/nexus/bin/nexus /etc/init.d/nexus
$ sudo chkconfig --add nexus
$ sudo chkconfig --levels 345 nexus on
$ sudo service nexus start
```

### Change configuration of Nexus
* Change `Nexus` Java options by edit `/opt/nexus/bin/nexus.vmoptions` file same as contents below:

```
-Xms1200M
-Xmx1200M
-XX:MaxDirectMemorySize=2G
-XX:+UnlockDiagnosticVMOptions
-XX:+UnsyncloadClass
-XX:+LogVMOutput
-XX:LogFile=../sonatype-work/nexus3/log/jvm.log
-XX:-OmitStackTraceInFastThrow
-Djava.net.preferIPv4Stack=true
-Dkaraf.home=.
-Dkaraf.base=.
-Dkaraf.etc=etc/karaf
-Djava.util.logging.config.file=etc/karaf/java.util.logging.properties
-Dkaraf.data=../sonatype-work/nexus3
-Djava.io.tmpdir=../sonatype-work/nexus3/tmp
-Dkaraf.startLocalConsole=false
```

* Change default `nexus` configuration in `/opt/nexus/etc/nexus-default.properties` as below:

```
# Jetty section
application-port=8080
application-host=0.0.0.0
nexus-args=${jetty.etc}/jetty.xml,${jetty.etc}/jetty-http.xml,${jetty.etc}/jetty-requestlog.xml
nexus-context-path=/nexus

# Nexus section
nexus-edition=nexus-pro-edition
nexus-features=\
 nexus-pro-feature
```

* Change limitation file of system by edit `/etc/security/limits.conf` file as below:

```
nexus - nofile 65536
```

* Change `nexus` configuration to use HTTPS protocol. Add contents below to tag `<New id="sslContextFactory" class="org.eclipse.jetty.util.ssl.SslContextFactory">` in `/opt/nexus/etc/jetty/jetty-https.xml` file:

```
    <Set name="ExcludeCipherSuites">
      <Array type="String">
        <Item>SSL_RSA_WITH_DES_CBC_SHA</Item>
        <Item>SSL_DHE_RSA_WITH_DES_CBC_SHA</Item>
        <Item>SSL_DHE_DSS_WITH_DES_CBC_SHA</Item>
        <Item>SSL_RSA_EXPORT_WITH_RC4_40_MD5</Item>
        <Item>SSL_RSA_EXPORT_WITH_DES40_CBC_SHA</Item>
        <Item>SSL_DHE_RSA_EXPORT_WITH_DES40_CBC_SHA</Item>
        <Item>SSL_DHE_DSS_EXPORT_WITH_DES40_CBC_SHA</Item>
      </Array>
    </Set>
    <Set name="ExcludeProtocols">
      <Array type="java.lang.String">
        <Item>SSL</Item>
        <Item>SSLv2</Item>
        <Item>SSLv3</Item>
        <Item>SSLv2Hello</Item>
      </Array>
    </Set>
```

* Restart `nexus` service to effect new configurations:

```
$ service nexus restart
```
