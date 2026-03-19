# Client Build Stage
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
# Install dependencies
RUN npm install
# Copy rest of the application code
COPY . .
# Build the application
RUN npm run build

# Nginx Production Stage
FROM nginx:alpine
# Create Nginx configuration for React Router and specify port 5173
# This matches the CORS configuration in the backend
RUN echo "server { \
    listen       5173; \
    server_name  localhost; \
    location / { \
        root   /usr/share/nginx/html; \
        index  index.html index.htm; \
        try_files \$uri \$uri/ /index.html; \
    } \
}" > /etc/nginx/conf.d/default.conf

# Copy built assets from builder stage
COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 5173
CMD ["nginx", "-g", "daemon off;"]
