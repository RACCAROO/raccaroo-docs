# System Overview

## Purpose

This document provides a high-level overview of the RACCAROO Systems architecture.

It describes the primary building blocks of the ecosystem and their responsibilities without going into implementation details.

---

## High-Level Architecture

The ecosystem is built around a central platform called **RACCAROO Core**.

The Core orchestrates all connected services, devices, and automation logic while remaining independent from any specific hardware vendor or smart home platform.

External systems communicate with the Core through dedicated integration modules.

---

## Primary Modules

### RACCAROO Core

Responsible for:

- orchestration
- decision making
- automation
- context management
- agent coordination

---

### User Interfaces

Interfaces used by users to interact with the ecosystem.

Examples:

- Mobile App
- Web Dashboard
- Voice Assistant
- Telegram
- CLI

---

### AI Services

Provides language understanding, reasoning and intelligent processing.

The Core may use one or multiple AI providers without being coupled to a specific model.

---

### Home Integration

Provides integration with smart home platforms.

Examples:

- Home Assistant
- Zigbee2MQTT
- ESPHome
- Matter (future)

---

### External Services

Integrates external platforms and services.

Examples:

- Calendar
- Email
- Weather
- GitHub
- Linear
- Messaging platforms

---

### Communication Layer

Responsible for communication between services and devices.

Examples:

- MQTT
- HTTP
- WebSocket
- BLE
- Zigbee
- Matter (future)

---

### Embedded Devices

Firmware running on physical hardware.

Examples:

- ESP32
- Sensors
- Relays
- Displays
- Future embedded platforms
