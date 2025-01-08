#*Using node js as the environment platform or base image of my container
FROM node:20

#*Makes all subsequent commands after this be in this directory of the container
WORKDIR /user/app

#*We will want to copy the file containing info of the dependencies first to the user/app directory
COPY package.json .

COPY package-lock.json .

#*This runs when the image is building.It is great for installing dependencies
RUN npm install

#*Copies all the code from our codebase to the container
COPY . .

#*Sets the environment variable for the container
ENV PORT=2000

#*Exposes the container to the port 4100 to use it from the outside world
EXPOSE 4100

#*This cmd will run when the container gets executed.It is for starting the app
ENTRYPOINT ["npm","run","start"]