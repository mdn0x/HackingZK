[Operational Technology (OT)](Operational%20Technology%20(OT).md) refers to systems used to monitor and control industrial operations. Industrial [Control Systems (ICS)](Control%20Systems%20(ICS).md) includes systems used to monitor and control industrial processes.

If IT operators have dozens of years of experience securing networks, systems and applications, OT operators are pretty new to this topic. Moreover in OT/ICS development availability is always preferred over integrity and confidentiality. In other words ICS softwares are designed to be fast but, often, insecure.

Supervisory Control and Data Acquisition (SCADA) systems are used to control and automate industrial processes. SCADA systems includes:

- Supervisory computers: the servers used to manage the process gathering data on the process and communicating with filed devices (PLC/RTU). In smaller deployments HMI is embedded in a single computer, in larger deploy HMI is installed into a dedicated computer.

- Programmable Logic Controllers (PLC): digital computers used mainly for automating industrial processes. They are used to continuously monitor sensors (input) and make decisions controlling devices (output).

- Remote Terminal Units (RTU): nowadays RTUs and PLCs functionalities overlap with each other. RTUs are usually preferred for wider geographical telemetry whereas PLCs are better with local controls.

- Communication network: the network connecting all SCADA components (Ethernet, Serial, telephones, radio, cellular...). Network failures do not necessarily impact negatively on the plant process.  Both RTU's and PLC's should be designed to operate autonomously, using the last instruction given from the supervisory system.

- Human Machine Interface (HMI): displays a digitalized representation of the plant. Operators can interact with the plant issuing commands using mouse, keyboards or touch screens. Operators can make supervisory decisions adjusting or overriding the normal plant behaviour.

In short and simple words:

- industries are managed by sophisticated, mission critical computers (SCADA systems);

- security is not the first priority in OT/ICS;

- operators can manually override the behaviour of the plant via mouse/keyboard/touchscreen, locally or remotely;

- a malicious software can override the behaviour of the plant like HMI does.


For more information see [Guide to Industrial Control Systems (ICS) Security (NIST 800-82)](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-82r2.pdf)

## Definition

Operational Technology (OT) and Industrial Control Systems (ICS) are critical components in managing and controlling industrial processes. They encompass a range of hardware and software systems used to monitor and control physical devices and processes in various sectors, including manufacturing, energy, transportation, and utilities. Below is a detailed overview of OT and ICS, including their architecture, functionalities, common vulnerabilities, and exploitation methods.

## Overview of OT and ICS

### Architecture
OT and ICS systems typically consist of several key components:

- **Field Devices**: Sensors, actuators, and other devices that interact with the physical environment to collect data and perform control actions.
- **Control Systems**: These include:
  - **Supervisory Control and Data Acquisition (SCADA)**: Systems that provide centralized monitoring and control.
  - **Distributed Control Systems (DCS)**: Systems that control production processes across multiple locations.
  - **Programmable Logic Controllers (PLCs)**: Industrial computers that automate specific processes based on input from field devices.
- **Human-Machine Interface (HMI)**: The interface that allows operators to interact with the control systems, visualize data, and manage processes.
- **Communication Networks**: Protocols and infrastructure that facilitate data exchange between devices, control systems, and operators.

### Key Features
- **Real-Time Monitoring**: Continuous monitoring of industrial processes to ensure optimal performance.
- **Data Acquisition**: Collecting data from various sensors and devices for analysis and reporting.
- **Control Capabilities**: Enabling operators to remotely control machinery and processes.
- **Alarm Management**: Alerting operators to abnormal conditions or system failures.

## Common Vulnerabilities and Exploitation Methods

### Vulnerabilities
OT and ICS systems can be vulnerable to various threats, including:

- **Legacy Systems**: Many OT and ICS environments rely on outdated technology, making them susceptible to known vulnerabilities.
- **Weak Security Practices**: Inadequate security measures, such as weak passwords and lack of encryption, can expose systems to attacks.
- **Insufficient Network Segmentation**: Poorly segmented networks can allow attackers to move laterally within the system.
- **Supply Chain Vulnerabilities**: Compromised third-party components can introduce risks into the OT environment.

### Common CVEs
Several CVEs are associated with OT and ICS systems. Some notable ones include:

- **CVE-2017-1001000**: A vulnerability in Schneider Electric's Modicon PLCs that allows remote code execution.
- **CVE-2018-15363**: A vulnerability in Siemens ICS that can lead to denial of service.
- **CVE-2020-10148**: A vulnerability in the OpenPLC project that allows unauthorized access to the web interface.

### Exploitation Techniques
Attackers may use various techniques to exploit vulnerabilities in OT and ICS systems:

1. **Network Attacks**:
   - **Man-in-the-Middle (MitM)**: Intercepting and altering communication between devices.
   - **Denial of Service (DoS)**: Overloading the system to disrupt operations.

2. **Remote Code Execution**:
   - Exploiting vulnerabilities in PLCs or other control devices to execute arbitrary code and gain control over processes.

3. **Social Engineering**:
   - Manipulating personnel to gain access to sensitive information or systems.

4. **Physical Attacks**:
   - Gaining physical access to OT components to manipulate or sabotage operations.

5. **Malware Deployment**:
   - Using malware to compromise OT systems, such as Stuxnet, which targeted industrial control systems in Iran.

## Mitigation Strategies

To protect OT and ICS systems from exploitation, organizations should implement the following strategies:

- **Regular Updates and Patching**: Keep all software and firmware up to date to mitigate known vulnerabilities.
- **Strong Authentication and Access Controls**: Implement multi-factor authentication and strict access controls to limit user access.
- **Network Segmentation**: Isolate OT networks from corporate networks to reduce the attack surface.
- **Encryption**: Use encryption for data in transit to protect against interception.
- **Incident Response Planning**: Develop and regularly test incident response plans to quickly address security breaches.

By understanding the architecture, vulnerabilities, and exploitation methods associated with OT and ICS systems, organizations can better secure their industrial environments against potential threats.