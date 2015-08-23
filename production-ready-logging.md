# Production Ready Logging

During my evaluation of Docker one point was more than obviously: Docker's builtin logging functionalities aren't of any use for production systems. With a little bit more useful base images like [phusion/baseimage](https://github.com/phusion/baseimage-docker) logging everything to stdout/stderr is just a mess, because you cannot determine any more which process printed which messages. 

So here we are back on field 1: which logging systems are available and which of them play nicely with docker even at large scale?

## Requirements

In order of importance:

  1. Stability (don't loose messages except in extremely rare border cases)
  2. Low resources consumption on client machines (log producers)
  3. Ease of use (setup, integration)
  4. standard features like forwarding, processing, filtering
  5. actively developed/maintained
  
## Available Solutions

Just what comes to my mind immediately:

  1. [rsyslog](http://www.rsyslog.com/)
  2. journald
  3. [logstash](https://www.elastic.co/products/logstash)
  4. [fluentd](http://www.fluentd.org/)
  
  
### Logstash

While the logstash server requires a JVM (1.7), [the forwarder](https://github.com/elastic/logstash-forwarder) on the log producing client machines is written in Go and supports secure forwarding (fast, simple, low resources consumption).

### Fluentd

While the fluentd server requires Ruby, [the forwarder](https://github.com/fluent/fluentd-forwarder) on the log producing client machines is written in Go and supports also secure forwarding (fast, simple, low resources consumption).

For quick testing with Docker, see: [fluent-docker-image on github](https://github.com/fluent/fluentd-docker-image)

## Quick Conclusion

After doing some quick evaluations (~ one day) over the mentioned solutions [Logstash](https://www.elastic.co/products/logstash) seems to be the most complete solution (due to **Elastic Search** and **Kibana** - read on). 

When you'll go with Fluentd sooner or later (sooner!) you will hit the point of how you want to evaluate the aggregated logs. If you don't want to re-implement much by yourself or stitch together pieces which won't necessarily fit easily, [ElasticSearch](https://www.elastic.co/products/elasticsearch) and [Kibana](https://www.elastic.co/products/kibana) seems the way to go.

Since Logstash, ElasticSearch and Kibana are all from the same vendor they will fit perfectly together without much troubles in the wiring (out-of-the-box).

Maybe, if done correctly, it may also solve the monitoring, health and metrics part of the system? Setup a simple agent on the machine which periodically generates logs for the required metrics ...

We'll see.
