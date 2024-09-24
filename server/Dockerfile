FROM node:latest
WORKDIR /usr/app
COPY package*.json /usr/app
RUN npm install
COPY . .
EXPOSE 3001
CMD [ "npm", "start" ]