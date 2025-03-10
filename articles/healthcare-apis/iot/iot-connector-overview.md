---
title: What is the IoT Connector? - Azure Healthcare APIs
description: In this article, you'll learn how to deploy the IoT Connector service in the Azure portal.
services: healthcare-apis
author: stevewohl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: overview
ms.date: 07/12/2021
ms.author: ginle
---

# What is the IoT Connector?

IoT Connector is an optional feature of Azure Healthcare APIs that provides the capability to ingest data from Internet of Medical Things (IoMT) devices.

This article provides an overview of data flow in Azure API for IoT Connector. You'll learn about different data processing stages within the IoT Connector service that transforms device data into FHIR-based [Observation](https://www.hl7.org/fhir/observation.html) resources.

Below are the different stages that data goes through once it is received by Azure API for IoT Connector.

## Ingest

Ingest is the first stage where device data is received into the IoT Connector service. The ingestion endpoint for device data is hosted on an Azure Event Hub. [Azure Event Hub](../../event-hubs/index.yml) platform supports high scale and throughput with ability to receive and process millions of messages per second. It also enables the IoT Connector service to consume messages asynchronously; thus, removing the need for devices to wait while device data gets processed.

> [!NOTE]
> JSON is the only supported format at this time for device data.

## Normalize

Normalize is the next stage where device data is retrieved from the above Azure Event Hub and processed using device mapping templates. This mapping process results in transforming device data into a normalized schema.
The normalization process not only simplifies data processing at later stages but also provides the ability to project one input message into multiple normalized messages. For instance, a device could send multiple vital signs for body temperature, pulse rate, blood pressure, and respiration rate in a single message. This input message would create four separate FHIR resources. Each resource would represent different vital sign, with the input message projected into four different normalized messages.


## Group

Group is the next stage where the normalized messages available from the previous stage are grouped using three different parameters: 

* Device identity
* Measurement type 
* Time period


Device identity and measurement type grouping enable use of [SampledData](https://www.hl7.org/fhir/datatypes.html#SampledData) measurement type. This type provides a concise way to represent a time-based series of measurements from a device in FHIR. And time period controls the latency at which Observation resources generated by the IoT Connector service is written to Azure API for FHIR.

> [!NOTE]
> The time period value is defaulted to 15 minutes and cannot be configured for preview.

## Transform

In the Transform stage, grouped-normalized messages are processed through FHIR mapping templates. Messages matching a template type get transformed into FHIR-based Observation resources as specified through the mapping.

At this point, Device resource, along with its associated Patient resource, is also retrieved from the FHIR server using the device identifier present in the message. These resources are added as a reference to the Observation resource being created.

> [!NOTE]
>All identity look ups are cached once resolved to decrease load on the FHIR server. If you plan on reusing devices with multiple patients, it is advised you create a virtual device resource that is specific to the patient and send virtual device identifier in the message payload. The virtual device can be linked to the actual device resource as a parent.

If no Device resource for a given device identifier exists in the FHIR server, the outcome depends upon the value of Resolution Type set at the time of creation. When set to Look up, the specific message is ignored, and the pipeline will continue to process other incoming messages. If set to Create, the IoT Connector service will create a bare-bones Device and Patient resources on the FHIR server.

## Persist

Once the Observation FHIR resource is generated in the Transform stage, the resource is saved into the FHIR Server. If the FHIR resource is new, it will be created on the server. If the FHIR resource already exists, it will be updated.

## Next steps

For more information about the IoT Connector mapping templates, see

>[!div class="nextstepaction"]
>[How to use the device mapping template](how-to-use-device-mapping-iot.md)

>[!div class="nextstepaction"]
>[How to use the FHIR mapping template](how-to-use-fhir-mapping-iot.md)