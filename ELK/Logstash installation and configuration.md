####Installation
- Logstash and elasticsearch from yum http://www.elastic.co/guide/en/elasticsearch/reference/1.4/setup-repositories.html#_yum
- Oracle JAVA RPMs suck balls. The following for reference:
  - Oracle JRE:`# wget --no-cookies --no-check-certificate --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/7u76-b13/jre-7u76-linux-x64.rpm`
  - Oracle JDK `# wget --no-cookies --no-check-certificate --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/7u76-b13/jdk-7u76-linux-x64.rpm` 
  - List `ld` search path: `# ld --verbose | grep SEARCH_DIR | tr -s ' ;' \\012`  *or* `# ldconfig -v`
- `yum install strace fatrace yum-utils nmap-ncat`

####Configuration
- **Beware** of IPv6 and service scripts (on CentOS6 derivative). As soon as I disabled IPv6 (in sysctl), `rsyslog` feeds `logstash` correctly! [wiki](http://wiki.centos.org/FAQ/CentOS7)
- A one liner, placed in a rsyslog conf file, is enough to start sending everything. `*.* @@localhost:5900` sends to tcp 5900.  
 
####Question
- logstash permissions in init scripts
- logstash data to elastic
