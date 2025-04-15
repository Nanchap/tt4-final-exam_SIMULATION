# Step 1: Use Node.js to build the app
FROM node:16 AS build

# Step 2: Set the working directory for building the app
WORKDIR /app

# Step 3: Copy the package files and install dependencies
COPY package*.json ./
RUN npm install

# Step 4: Copy the rest of the source code
COPY . .

# Step 5: Build the app for production
RUN npm run build

# Step 6: Use Nginx to serve the built files
FROM nginx:alpine

# Step 7: Copy the built app files into Nginx
COPY --from=build /app/public /usr/share/nginx/html

# Step 8: Expose the port Nginx is running on
EXPOSE 80

# Step 9: Start the Nginx server
CMD ["nginx", "-g", "daemon off;"]
