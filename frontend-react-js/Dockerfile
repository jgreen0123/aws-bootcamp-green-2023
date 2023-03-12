FROM node:16.18

ENV PORT=3000

COPY . /frontend-react-js
WORKDIR /frontend-react-js
RUN npm i -g npm@9.6.1
EXPOSE ${PORT}
CMD ["npm", "start"]