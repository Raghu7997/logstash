# Logstash filter in ELK:

The following text represents the skeleton of a Logstash configuration pipeline:
```
input {
}
filter {
}
output {
}
```
You need to create the filter part according to your log pattern. Filters are often applied conditionally depending on the characteristics of the event. Logstash provide several filter plugins to create specific, named fields from the logs. 

- To know more about filter plugins in logstash please refer to this page:
https://www.elastic.co/guide/en/logstash/current/filter-plugins.html

### Following is the example of a grok filter:

Sample log : 55.3.244.1 GET /index.html 15824 0.043

Pattern : %{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}

So, the template for the filter would be:  

```
filter {
  grok {
    match => { "message" => "< your pattern >" }
  }
}
```
Logstash ships with about 120 patterns by default. You can find them here: 
https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns.

If you need help building patterns to match your logs, you will find the http://grokdebug.herokuapp.com and http://grokconstructor.appspot.com/ applications quite useful!

### Following is the example of a json filter:

if you have JSON data in the message field:

```
filter {
  json {
    source => "message"
  }
}
```

### Following is the example of a mutate filter:

- The mutate filter allows you to perform general mutations on fields. You can rename, remove, replace, and modify fields in your events.

For example, 

sample log : {"xyz" : { "abc" : "pqr" } }

So, the template for the filter would be:
```
filter {
    json {
        source => "message"
    }
    mutate {
        add_field => { "new_field_name" => "%{[xyz][abc]}" }
    }

    mutate {
        remove => ["xyz"]
    }
}
```
It will add new field "new_field_name" with value "pqr". Also removes the "xyz" field.

Once you created the filter you need to colne the ESS/elastic-logstash-filters repo. Make a new branch from master followed by \<jira-ticket-number\>-\<index-name>-\<message\>.

If the filter is already exist, you need to modified it, commit it and open a PR. In reviwers add "imranm" & "t-davem".

If not then you need to create a new file with "500-output-\<index\>-filter.conf" name.

Follow this template for the new file: 

```
filter {
  if [auto_elasticsearch_index] == "< index_name >" {
    < your filter content >
  }
}
```

Push those filters into the new branch.

Open a PR.

In reviwers add "imranm" & "t-davem".

Check the status of PR, If it fails then click on details which will give you desctiption. update your filter accourdingly and commit again.

After PR will be approved by the team filter will be updated in half an hour into Elasticsearch.
