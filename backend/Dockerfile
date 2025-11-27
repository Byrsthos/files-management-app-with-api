FROM node:22-alpine

WORKDIR /app


COPY backend/package*.json ./
RUN npm install

RUN npm install -g typescript

COPY backend/ ./
RUN npm run build

RUN mkdir -p data uploads

EXPOSE 8001

CMD ["npm", "start"]