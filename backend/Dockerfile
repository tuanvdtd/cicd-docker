# syntax=docker/dockerfile:1
FROM node:20-alpine

# Create app directory
WORKDIR /app

# Install dependencies first (leverage layer caching)
COPY package*.json ./
RUN npm install --omit=dev

# Bundle app source
COPY . .

# App port
ENV PORT=3001
EXPOSE 3001

# Start the service
CMD ["npm", "start"]
