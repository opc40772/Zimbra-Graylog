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

Edit zimbra the stream and select the index previusly created

![alt text](https://www.sysadminsdecuba.com/wp-content/uploads/2018/03/Graylog_-_Streams_-_2018-03-09_11.28.49.png)

Pipelines

Graylog internally processes and stores the messages in the UTC time zone, it is important to set the correct time zone for each user, but as our goal is to show in a dashboard what parse and store graylog in elasticsearch through grafana I got a problem with the timestamp that creates the same since as it said it stores it in UTC.

We go through a pipeline to establish a field called timestamp_graf to correct this when it is displayed in grafana dashboard.

To create the pipeline go to System/Configuration/Pipelines.

![alt text](https://www.sysadminsdecuba.com/wp-content/uploads/2018/03/Graylog_-_Pipelines_-_2018-03-06_22.png)

We add a new pipeline and put the title and description and save it

![alt text](https://www.sysadminsdecuba.com/wp-content/uploads/2018/03/Graylog_-_New_pipeline_-_2018-03-06_22.png)

As we see the pipeline created has no associated stream so we must establish the stream (s) that it will handle

![alt text](https://www.sysadminsdecuba.com/wp-content/uploads/2018/03/Graylog_-_Pipeline_Pipeline_Timestamp_-_2018-03-07_07_002.png)

In edit connections we associate the stream or streams and in this case we will only associate the zimbra we created and save it.

![alt text](https://www.sysadminsdecuba.com/wp-content/uploads/2018/03/Graylog_-_Pipeline_Pipeline_Timestamp_-_2018-03-07_07.png)

Now we will add the rule that will allow us to do what we want and press the Manage rules button and then Create rule.

![alt text](https://www.sysadminsdecuba.com/wp-content/uploads/2018/03/Graylog_-_Pipeline_rule_timestamp_for_grafana_-_2018-03-07_1.png)

    rule "timestamp_for_grafana"
      when
      has_field("timestamp")
    then
      // the following date format assumes there's no time zone in the string
      let source_timestamp = parse_date(substring(to_string(now("America/Habana")),0,23), "yyyy-MM-dd'T'HH:mm:ss.SSS");
      let dest_timestamp = format_date(source_timestamp,"yyyy-MM-dd HH:mm:ss.SSSZ");
      set_field("timestamp_graf", dest_timestamp);
    end

As you can see, the timezone we use is America / Havana but you can adapt it according to your area of the [list of available time zones.](http://www.joda.org/joda-time/timezones.html). Save it.

![alt text](https://www.sysadminsdecuba.com/wp-content/uploads/2018/03/Graylog_-_Pipeline_rules_-_2018-03-07_11.png)

We go back to Manage pipelines and edit the one we added earlier and edit Stage 0 and in stage rules we select the rule that we add and save

![alt text](https://www.sysadminsdecuba.com/wp-content/uploads/2018/03/Graylog_-_Pipeline_Pipeline_Timestamp_-_2018-03-07_11.png)

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
