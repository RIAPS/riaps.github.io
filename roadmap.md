## Roadmap for RIAPS

Last update: 2/4/20

### Apps
- Energy Management App
  - Aggregator node acting as gateway between EV chargers, data center servers, building management systems and a microgrid controller
  - Aggregator collects current and projected energy demands and energy availability, computes
  scheduled allocations and disseminates 
- Integrated Microgrid Control Platform – 24m
  - Complete suite of microgrid control functions implemented, based on proposed standard functions
  - Reference implementation of a generic microgrid  controller

### Interfaces
- RIAPS Device Interface Components for:
  - IEEE 2030.5, DNP3, IEC 61850, and SunSpec – standard interfaces
  - Gridlab-D, OpenDSS:  power system sims, useful in training and app development
- Generic, customizable ‘bridge’ to: Edge-X, OpenFMB (nats.io), MQTT 
- Generic, customizable interface to
  - InfluxDB (to store time-series data)
  - PVBrowser (open source SCADA) 

### Platforms ports
- Raspberry PI 4 / ARM- 64 bit
- NVIDIA Jetson Nano (ML platform)
- RISC-V

### Features
- Fine-grain scheduling of component operations (priority, round-robin)
- Real-time actors running at real-time priority and with scheduling policy 
- Direct inter-thread (component) messages
- App-to-app communication / multi-app deployment
- Leader election algorothm parameterization via architecture model
- Improved fault management: restart policies, checkpointing, etc. 
- Synchronize C++ distributed coordination with Python version
- Fine-grain per-app security policy control
- Better logging framework, log management
- Better app packaging: tooling for deploying large libraries
- Security review – NIST ICS security requirements
- Improved documentation, tutorials, hackathon materials
