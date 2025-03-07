---
title: Certificates for device security - Azure IoT Edge | Microsoft Docs 
description: Azure IoT Edge uses certificate to validate devices, modules, and leaf devices and establish secure connections between them. 
author: stevebus

ms.author: stevebus
ms.reviewer: kgremban
ms.date: 10/25/2021
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom:  mqtt
---

# Understand how Azure IoT Edge uses certificates

[!INCLUDE [iot-edge-version-201806-or-202011](../../includes/iot-edge-version-201806-or-202011.md)]

IoT Edge certificates are used by the modules and downstream IoT devices to verify the identity and legitimacy of the [IoT Edge hub](iot-edge-runtime.md#iot-edge-hub) runtime module. These verifications enable a TLS (transport layer security) secure connection between the runtime, the modules, and the IoT devices. Like IoT Hub itself, IoT Edge requires a secure and encrypted connection from IoT downstream (or leaf) devices and IoT Edge modules. To establish a secure TLS connection, the IoT Edge hub module presents a server certificate chain to connecting clients in order for them to verify its identity.

>[!NOTE]
>This article talks about the certificates that are used to secure connections between the different components on an IoT Edge device or between an IoT Edge device and any leaf devices. You may also use certificates to authenticate your IoT Edge device to IoT Hub. Those authentication certificates are different, and are not discussed in this article. For more information about authenticating your device with certificates, see [Create and provision an IoT Edge device using X.509 certificates](how-to-provision-devices-at-scale-linux-x509.md).

This article explains how IoT Edge certificates can work in production, development, and test scenarios.

<!--1.2-->
:::moniker range=">=iotedge-2020-11"

## Changes in version 1.2

* The **device CA certificate** was renamed as **edge CA certificate**.
* The **workload CA certificate** was deprecated. Now the IoT Edge security manager generates the IoT Edge hub server certificate directly from the edge CA certificate, without the intermediate workload CA certificate between them.

:::moniker-end

## IoT Edge certificates

There are two common scenarios for setting up certificates on an IoT Edge device. Sometimes the end user, or operator, of a device purchases a generic device made by a manufacturer then manages the certificates themselves. Other times, the manufacturer works under contract to build a custom device for the operator and does some initial certificate signing before handing off the device. The IoT Edge certificate design attempts to take both scenarios into account.

The following figure illustrates IoT Edge's usage of certificates. There may be zero, one, or many intermediate signing certificates between the root CA certificate and the device CA certificate, depending on the number of entities involved. Here we show one case.

<!--1.1-->
:::moniker range="iotedge-2018-06"
![Diagram of typical certificate relationships](./media/iot-edge-certs/edgeCerts-general.png)

> [!NOTE]
> Currently, a limitation in libiothsm prevents the use of certificates that expire on or after January 1, 2038. This limitation applies to the device CA certificate, any certificates in the trust bundle, and the device ID certificates used for X.509 provisioning methods.

:::moniker-end

<!--1.2-->
:::moniker range=">=iotedge-2020-11"

:::image type="content" source="./media/iot-edge-certs/iot-edge-certs-general-1-2.png" alt-text="Diagram of typical IoT Edge certificate relationships.":::

:::moniker-end

### Certificate authority

The certificate authority, or 'CA' for short, is an entity that issues digital certificates. A certificate authority acts as a trusted third party between the owner and the receiver of the certificate. A digital certificate certifies the ownership of a public key by the receiver of the certificate. The certificate chain of trust works by initially issuing a root certificate, which is the basis for trust in all certificates issued by the authority. Afterwards, the owner can use the root certificate to issue additional intermediate certificates ('leaf' certificates).

### Root CA certificate

A root CA certificate is the root of trust of the entire process. In production scenarios, this CA certificate is usually purchased from a trusted commercial certificate authority like Baltimore, Verisign, or DigiCert. Should you have complete control over the devices connecting to your IoT Edge devices, it's possible to use a corporate level certificate authority. In either event, the entire certificate chain from the IoT Edge hub up rolls to it, so the leaf IoT devices must trust the root certificate. You can store the root CA certificate either in the trusted root certificate authority store, or provide the certificate details in your application code.

### Intermediate certificates

In a typical manufacturing process for creating secure devices, root CA certificates are rarely used directly, primarily because of the risk of leakage or exposure. The root CA certificate creates and digitally signs one or more intermediate CA certificates. There may be only one, or there may be a chain of these intermediate certificates. Scenarios that would require a chain of intermediate certificates include:

* A hierarchy of departments within a manufacturer.

* Multiple companies involved serially in the production of a device.

* A customer buying a root CA and deriving a signing certificate for the manufacturer to sign the devices they make on that customer's behalf.

In any case, the manufacturer uses an intermediate CA certificate at the end of this chain to sign the edge CA certificate placed on the end device. Generally, these intermediate certificates are closely guarded at the manufacturing plant. They undergo strict processes, both physical and electronic for their usage.

<!--1.1-->
:::moniker range="iotedge-2018-06"
### Device CA certificate

The device CA certificate is generated from and signed by the final intermediate CA certificate in the process. This certificate is installed on the IoT Edge device itself, preferably in secure storage such as a hardware security module (HSM). In addition, a device CA certificate uniquely identifies an IoT Edge device. The device CA certificate can sign other certificates.

### IoT Edge Workload CA

The [IoT Edge Security Manager](iot-edge-security-manager.md) generates the workload CA certificate, the first on the "operator" side of the process, when IoT Edge first starts. This certificate is generated from and signed by the device CA certificate. This certificate, which is just another intermediate signing certificate, is used to generate and sign any other certificates used by the IoT Edge runtime. Today, that is primarily the IoT Edge hub server certificate discussed in the following section, but in the future may include other certificates for authenticating IoT Edge components.

The purpose of this "workload" intermediate certificate is to separate concerns between the device manufacturer and the device operator. Imagine a scenario where an IoT Edge device is sold or transferred from one customer to another. You would likely want the device CA certificate provided by the manufacturer to be immutable. However, the "workload" certificates specific to operation of the device should be wiped and recreated for the new deployment.

:::moniker-end

<!--1.2-->
:::moniker range=">=iotedge-2020-11"
### Edge CA certificate

The edge CA certificate is generated from and signed by the final intermediate CA certificate in the process. This certificate is installed on the IoT Edge device itself, preferably in secure storage such as a hardware security module (HSM). In addition, an edge CA certificate uniquely identifies an IoT Edge device. The edge CA certificate can sign other certificates.

:::moniker-end

### IoT Edge hub server certificate

The IoT Edge hub server certificate is the actual certificate presented to leaf devices and modules for identity verification during establishment of the TLS connection required by IoT Edge. This certificate presents the full chain of signing certificates used to generate it up to the root CA certificate, which the leaf IoT device must trust. When generated by IoT Edge, the common name (CN), of this IoT Edge hub certificate is set to the 'hostname' property in the config file after conversion to lower case.

>[!Tip]
>Since the IoT Edge hub server certificate uses the device's hostname property as its common name, no other certificates in the chain should use the same common name.

<!--1.2-->
:::moniker range=">=iotedge-2020-11"
The [IoT Edge Security Manager](iot-edge-security-manager.md) generates the IoT Edge hub certificate, the first on the "operator" side of the process, when IoT Edge first starts. This certificate is generated from and signed by the edge CA certificate.
:::moniker-end

## Production implications

Because manufacturing and operation processes are separated, consider the following implications when preparing production devices:

<!--1.1-->
:::moniker range="iotedge-2018-06"

* With any certificate-based process, the root CA certificate and all intermediate CA certificates should be secured and monitored during the entire process of rolling out an IoT Edge device. The IoT Edge device manufacturer should have strong processes in place for proper storage and usage of their intermediate certificates. In addition, the device CA certificate should be kept in as secure storage as possible on the device itself, preferably a hardware security module.

* The IoT Edge hub server certificate is presented by IoT Edge hub to the connecting client devices and modules. The common name (CN) of the device CA certificate **must not be** the same as the "hostname" that will be used in the config file on the IoT Edge device. The name used by clients to connect to IoT Edge (for example, via the GatewayHostName parameter of the connection string or the CONNECT command in MQTT) **can't be** the same as the common name used in the device CA certificate. This restriction is because the IoT Edge hub presents its entire certificate chain for verification by clients. If the IoT Edge hub server certificate and the device CA certificate both have the same CN, you get in a verification loop and the certificate invalidates.

* Because the device CA certificate is used by the IoT Edge security daemon to generate the final IoT Edge certificates, it must itself be a signing certificate, meaning it has certificate signing capabilities. Applying "V3 Basic constraints CA:True" to the device CA certificate automatically sets up the required key usage properties.
:::moniker-end

<!--1.2-->
:::moniker range=">=iotedge-2020-11"

* With any certificate-based process, the root CA certificate and all intermediate CA certificates should be secured and monitored during the entire process of rolling out an IoT Edge device. The IoT Edge device manufacturer should have strong processes in place for proper storage and usage of their intermediate certificates. In addition, the edge CA certificate should be kept in as secure storage as possible on the device itself, preferably a hardware security module.

* The IoT Edge hub server certificate is presented by IoT Edge hub to the connecting client devices and modules. The common name (CN) of the edge CA certificate **must not be** the same as the "hostname" that will be used in the config file on the IoT Edge device. The name used by clients to connect to IoT Edge (for example, via the GatewayHostName parameter of the connection string or the CONNECT command in MQTT) **can't be** the same as the common name used in the edge CA certificate. This restriction is because the IoT Edge hub presents its entire certificate chain for verification by clients. If the IoT Edge hub server certificate and the edge CA certificate both have the same CN, you get in a verification loop and the certificate invalidates.

* Because the edge CA certificate is used by the IoT Edge security daemon to generate the final IoT Edge certificates, it must itself be a signing certificate, meaning it has certificate signing capabilities. Applying "V3 Basic constraints CA:True" to the edge CA certificate automatically sets up the required key usage properties.
:::moniker-end

## Dev/Test implications

To ease development and test scenarios, Microsoft provides a set of [convenience scripts](https://github.com/Azure/iotedge/tree/master/tools/CACertificates) for generating non-production certificates suitable for IoT Edge in the transparent gateway scenario. For examples of how the scripts work, see [Create demo certificates to test IoT Edge device features](how-to-create-test-certificates.md).

>[!Tip]
> To connect your device IoT "leaf" devices and applications that use our IoT device SDK through IoT Edge, you must add the optional GatewayHostName parameter on to the end of the device's connection string. When the IoT Edge hub server certificate is generated, it is based on a lower-cased version of the hostname from the config file, therefore, for the names to match and the TLS certificate verification to succeed, you should enter the GatewayHostName parameter in lower case.

## Example of IoT Edge certificate hierarchy

To illustrate an example of this certificate path, the following screenshot is from a working IoT Edge device set up as a transparent gateway. OpenSSL is used to connect to the IoT Edge hub, validate, and dump out the certificates.

<!--1.1-->
:::moniker range="iotedge-2018-06"
![Screenshot of the certificate hierarchy at each level](./media/iot-edge-certs/iotedge-cert-chain.png)

You can see the hierarchy of certificate depth represented in the screenshot:

| Certificate type | Certificate name|
|--|--|
| Root CA Certificate | Azure IoT Hub CA Cert Test Only |
| Intermediate CA Certificate | Azure IoT Hub Intermediate Cert Test Only |
| Device CA Certificate | iotgateway.ca ("iotgateway" was passed in as the CA cert name to the convenience scripts) |
| Workload CA Certificate | iotedge workload ca |
| IoT Edge Hub Server Certificate | iotedgegw.local  (matches the 'hostname' from the config file) |
:::moniker-end

<!--1.2-->
:::moniker range=">=iotedge-2020-11"

![Screenshot of the certificate hierarchy at each level](./media/iot-edge-certs/iot-edge-cert-chain-1-2.png)

You can see the hierarchy of certificate depth represented in the screenshot:

| Certificate type | Certificate name |
|--|--|
| Root CA Certificate | Azure IoT Hub CA Cert Test Only |
| Intermediate CA Certificate | Azure IoT Hub Intermediate Cert Test Only |
| Edge CA Certificate | iotgateway.ca ("iotgateway" was passed in as the CA cert name to the convenience scripts) |
| IoT Edge Hub Server Certificate | iotedgegw.local  (matches the 'hostname' from the config file) |
:::moniker-end

## Next steps

[Understand Azure IoT Edge modules](iot-edge-modules.md)

[Configure an IoT Edge device to act as a transparent gateway](how-to-create-transparent-gateway.md)
