---
title: "Exploring Nvidia Aerial-Omniverse Digital Twin for Autonomous Driving"
date: 2026-10-10 18:44:00 +0200
categories: [AutonomousDriving, Simulation]
tags: [Omniverse, SUMO, CARLA, DigitalTwin, MultiScale]
toc: true
---
# Exploring Nvidia Aerial-Omniverse Digital Twin for Autonomous Driving

EURECOM developped a multi-scale framework to model autonomous driving vehicles in urban environment. While SUMO (Simulation of Urban Mobiltiy) addresses large-scale mobility, CARLA  (Cars learning how to act) addresses precise control and manoeuver for autonomous vehicles in selected areas. Nvidia Omnivese proposes interoperable APIs to interconnect multiple tools in order to build digital twins. 

## Understanding the project

### What is a multi-scale framework?

In this context, a multi-scale framework refers to an integrated simulation architecture that bridges different levels of spatial and temporal resolution. It combines macroscopic (city wide traffic) and microscopic (individual vehicle behavior and physics) modeling. Testing an autonomous vehicles (AVs) requires analyzing everything from global city traffic patterns, a single tool cannot handle all scales efficiently. 

It’s possible to distinguish the multi-scale framework by separating it into 3 main categories: 

| Categories | Macroscopic level | Microscopic level | The integration layer  |
| --- | --- | --- | --- |
| Tools example | SUMO (simulation of urban mobility) | CARLA (cars learning how to act) | NVIDIA Omniverse |
| What it does | Models large-scale traffic networks, city grids, public transit, and thousands of vehicles simultaneously at a statistical or aggregate level. It focuses on traffic flow, congestion, routing, and urban infrastructure rather than individual vehicle physics. | Focuses on the fine-grained details. It simulates precise vehicle kinematics, complex maneuvers, pedestrian behavior, and high-fidelity sensor physics (such as LiDAR, cameras, and radar) in specific urban zones. | Acts as the interoperable backbone that connects these disparate tools. It allows data to flow seamlessly between macro-simulators (like SUMO) and micro-simulators (like CARLA) in real time, creating a unified, synchronized **digital twin** of an entire city. |

## Source

1. https://www.nvidia.com/en-us/omniverse/
