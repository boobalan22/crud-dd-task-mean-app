FROM node:18
WORKDIR /app
RUN npm install -g @angular/cli
COPY package*.json ./
RUN npm install
COPY . .
RUN ng build --configuration production
RUN npm install -g http-server
EXPOSE 8081
CMD ["http-server", "dist/angular-15-crud", "-p", "8081"]
