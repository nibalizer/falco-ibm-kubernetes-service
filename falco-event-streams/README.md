# Modified from: IBM Event Streams Kafka Python console sample application

This application takes events from falco and sends them out to IBM Event Streams. See [this repo](https://github.com/ibm-messaging/event-streams-samples) for more and better examples of using Event Streams.

This Python console application demonstrates how to connect to [IBM Event Streams](https://console.ng.bluemix.net/docs/services/EventStreams/index.html), send and receive messages using the [confluent-kafka-python](https://github.com/confluentinc/confluent-kafka-python) library. It also shows how to create and list topics using the Event Streams Admin REST API.

__Important Note__: This sample creates a topic with one partition on your behalf. On the Standard plan, this will incur a fee if the topic does not already exist.

## Running the application

The application can be run in the following environments:

* [IBM Cloud Kubernetes Service](./docs/Kubernetes_Service.md) 
* [IBM Cloud Foundry](./docs/Cloud_Foundry.md)
* [Docker Local](./docs/Docker_Local.md)
* [Local Development](./docs/Local.md)
