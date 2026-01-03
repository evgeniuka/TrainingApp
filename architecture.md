# FitCoach / TrainingApp — System Architecture

This document describes the high-level architecture of the FitCoach/TrainingApp project (React clients + Flask backend + Firebase + WebSocket real-time updates).

## 1) High-Level Architecture Diagram

```mermaid
flowchart LR
  subgraph Clients["Clients (React)"]
    TrainerUI["Trainer UI (React)"]
    TraineeUI["Trainee UI (React)"]
  end

  subgraph Backend["Backend (Flask)"]
    API["Flask REST API"]
    WS["Socket.IO / WebSocket"]
    Auth["JWT + RBAC (trainer/trainee)"]
    SessionSvc["Session Service"]
    LogSvc["Session Logging"]
    HistorySvc["History Queries"]
    AnalyticsSvc["Analytics Aggregations"]
  end

  Firebase[(Firebase)]

  %% Login + Auth
  TrainerUI -->|"POST /auth/login"| API
  TraineeUI -->|"POST /auth/login"| API
  API --> Auth
  Auth -->|"JWT token"| TrainerUI
  Auth -->|"JWT token"| TraineeUI

  %% Core CRUD + Persistence
  TrainerUI -->|"CRUD trainees/programs"| API
  API <--> Firebase

  %% Sessions: start / live updates / end
  TrainerUI -->|"POST /sessions (2-4 trainees)"| API
  API --> SessionSvc
  SessionSvc --> LogSvc

  TrainerUI <-->|"session events"| WS
  TraineeUI <-->|"session events (read-only)"| WS
  WS --> Auth
  WS --> SessionSvc
  SessionSvc --> LogSvc
  LogSvc --> Firebase

  %% Reports: history + analytics
  TrainerUI -->|"History"| API --> HistorySvc
  TraineeUI -->|"History"| API --> HistorySvc
  TrainerUI -->|"Analytics"| API --> AnalyticsSvc
  TraineeUI -->|"Analytics"| API --> AnalyticsSvc
  HistorySvc --> Firebase
  AnalyticsSvc --> Firebase
