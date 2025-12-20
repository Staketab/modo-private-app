# Production Makefile for Modo App (Rust Backend)
# Docker compose file selection
COMPOSE_LOCAL := docker-compose-local-db-prd.yml
COMPOSE_REMOTE := docker-compose-remote-db-prd.yml

# Detect docker compose command (v1 or v2)
compose_v2_supported = $(shell command docker compose 2> /dev/null)
ifeq (,$(compose_v2_supported))
  DOCKER_COMPOSE_COMMAND = docker-compose
else
  DOCKER_COMPOSE_COMMAND = docker compose
endif

# Load environment variables from .env file
-include .env
export

# Default image tag
IMAGE_TAG ?= latest
DOCKER_IMAGE_PREFIX ?= modoapp

.PHONY: help
help:
	@echo "Production Makefile Commands (Rust Backend):"
	@echo ""
	@echo "Quick Start:"
	@echo "  make setup-local      - Setup and start with local database"
	@echo "  make setup-remote     - Setup and start with remote database"
	@echo ""
	@echo "Local Database Setup:"
	@echo "  make up-local         - Start all services with local database"
	@echo "  make down-local       - Stop all services"
	@echo "  make logs-local       - View logs"
	@echo "  make restart-local    - Restart all services"
	@echo ""
	@echo "Remote Database Setup:"
	@echo "  make up-remote        - Start all services with remote database"
	@echo "  make down-remote      - Stop all services"
	@echo "  make logs-remote      - View logs"
	@echo "  make restart-remote   - Restart all services"
	@echo ""
	@echo "Database Management:"
	@echo "  make db-up            - Start only database (local)"
	@echo "  make db-down          - Stop database (local)"
	@echo "  make db-logs          - View database logs (local)"
	@echo ""
	@echo "Service Management:"
	@echo "  make logs-backend     - View backend logs"
	@echo "  make logs-frontend    - View frontend logs"
	@echo "  make restart-backend  - Restart backend only"
	@echo ""
	@echo "Utilities:"
	@echo "  make ps               - Show running containers"
	@echo "  make clean            - Remove stopped containers"
	@echo "  make pull             - Pull latest images"

# =============================================================================
# Quick Setup
# =============================================================================

.PHONY: setup-local
setup-local:
	@echo "Setting up with LOCAL database..."
	@if [ ! -f .env ]; then \
		echo "Creating .env from env.example..."; \
		cp env.example .env; \
		echo "⚠️  Please edit .env with your configuration"; \
		exit 1; \
	fi
	@echo "Starting database..."
	@$(DOCKER_COMPOSE_COMMAND) -f $(COMPOSE_LOCAL) up -d db redis
	@echo "Waiting for database to be ready..."
	@sleep 5
	@echo "Starting all services..."
	@$(DOCKER_COMPOSE_COMMAND) -f $(COMPOSE_LOCAL) up -d
	@echo "✅ Setup complete!"
	@echo ""
	@echo "Services:"
	@echo "  Frontend: http://localhost:3000"
	@echo "  Backend:  http://localhost:$(LOCAL_PORT)"

.PHONY: setup-remote
setup-remote:
	@echo "Setting up with REMOTE database..."
	@if [ ! -f .env ]; then \
		echo "Creating .env from env.example..."; \
		cp env.example .env; \
		echo "⚠️  Please edit .env with your configuration (set DB_URL)"; \
		exit 1; \
	fi
	@if [ -z "$(DB_URL)" ]; then \
		echo "❌ Error: DB_URL is not set in .env"; \
		exit 1; \
	fi
	@echo "Starting services..."
	@$(DOCKER_COMPOSE_COMMAND) -f $(COMPOSE_REMOTE) up -d
	@echo "✅ Setup complete!"
	@echo ""
	@echo "Services:"
	@echo "  Frontend: http://localhost:3000"
	@echo "  Backend:  http://localhost:$(LOCAL_PORT)"

# =============================================================================
# Local Database Setup
# =============================================================================

.PHONY: up-local
up-local:
	@echo "Starting services with LOCAL database..."
	@if [ ! -f .env ]; then \
		echo "❌ Error: .env file not found. Run: make setup-local"; \
		exit 1; \
	fi
	@$(DOCKER_COMPOSE_COMMAND) -f $(COMPOSE_LOCAL) up -d
	@echo "✅ Services started"

.PHONY: down-local
down-local:
	@echo "Stopping services..."
	@$(DOCKER_COMPOSE_COMMAND) -f $(COMPOSE_LOCAL) down
	@echo "✅ Services stopped"

.PHONY: logs-local
logs-local:
	@$(DOCKER_COMPOSE_COMMAND) -f $(COMPOSE_LOCAL) logs -f --tail 100

.PHONY: restart-local
restart-local:
	@echo "Restarting services..."
	@$(DOCKER_COMPOSE_COMMAND) -f $(COMPOSE_LOCAL) restart
	@echo "✅ Services restarted"

# =============================================================================
# Remote Database Setup
# =============================================================================

.PHONY: up-remote
up-remote:
	@echo "Starting services with REMOTE database..."
	@if [ ! -f .env ]; then \
		echo "❌ Error: .env file not found. Run: make setup-remote"; \
		exit 1; \
	fi
	@if [ -z "$(DB_URL)" ]; then \
		echo "❌ Error: DB_URL is not set in .env"; \
		exit 1; \
	fi
	@$(DOCKER_COMPOSE_COMMAND) -f $(COMPOSE_REMOTE) up -d
	@echo "✅ Services started"

.PHONY: down-remote
down-remote:
	@echo "Stopping services..."
	@$(DOCKER_COMPOSE_COMMAND) -f $(COMPOSE_REMOTE) down
	@echo "✅ Services stopped"

.PHONY: logs-remote
logs-remote:
	@$(DOCKER_COMPOSE_COMMAND) -f $(COMPOSE_REMOTE) logs -f --tail 100

.PHONY: restart-remote
restart-remote:
	@echo "Restarting services..."
	@$(DOCKER_COMPOSE_COMMAND) -f $(COMPOSE_REMOTE) restart
	@echo "✅ Services restarted"

# =============================================================================
# Database Management (Local only)
# =============================================================================

.PHONY: db-up
db-up:
	@echo "Starting local database..."
	@$(DOCKER_COMPOSE_COMMAND) -f $(COMPOSE_LOCAL) up -d db
	@echo "✅ Database started"

.PHONY: db-down
db-down:
	@echo "Stopping local database..."
	@$(DOCKER_COMPOSE_COMMAND) -f $(COMPOSE_LOCAL) stop db
	@echo "✅ Database stopped"

.PHONY: db-logs
db-logs:
	@$(DOCKER_COMPOSE_COMMAND) -f $(COMPOSE_LOCAL) logs -f db

# =============================================================================
# Service Management
# =============================================================================

.PHONY: logs-backend
logs-backend:
	@$(DOCKER_COMPOSE_COMMAND) -f $(COMPOSE_LOCAL) logs -f rust-backend 2>/dev/null || \
	$(DOCKER_COMPOSE_COMMAND) -f $(COMPOSE_REMOTE) logs -f rust-backend

.PHONY: logs-frontend
logs-frontend:
	@$(DOCKER_COMPOSE_COMMAND) -f $(COMPOSE_LOCAL) logs -f frontend 2>/dev/null || \
	$(DOCKER_COMPOSE_COMMAND) -f $(COMPOSE_REMOTE) logs -f frontend

.PHONY: restart-backend
restart-backend:
	@echo "Restarting backend..."
	@$(DOCKER_COMPOSE_COMMAND) -f $(COMPOSE_LOCAL) restart rust-backend 2>/dev/null || \
	$(DOCKER_COMPOSE_COMMAND) -f $(COMPOSE_REMOTE) restart rust-backend
	@echo "✅ Backend restarted"

# =============================================================================
# Utilities
# =============================================================================

.PHONY: ps
ps:
	@echo "Running containers:"
	@$(DOCKER_COMPOSE_COMMAND) -f $(COMPOSE_LOCAL) ps 2>/dev/null || \
	$(DOCKER_COMPOSE_COMMAND) -f $(COMPOSE_REMOTE) ps 2>/dev/null || \
	echo "No containers running"

.PHONY: clean
clean:
	@echo "Removing stopped containers..."
	@docker container prune -f
	@echo "✅ Cleanup completed"

.PHONY: pull
pull:
	@echo "Pulling latest images..."
	@docker pull $(DOCKER_IMAGE_PREFIX)/rust-backend:$(IMAGE_TAG)
	@docker pull $(DOCKER_IMAGE_PREFIX)/frontend:$(IMAGE_TAG)
	@echo "✅ Images updated"
