FROM node:15-alpine

RUN mkdir -p /opt/presentation/component
RUN mkdir -p /opt/presentation/images

WORKDIR /opt/presentation/

COPY deck.mdx .
ADD component/ /opt/presentation/component/
ADD images/ /opt/presentation/images/
COPY package-lock.json .
COPY package.json .

RUN ls *

RUN npm install

EXPOSE 8000

CMD npm start
