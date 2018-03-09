# Graylog Server.

Graylog version: 2.4

Elasticsearch version: 5.6.8

Create indice for zimbra. In System / Indices. The index prefix must be zimbra as the image show.
This is important for the proper functioning of the streams.
![alt text](https://www.sysadminsdecuba.com/wp-content/uploads/2018/03/Graylog_-_Indices_and_Index_Sets_-_2018-03-07_13.png)

Upload de file in forder Content Pack.

![alt text](https://www.sysadminsdecuba.com/wp-content/uploads/2018/03/Graylog_-_Content_packs_-_2018-03-08_10.32.39.png)

This content pack have grok patterns, beats input for zimbra and stream for zimbra with it's rules

The beats inputs port:5045

![alt text](https://www.sysadminsdecuba.com/wp-content/uploads/2018/03/Graylog_-_Inputs_-_2018-03-05_14.png)

Create indice for zimbra. In System / Indices. The index prefix must be zibra as the image show. 61/5000
This is important for the proper functioning of the streams.
![alt text](https://www.sysadminsdecuba.com/wp-content/uploads/2018/03/Graylog_-_Indices_and_Index_Sets_-_2018-03-07_13.png)

# Zimbra Server

You must install filebeat 5.6.x or any of the versions compatible with the version of elasticsearch 5.6.x. See [Matrix compatibility](https://www.elastic.co/support/matrix#matrix_compatibility)

We will only modify the sessions of Filebeat prospectors and Logstash output.

    #=================== Filebeat prospectors ======================
    filebeat.prospectors:
     
    # Each – is a prospector. Most options can be set at the prospector level, so
    # you can use different prospectors for various configurations.
    # Below are the prospector specific configurations.
 
    – input_type: log
    document_type: postfix
    paths:
    – /var/log/zimbra.log
    – input_type: log
    document_type: zimbra_audit
    paths:
    – /opt/zimbra/log/audit.log
    – input_type: log
    document_type: zimbra_mailbox
    paths:
    – /opt/zimbra/log/zmmailboxd.out
    – input_type: log
    document_type: nginx
    paths:
    – /opt/zimbra/log/nginx.access.log
 
    #———————- Logstash output —————————
    output.logstash:
    # The Logstash hosts
    hosts: ["graylog.dominio.com:5045"]
 
    # Optional SSL. By default is off.
    # List of root certificates for HTTPS server verifications
    bulk_max_size: 2048
    #ssl.certificate_authorities: ["/etc/filebeat/graylog.crt"]
    template.name: "filebeat"
    template.path: "filebeat.template.json"
    template.overwrite: false
    # Certificate for SSL client authentication
    #ssl.certificate: "/etc/pki/client/cert.pem"
 
    # Client Certificate Key
    #ssl.key: "/etc/pki/client/cert.key"

The host in Logstash output section is graylog server.
