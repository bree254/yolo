# Use Node 16 (alpine) as base image
FROM node:16-alpine AS build

# Working directory
WORKDIR /app

# Copy the package.json and package-lock.json files
COPY package*.json ./

#Install dependencies
RUN npm ci --omit=dev

# Copy the rest of the application code
COPY . .

# Add multi stage build
FROM alpine:3.16.7 AS prod

ENV NODE_ENV=production
WORKDIR /app

RUN apk add --no-cache nodejs

COPY --from=build /app /app

#Expose the port
EXPOSE 5000

#Define command to run your app
CMD ["node", "server.js"]
