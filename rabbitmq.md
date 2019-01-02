# RabbitMQ

### Delete multiple Queues

Download Python management plugin 

{% embed url="https://www.rabbitmq.com/management-cli.html" %}

List queues into file:

```text
python rabbitmqadmin --host=localhost --port=8080 --username=guest --password=guest -f tsv -q list queues name > q.txt
```

Delete queues from file:

```text
while read -r name; do python rabbitmqadmin --host=localhost --port=8080 --username=guest --password=guest delete queue name="${name}"; done < q.txt
```

