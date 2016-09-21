FROM mhart/alpine-node:latest
RUN mkdir /code
WORKDIR /code
ADD ./app/. /code/
RUN npm install
CMD ["node", "main.js"]
