# See running containers in this compose
docker compose -f ops/docker-compose.yml ps

# Stream logs
docker compose -f ops/docker-compose.yml logs -f

# Exec into a container (e.g., check broker)
docker compose -f ops/docker-compose.yml exec rabbitmq rabbitmq-diagnostics ping

# Restart just RabbitMQ
docker compose -f ops/docker-compose.yml restart rabbitmq

# Stop containers (keeps volumes)
docker compose -f ops/docker-compose.yml down

# Stop and remove volumes (wipes data)
docker compose -f ops/docker-compose.yml down -v
