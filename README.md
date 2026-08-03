# Automotive Grade Linux (automotive-grade-linux)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
