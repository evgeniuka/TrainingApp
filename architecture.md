# FitCoach / TrainingApp — System Architecture

```mermaid
flowchart LR
  subgraph Clients[Clients (React)]
    TrainerUI[Trainer UI (React)]
    TraineeUI[Trainee UI (React)]
  end

  subgraph Backend[Backend (Flask)]
    API[Flask REST API]
    WS[Socket.IO / WebSocket]
    Auth[JWT Authentication + RBAC<br/>(trainer / trainee)]
    SessionSvc[Session Service]
    LogSvc[Session Logging]
    HistorySvc[History Queries]
    AnalyticsSvc[Analytics Aggregations]
  end

  DB[(Firebase Database)]

  TrainerUI -->|POST /auth/login| API
  TraineeUI -->|POST /auth/login| API
  API --> Auth
  Auth -->|JWT token| TrainerUI
  Auth -->|JWT token| TraineeUI

  TrainerUI -->|Trainees/Programs CRUD| API
  API <--> DB

  TrainerUI -->|POST /sessions (2–4 trainees)| API
  API --> SessionSvc
  SessionSvc --> LogSvc

  TrainerUI <--> |session events| WS
  TraineeUI <--> |session events (read-only)| WS
  WS --> Auth
  WS --> SessionSvc
  SessionSvc --> LogSvc
  LogSvc --> DB

  TrainerUI -->|History| API --> HistorySvc
  TraineeUI -->|History| API --> HistorySvc
  TrainerUI -->|Analytics| API --> AnalyticsSvc
  TraineeUI -->|Analytics| API --> AnalyticsSvc
  HistorySvc --> DB
  AnalyticsSvc --> DB
