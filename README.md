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

- once you created the filter you need to colne the ESS/elastic-logstash-filters repo. Make a new branch from master followed by "<index-branch_name>".

- If the filter is already exist, you need to modified it, commit it and open a PR.

- If not then you need to create a new file with "500-output-<index>-filter.conf" name. Commit the changes and open a PR.

Follow this template for the new file: 

```
filter {
  if [auto_elasticsearch_index] == "< index_name >" {
    < your filter content >
  }
}
```

make a new branch followed by,

push those filters into the new branch, 
open a PR
Just check the status of PR, If it fails check your filter and commit again
The PR will be approved by the team and the filter will be updated in half an hour into Elasticsearch.



# Watcher Alerts in DES ELK:

- To know more about creating watchers (alerts) in elasticsearch please refer to this page: https://wiki.autodesk.com/pages/viewpage.action?spaceKey=DES&title=Watcher+alerts+for+Elasticsearch

To place the watcher into Elasticsearch, the user needs to clone the repo, and commit the alert body as shown in the template into the develop branch and create a pull request.

The PR will be approved by the team and the job will post the watcher into Elasticsearch.

Following is the template for the alert:
```
{
 "done": "false",
 "immediate_action": "none",   # Supports: none|execute|ack|activate|deactivate
 "watch" : {
   "id": "<index_name>-xyz",
   "watch" : {
     <your watch definition here>    <--- Your watcher body goes here.
   }
 }
}
```
#### Attributes:

- **done** : Set to false if you want to take an action or update the alert.
- **immediate_action** : Set to the action that is to be taken after updating the alert.
   
   > Supported actions are:
   > - none: https://www.elastic.co/guide/en/elasticsearch/reference/current/watcher-api-put-watch.html
   > - execute: https://www.elastic.co/guide/en/elasticsearch/reference/current/watcher-api-execute-watch.html
   > - ack: https://www.elastic.co/guide/en/elasticsearch/reference/current/watcher-api-ack-watch.html
   > - activate: https://www.elastic.co/guide/en/elasticsearch/reference/current/watcher-api-activate-watch.html
   > - deactivate: https://www.elastic.co/guide/en/elasticsearch/reference/current/watcher-api-deactivate-watch.html

`Note`: `Actions execute/ack/actiavte/deactivate still post the alert. So any action taken will update the watcher first and then perform the action.`

- **watch.id** : ID must begin with a letter or underscore and contain only letters, underscores, dashes, and numbers.

`Note`: `The id should begin with index name, <index_name>-id. The filename should be same as id.`

- **watch.watch** : The watch body as described [here](https://wiki.autodesk.com/pages/viewpage.action?spaceKey=DES&title=Watcher+alerts+for+Elasticsearch)
