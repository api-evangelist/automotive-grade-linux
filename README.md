# Automotive Grade Linux (automotive-grade-linux)

Automotive Grade Linux (AGL) is a collaborative open source project under the Linux Foundation that develops a unified software platform for connected vehicles. AGL brings together automakers (Toyota, Honda, Mercedes-Benz), suppliers, and technology companies to build an open Linux-based software stack for in-vehicle infotainment, instrument clusters, telematics, and software-defined vehicle (SoDeV) architectures.

**URL:** [https://www.automotivelinux.org](https://www.automotivelinux.org)

**Run:** [Capabilities Using Naftiko](https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=company-api-evangelist&utm_content=repo)

## Tags

 - Automotive, Connected Vehicles, Embedded Linux, In-Vehicle Infotainment, IoT, Linux Foundation, Open Source, Software Defined Vehicles

## Timestamps

- **Created:** 2026-03-16
- **Modified:** 2026-04-19

## APIs

### AGL Application Framework API

The AGL Application Framework provides APIs for managing applications on the AGL platform including installation, lifecycle management, permission enforcement, and inter-application communication using D-Bus and WebSocket-based APIs.

**Human URL:** [https://docs.automotivelinux.org/en/master/#0_Getting_Started/2_Developing_an_AGL_Application/](https://docs.automotivelinux.org/en/master/#0_Getting_Started/2_Developing_an_AGL_Application/)

#### Tags

 - Application Framework, Lifecycle Management, IVI, D-Bus

#### Properties

- [Documentation](https://docs.automotivelinux.org/en/master/#0_Getting_Started/2_Developing_an_AGL_Application/)

### AGL VSOMEIP Service API

AGL uses SOME/IP via the vSomeIP library for vehicle service communication, enabling microservice communication between ECUs over Ethernet using service discovery, request-response, and event notification patterns.

**Human URL:** [https://github.com/COVESA/vsomeip](https://github.com/COVESA/vsomeip)

#### Tags

 - SOME/IP, Vehicle Services, ECU Communication, Ethernet, Microservices

#### Properties

- [Documentation](https://github.com/COVESA/vsomeip)
- [GitHub Repository](https://github.com/COVESA/vsomeip)

### AGL SoDeV Software Defined Vehicle API

The AGL SoDeV reference platform provides APIs for software-defined vehicle architectures that decouple software from hardware, building on Zephyr RTOS and meta-AGL layers.

**Human URL:** [https://www.automotivelinux.org](https://www.automotivelinux.org)

#### Tags

 - Software Defined Vehicles, Zephyr, OTA Updates, Vehicle Computing

#### Properties

- [Documentation](https://docs.automotivelinux.org)
- [GitHub Repository](https://github.com/automotive-grade-linux/meta-agl-monorepo)

## Common Properties

- [Website](https://www.automotivelinux.org)
- [Documentation](https://docs.automotivelinux.org)
- [GitHub Organization](https://github.com/automotive-grade-linux)
- [GitHub Repository](https://github.com/automotive-grade-linux/meta-agl-monorepo)

## Features

| Name | Description |
|------|-------------|
| Yocto-Based Build System | AGL uses the Yocto Project and OpenEmbedded build framework with meta-AGL layers for creating customized embedded Linux distributions targeting automotive hardware platforms including Renesas R-Car and Raspberry Pi. |
| SOME/IP Vehicle Services | Service-oriented in-vehicle communication using the SOME/IP protocol via vSomeIP for microservice architectures across ECUs over automotive Ethernet networks. |
| Software Defined Vehicle (SoDeV) | AGL SoDeV reference platform for decoupling software from hardware, enabling flexible, updatable in-vehicle software architectures using Zephyr RTOS and container-based application isolation. |
| Over-The-Air Updates | OTA update framework for delivering software updates to AGL-based in-vehicle systems without physical access, supporting fleet-wide update management. |
| IVI and Instrument Cluster Support | Wayland/Weston-based display framework for in-vehicle infotainment and digital instrument cluster applications with automotive-grade display requirements. |

## Use Cases

| Name | Description |
|------|-------------|
| In-Vehicle Infotainment Development | Develop navigation, media, and connectivity applications for automotive head units using AGL application framework APIs and the Wayland display system. |
| Connected Car Platform | Build telematics, V2X communication, and cloud connectivity capabilities on AGL-based vehicle computing platforms. |
| Software Defined Vehicle Architecture | Design vehicle software architectures that decouple application software from hardware using AGL SoDeV as the foundation platform. |
| Instrument Cluster Applications | Create digital instrument cluster applications for speedometers, tachometers, and ADAS status displays using AGL display APIs. |

## Integrations

| Name | Description |
|------|-------------|
| COVESA Vehicle Signal Specification | Integration with the COVESA Vehicle Signal Specification (VSS) for standardized access to vehicle sensor and actuator data. |
| Eclipse KUKSA | Integration with Eclipse KUKSA for vehicle signal API abstraction enabling portable in-vehicle application development. |
| Zephyr RTOS | AGL SoDeV integrates Zephyr RTOS for safety-critical microcontroller domains within software-defined vehicle architectures. |
| Renesas R-Car Platforms | Primary hardware reference platform support for Renesas R-Car SoCs used in production automotive IVI and cluster systems. |

## Maintainers

**FN:** Kin Lane

**Email:** info@apievangelist.com
