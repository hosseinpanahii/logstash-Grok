## config geo in logstash
    1- Download GeoLite2-City.mmdb from  from the MaxMind site.
    2- edit docker-compose and bind volume to /usr/share/logstash/GeoLite2-City.mmdb
    3- edit pipline.conf like this:

```jsonc
filter {
    grok { 
      match => { "message" => '%{IPORHOST:host_name} - %{IPV4:remote_ip} - %{DATA:user_name} \[%{HTTPDATE:access_time}\] - "%{WORD:http_method} %{DATA:url} HTTP/%{NUMBER:http_version}" %{NUMBER:response_code} %{NUMBER:body_sent_bytes} - "%{DATA:referrer}" "%{DATA:user_agent}" - cache: %{DATA:cache_status} \[%{HTTPDATE:time_local}\] - content_type: %{DATA:content_type}$' } 
      remove_field => "message"
    } 
      geoip {
       database => "/usr/share/logstash/GeoLite2-City.mmdb"
       source => "remote_ip"
       target => "client_geo"
       fields => [ "country_name", "ip" ] 
    }
}
```

    4 docker-compose restart logstash
