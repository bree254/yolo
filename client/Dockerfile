# Build static assets using node
FROM node:16-alpine AS build

# Working directory
WORKDIR /app

# Copy the package.json and package-lock.json files
COPY package*.json ./

#Install dependencies
RUN npm ci

# Copy the rest of the application code
COPY . .

# Compile the production build
RUN npm run build

# Serve the production build with Nginx
FROM nginx:1.25-alpine

#Copy the compiled frontend into the default nginx public folder
COPY --from=build /app/build /usr/share/nginx/html

# Expose the port
EXPOSE 80

# Run Nginx in the foreground
CMD ["nginx" , "-g" , "daemon off;"]
